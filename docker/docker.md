# docker

mysql

``` bash
docker run -id 
-p 3307:3306 
--name=c_mysql 
-v $PWD/mysql/conf:/etc/mysql/conf.d 
-v $PWD/mysql/logs:/logs 
-v $PWD/mysql/data:/var/lib/mysql 
-e MYSQL_ROOT_PASSWORD=1234 
mysql
```

tomcat

``` bash
docker run -id 
--name=c_tomcat 
-p 8080:8080 
-v $PWD:/usr/local/tomcat/webapps 
tomcat
```

nginx

``` bash
docker run -id \
--name=c_nginx \
-p 80:80 \
-v $PWD/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/nginx/logs:/var/log/nginx \
-v $PWD/nginx/html:/usr/share/nginx/html \ 
nginx
```

> 要先创建nginx.conf配置文件 

redis

``` bash
docker run -id 
--name=c_redis 
-p 6379:6379 redis
```



## dockerfile

### docker镜像原理

+ Docker 镜像是由特殊的文件系统叠加而成

+ 最底端是bootfs，并使用宿主机的bootfs
+ 第二层是rootfs文件系统，称为base image
+ 然后再往上可以叠加其他镜像文件
+ 统一文件系统（Union File System）技术能够将不同的层整合成一个文件系统，不同镜像的层可以复用，但不能修改，称为只读镜像；为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度来看，只存在一个文件系统
+ 一个镜像可以放在另一个镜像的上面，位于下面的镜像成为父镜像，最底部的镜像成为基础镜像
+ 当从一个镜像启动容器时，docker会在最顶层加载一个读写文件系统作为可写容器，可以对其修改，在封装为新的镜像



### 镜像制作

1. 容器转为镜像

   ``` bash
   docker commit 容器id 镜像名称：版本号
   ```

2. 压缩命令为：

   镜像需要压缩称压缩文件才可以传输，

   docker save -o 压缩文件名称 镜像名称：版本号

3. 压缩文件还原

   ``` bash
   docker load -i 压缩文件名称
   ```

   新的镜像不包括目录挂载中的文件



### dockerfile 概念

+ dockerfile 是一个文本文件
+ 包含了一条条的指令
+ 每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像
+ 对于开发人员：可以为开发团队提供一个完全一直的开发环境
+ 对于测试人员：可以直接拿开发时所构建的镜像或者通过dockerfile文件构建一个新的镜像开始工作
+ 对于运维：在部署时，可以实现应用的无缝移植

关键字

| 关键字      | 作用                     | 备注                                                         |
| ----------- | ------------------------ | ------------------------------------------------------------ |
| FROM        | 指定父镜像               | 指定dockerfile基于那个image构建                              |
| MAINTAINER  | 作者信息                 | 用来标明这个dockerfile谁写的                                 |
| LABEL       | 标签                     | 用来标明dockerfile的标签 可以使用Label代替Maintainer 最终都是在docker image基本信息中可以查看 |
| RUN         | 执行命令                 | 执行一段命令 默认是/bin/sh 格式: RUN command 或者 RUN [“command” , “param1”,“param2”] |
| CMD         | 容器启动命令             | 提供启动容器时候的默认命令 和ENTRYPOINT配合使用.格式 CMD command param1 param2 或者 CMD [“command” , “param1”,“param2”] |
| ENTRYPOINT  | 入口                     | 一般在制作一些执行就关闭的容器中会使用                       |
| COPY        | 复制文件                 | build的时候复制文件到image中                                 |
| ADD         | 添加文件                 | build的时候添加文件到image中 不仅仅局限于当前build上下文 可以来源于远程服务 |
| ENV         | 环境变量                 | 指定build时候的环境变量 可以在启动的容器的时候 通过-e覆盖 格式ENV name=value |
| ARG         | 构建参数                 | 构建参数 只在构建的时候使用的参数 如果有ENV 那么ENV的相同名字的值始终覆盖arg的参数 |
| VOLUME      | 定义外部可以挂载的数据卷 | 指定build的image那些目录可以启动的时候挂载到文件系统中 启动容器的时候使用 -v 绑定 格式 VOLUME [“目录”] |
| EXPOSE      | 暴露端口                 | 定义容器运行的时候监听的端口 启动容器的使用-p来绑定暴露端口 格式: EXPOSE 8080 或者 EXPOSE 8080/udp |
| WORKDIR     | 工作目录                 | 指定容器内部的工作目录 如果没有创建则自动创建 如果指定/ 使用的是绝对地址 如果不是/开头那么是在上一条workdir的路径的相对路径 |
| USER        | 指定执行用户             | 指定build或者启动的时候 用户 在RUN CMD ENTRYPONT执行的时候的用户 |
| HEALTHCHECK | 健康检查                 | 指定监测当前容器的健康监测的命令 基本上没用 因为很多时候 应用本身有健康监测机制 |
| ONBUILD     | 触发器                   | 当存在ONBUILD关键字的镜像作为基础镜像的时候 当执行FROM完成之后 会执行 ONBUILD的命令 但是不影响当前镜像 用处也不怎么大 |
| STOPSIGNAL  | 发送信号量到宿主机       | 该STOPSIGNAL指令设置将发送到容器的系统调用信号以退出。       |
| SHELL       | 指定执行脚本的shell      | 指定RUN CMD ENTRYPOINT 执行命令的时候 使用的shell            |

### dockerfile 案例

定义springboot镜像



1. 定义父镜像                                              FROM java:8
2. 定义作者                                                  MAINTAINER fox \<fox@123.com>
3. 将jar包添加的到容器                            ADD sspringboot.jar app.jar   // 改名为app.jar
4. 定义容器启动执行的命令                    CMD java -jar app.jar
5. 通过dockerfile构建镜像                       docker build -f dockerfile文件路径 -t 镜像名称：版本

## docker 服务编排

### 服务编排概念

微服务架构的应用系统中一般包含多个微服务，每个微服务一般都会不部署多个实例，如果每个微服务都要手动启停，维护的工作量会很大

+ 要从Dockerfile build image或者去dockerhub拉取image
+ 要创建多个container
+ 要管理这些container（启动停止删除）



>  服务编排: 按照一定的业务规则批量管理容器



### Docker Compose 概念

Docker Compose 是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包含服务构建，启动和停止

1. 利用Dockerfile定义运行环境镜像
2. 利用Docker-compose.yml 定义组成应用的各服务
3. 运行docker-compose up 启动应用



### 案例

使用docker compose 编排nginx + springboot项目

1. 创建docker-compose目录

2. 编写docker-compose.yml 文件

   ```shell
   
   version: '3'
   services:
   	nginx:
   		image: nginx
   		ports:
   			- 80:80
   		links:
   			-app # nginx所代理的服务名
   		volumes:
   			- ./nginx/conf.d:/etc/nginx/conf.d
   	app:
   		image: app
   		expose: - "8080"
   
   
   ```

3. 创建./nginx/conf.d 目录

4. 在conf.d目录下编写配置文件

   ``` bash
   server {
   	listen 80;
   	access_log off;
   	location / {
   		proxy_pass http://app:8080; # app为docker-compose.yml中配置的服务名
   	}
   }
   
   ```




## docker私有仓库

### 搭建私有仓库

+ docekrhub是用于管理公共镜像的仓库



一、私有仓库搭建

``` shell
# 1. 拉取私有仓库镜像
docker pull registry
# 2. 启动私有仓库容器
docker run -id --name=registry -p 5000:5000 registry
# 3. 打开浏览器输入地址 http://ip:5000/v2/_catalog, 看到{“repositories”:[]} 表示私有仓库搭建成功
# 4. 修改daemon.json
vim /etc/docker/daemon.json
# 在上述文件中添加一个key，保存退出。此步用于让docker信任私有仓库地址；注意将私有仓库服务器ip修改为自己私有仓库服务器真实ip
{"insecure-registries": ["私有仓库服务器ip:5000"]}
# 5. 重启docker 服务
systemctl restart docker
docker start registry

```



二、将镜像上传至私有仓库

``` bash
# 1. 标记镜像为私有仓库的镜像
docker tag centos:7 私有仓库ip:5000/centos:7

# 2. 上传标记的镜像
docker push 私有仓库服务器ip:5000/centos:7
```



三、从私有仓库拉取镜像

``` bash
# 拉取镜像
docker pull 私有仓库服务器ip:5000/centos:7
```



## docker 相关概念

容器就是将软件打包成标准化单元，以用于开发、交付和部署

+ 容器镜像是轻量级的、可执行的独立软件包，包含软件运行所需的所有内容：代码、运行时环境、系统工具、系统库和设置
+ 容器化软件在任何环境中都能够始终如一的运行
+ 容器赋予了软件独立性，使其免受外在环境差异的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突





容器和传统虚拟机对比

相同

+ 容器和虚拟机具有相似的资源隔离和分配优势

不同

+ 容器虚拟化的是操作系统，虚拟机虚拟化的是硬件
+ 传统虚拟机可以运行不同操作系统，容器只能运行同一类操作系统

| 特性       | 容器             | 虚拟机     |
| ---------- | ---------------- | ---------- |
| 启动       | 秒级             | 分钟级     |
| 硬盘使用   | 一般为MB         | 一般为GB   |
| 性能       | 接近原生         | 弱于       |
| 系统支持量 | 单机支持上千容器 | 一般几十个 |

