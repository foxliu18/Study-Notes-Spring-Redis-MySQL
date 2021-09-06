# 1. 特性

hash表：

1. 哈希表在使用层面可以理解为一种集合结构
2. 如果只有key，没有伴随数据value，可以使用HashSet结构
3. 如果既有key，又有伴随数据value，可以使用HashMap结构
4. 有无伴随数据，是hashSet和HashMap的唯一区别，低层的实际结构是一回事
5. 使用Hash表曾、删、改、查的操作，可以认为时间复杂度为O(1)，但是常数时间比较大
6. 放入hash表的东西，如果是基础类型，内部按值传递，内存占用就是这个东西的大小
7. 放入hash表的东西，如果不是基础类型，内部按引用传递，内存占用是这个东西内存地址大小，一律8字节

有序表：

1. 有序表在使用层面上可以理解为一种集合结构
2. 如果只有key，没有伴随数据value，可以使用TreeSet结构
3. 如果既有key，又有伴随数据value，可以使用TreeMap结构
4. 有无伴随数据，是TreeSet和TreeMap唯一的区别，底层的实际结构是一回事
5. 有序表和hash表的区别是，有序表把key按照顺序组织起来，而hash表完全不组织
6. 红黑树、AVL树、size-blance-tree和跳表等都属于有序表结构，只是底层具体实现不同
7. 放入有序表的东西，如果是基础类型内部按值传递，内存占用就是这个东西的大小
8. 放入有序表的东西，如果不是基础类型，必须提供比较器，内部按引用传递，内存占用是这个东西内存地址大小，一律8字节
9. 不管是什么底层具体实现，只要是有序表，都有以下固定的基本功能和固定的时间复杂度O(logN)， N为有序表函数的记录数

> 问题：判断单链表是否为回温，使用面试解法，不新增容器，额外空间复杂度O(1)

``` java
//need O(1) extra space
public static boolean isPalindrome3(Node head){
  if(head == null || head.next == null){
    return true;
  }
  Node n1 = head;
  Node n2 = head;
  while(n2.next != null && n2.next.next != null){ //find mid node
    n1 = n1.next; // n1 -> mid
    n2 = n2.next.next; // n2 -> end
  }
  n2 = n1.next; // n2 -> right part first node
  n1.next = null; //mid.next -> null
  Node n3 = null;
  while(n2 != null){ // right part convert
    n3 = n2.next; // n3 -> save next node
    n2.next = n1; // next of right node convert
    n1 = n2; // n1 move
    n2 = n3; // n2 move
  }
  n3 = n1; // n3 -> save last node
  n2 = head; // n2 -> left first node
  boolean res = true;
  while(n1 != null && n2 != null){ // check palindrome
    if(n1.value != n2.value){
      res = false;
    }
    n1 = n1.next; // left to mid
    n2 = n2.next; // right to mid
  }
  
  n1 = n3.next;
  n3.next = null;
  while(n1 != null){ // recover list
    n2 = n1.next;
    n1.next = n3;
    n3 = n1;
    n1 = n2;
  }
  return res;
}
```

> 问题：把单链表按固定值privot分为，左边都是小于privot，中间等于privot，右边大于privot的节点

``` java
public static Node listPartition2(Node head, int privot){
  Node sH = null; // small zone head
  Node sT = null; // small zone tail
  Node eH = null; // equal zone head
  Node eT = null; // equal zone head
  Node mH = null; // big zone head
  Node mT = null; // big zone head
  Node next = null; // save the node
  // every node distributed to three lists
  while(head != null){
    next = head.next;
    head.next = null;
    if(head.vakue < privot){
      if(sH == null){
        sH = head;
        sT = head;
      }else{
        sT.next = head;
        sT = head;
      }
    }else if(head.value == privot){
      if(eH == null){
        eH = head;
        eT = head;
      }else{
        eT.next = head;
        eT =head;
      }
    }else{
      if(mH ==null){
        mT = head;
        mH = head;
      }else{
        mT.next = head;
        mT =head;
      }
    }
    head = next;
  }
  // small and equal reconnect
  if(sT != null){ // 如果有小于区域
    sT.next = eH;
    eT =eT == null? sT : eT; 谁去连大于区的头，谁就变成eT
  }
  if(eT != null){
    eT.next = mH;
  }
  return sH != null ? sH : (eH != null eH : mH);
}
```

> 拷贝一个有随机节点的单向链表

一、使用额外空间

使用HashMap，key为老节点，value为克隆出的新节点

1. 拷贝所有节点，不考虑next和random，先放入map中
2. 开始设置新节点next和random的新节点，遍历链表或map可以一次获得原节点和复制节点，一一设置next和random即可

``` java
public static Node copyListWithRand1(Node node){
  HashMAp<Node, Node> map = new HashMap<Node, Node>();
  Node cur = head;
  while(cur != null){
    map.put(cur, new Node(cur.value));
    cur = cur.next;
  }
  cur = head;
  while(cur != null){
    // cur old
    // map.get(cur) new
    map.get(cur).next = map.get(cur.next);
    map.get(cur).rand = map.get(cur.ramd);
    cur = cur.next;
  }
  return map.get(head);
}
```

二、不使用额外空间

把复制节点插入到当前节点的下一个节点，依次复制插入，形成一对一对的复制节点，在将rand节点复制，之后讲复制节点依次分离出新的链表

``` java
public static Node copyListWithRand2(Node node){
  if(head == null){
    return null;
  }
  Node cur = head;
  Node next = null;
  // copu node and link to every node
  // 1 -> 2
  // 1 -> 1` -> 2
  while(cur != null){
    next = cur.next;
    cur.next = new Node(cur.value);
    cur.next.next = next;
    cur = next;
  }
  cur = head;
  Node curCopy = null;
  // set copy node rand
  // 1 -> 1` - > 2 -> 2`
  while(cur != null){
    next = cur.next.next;
    curCopy = cur.next;
    curCopy.rand = cur.rand != null ? cur.rand.next : null;
    cur = next;
  }
  Node res = head.next;
  cur = head;
  // split
  while(cur != null){
    next = cur.next.next;
    curCopy = cur.next;
    cur.next = next;
    curCopy.next = next != null ? next.next : null;
    cur = next;
  }
  return res;
}
```

# 2. 题目 两个链表相交问题

> 给定两个可能有环也可能无环的链表，头节点head1和head2.请实现一个函数，如果两个链表相交，返回相交的第一个节点。如果不相交，返回null

==相交的含义是节点内存地址相同，不单单是节点值相同==

###  1、判断链表是否有环

1. 使用额外空间，申请一个hashSet，遍历链表，判断Set是否含有改节点，没有的话节点放入set，直到Set中含有遍历到的节点，或next为null

2. 不使用额外空间

   快慢指针：

   		1. 快慢指针从头开始，快指针一次两步，慢指针一次一步
     		2. 如果没有环，快指针会先到null
     		3. 如果有环，快慢指针一定在环内某节点相遇，并且两指针不会在环内转超过两圈，复杂度为常数级
     		4. 两指针相遇后，快指针回到链表头，慢指针不变
     		5. 快慢指针继续走，快指针也每一次走一步，两指针会在环的第一个节点相遇

``` java
// 找到链表第一个入环节点，如果无环，返回null
public static Node getLoopNode(Node head){
  if(head == null || head.next == null || head.next.next == null){
    return null;
  }
  Node n1 = head.next; // n1 -> slow
  Node n2 = head.next.next; // n2 -> fast
  while(n1 != n2){
    if(n2.next == null || n2.next.next == null){
      return null;
    }
    n2 = n2.next.next;
    n1 = n1.next;
  }
  n2 = head; // n2 -> walk again from head
  while(n1 != n2){
    n1 = n1,next;
    n2 = n2.next;
  }
  return n1;
}
```

### 2、两链表都无环

 1. 遍历两链表得到链表长度和最后一个节点

 2. 判断end节点是否为一个，

    + 为同一个，则两链表相交
    + 不为同一个，则两链表一定不相交

	3. 若相交，长列表先从头走两链表长度差值步后，短链表开始从头开始走

	4. 依次比较遍历各个节点是否相同

    ``` java
    // 如果两链表都无环，返回第一个相交节点，如果不相交，返回null
    public static Node noLoop(Node head1, Node head2){
      if(head1 == null || head2 == null){
        return null;
      }
      Node cur1 = head1;
      Node cur2 = head2;
      int n = 0;
      while(cuir1.next != null){
        n++;
        cur1 = cur1.next;
      }
      while(cur2.next != null){
        n--;
        cur2 = cur2.next;
      }
      if(cur1 != cur2){
        return null;
      }
      // n 是链表1长度减链表2的长度
      cur1 = n > 0 ? head1:head2; // 谁长谁的头变成cur1
      cur2 = cur1 == head1 ? head2 : head1; // 谁短谁的头变成cur2
      n = Math.abs(n);
      while(n != 0){
        n--;
        cur1 = cur1.next;
      }
      while(cur1 != cur2){
        cur1 = cur1.next;
        cur2 = cur2.next;
      }
      return cur1;
    }
    ```

    

### 3、一个链表有环一个链表无环，两链表不会相交

### 4、两个链表都有环

1. 两个有环链表不相交

2. 在入环前两链表相交

3. 在环中相交，入环点可能不一样

```java
// 两个有环链表，返回第一个相交节点， 如果不相交返回null
public static Node bothLoop(Node head1, Node loop1, Node head2, Node loop2){
  Node cur1 = null;
  Node cur2 = null;
  if(loop1 == loop2){
    cur1 = head1;
    cur2 = head2;
    int n = 0;
    while(cur1 != loop1){
      n++;
      cur1 = cur1.next;
    }
    while(cur2 != loop2){
      n--;
      cur2 = cur2.next;
    }
    cur1 = n > 0 ? head1 : head2;
    cur2 = cur1 == head1 ? head2 : head1;
    n = MAth.abs(n);
    while(n != 0){
      n--;
      cur1 = cur1.next;
    }
    while(cur1 != cur2){
      n--;
      cur1 = cur1.next;
    }
    while(cur1 != cur2){
      cur1 = cur1.next;
      cur2 = cur2.next;
    }
    return cur1;
  }else{
    cur1 = loop1.next;
    while(cur1 != loop1){
      if(cur1 == loop2){
        return loop1;
      }
      cur1 = cur1.next;
    }
    return null;
  }
}
```

