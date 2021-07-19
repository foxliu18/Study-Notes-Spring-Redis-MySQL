<center><b><h1>
Java SE  </h1></b></center>

## 概述

![](image/JAVA%20SE.svg)

## Reflection



**Reflection**(反射)是Java被视为动态语言的关键, 反射机制允许程序在执行期借助与Reflection API取得任何类的内部信息, 并能直接操作任意对象的内部属性及方法

``` java
Class c = Class.forName("java.lang.String");
```

加载完类之后，在堆内存方法区中就产生了一个class类型的对象（一个类只有一个Class对象）,这个对象就包含了完成类的结构信息. 我们可以通过这个对象看到累的结构. 这个对象就像一面镜子, 透过这个镜子看到类的结构, 所以, 我们形象称之为反射

