---
title: Java 局部变量内存占用的问题
date: 2021-09-03
categoreis:
- 随笔
tags:
- Java
- 面试 
---

今日面试受挫，记录一下

问题：Java 函数局部变量引用是否需要内存

正文来了:

今天面试了小米，面试官最后问了一道简单的算法题，记录：

> 用两个队列实现栈，要求实现入栈出栈以及获取大小 三个函数

题目本身并不难，简单实现了一下  
代码如下：
<!-- more -->

```java
class Stack<T> {
    private Queue<T> inQueue = new LinkedList<>();
    private Queue<T> swapQueue = new LinkedList<>();

    public void push(T element) {
        if (element == null) {
            return;
        }
        inQueue.add(element);
    }

    public T pop() {
        if (inQueue.isEmpty()) {
            return null;
        }
        while (inQueue.size() > 1) {
            swapQueue.add(inQueue.poll());
        }
        Object target = inQueue.poll();
        Queue t = inQueue;
        inQueue = swapQueue;
        swapQueue = t;
        return (T) target;
    }

    public int getSize() {
        return inQueue.size() + swapQueue.size();
    }
}
```

至此，重点来了!!!  

__来自面试官的灵魂拷问：要求是用两个队列实现，你这不是三个队列吗？__
_( 诶 ~ 难道用于交换的局部变量也算么？_（o´・ェ・｀o）

__来自面试官的灵魂拷问2：你看这个交换的地方，增加了一个变量，内存消耗是不是增加了__

…

说实话，此刻真的有点被问懵了 o((⊙﹏⊙))o.

我寻思好像确实是啊，局部变量也要在虚拟机栈帧中分配内存来着。。。啊嘞 (・∀・*)
不对啊，方法执行肯定有对应的栈帧，众所周知 Java 虚拟机栈帧内需要维护局部变量表、操作数栈 … 等，需要内存记录不是应该的吗，这个也不应该算是内存开销吧

搞得我都有点不自信了，于是一顿操作 javap 大法 ```javap -v Stack.class``` 瞅一眼确认一下

```plain
  public T pop();
    descriptor: ()Ljava/lang/Object;
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: getfield      #4                  // Field inQueue:Ljava/util/Queue;
         4: invokeinterface #7,  1            // InterfaceMethod java/util/Queue.isEmpty:()Z
         9: ifeq          14
        12: aconst_null
        13: areturn
        14: aload_0
        15: getfield      #4                  // Field inQueue:Ljava/util/Queue;
        18: invokeinterface #8,  1            // InterfaceMethod java/util/Queue.size:()I
        23: iconst_1
        24: if_icmple     49
        27: aload_0
        28: getfield      #5                  // Field swapQueue:Ljava/util/Queue;
        31: aload_0
        32: getfield      #4                  // Field inQueue:Ljava/util/Queue;
        35: invokeinterface #9,  1            // InterfaceMethod java/util/Queue.poll:()Ljava/lang/Object;
        40: invokeinterface #6,  2            // InterfaceMethod java/util/Queue.add:(Ljava/lang/Object;)Z
        45: pop
        46: goto          14
        49: aload_0
        50: getfield      #4                  // Field inQueue:Ljava/util/Queue;
        53: invokeinterface #9,  1            // InterfaceMethod java/util/Queue.poll:()Ljava/lang/Object;
        58: astore_1
        59: aload_0
        60: getfield      #4                  // Field inQueue:Ljava/util/Queue;
        63: astore_2
        64: aload_0
        65: aload_0
        66: getfield      #5                  // Field swapQueue:Ljava/util/Queue;
        69: putfield      #4                  // Field inQueue:Ljava/util/Queue;
        72: aload_0
        73: aload_2
        74: putfield      #5                  // Field swapQueue:Ljava/util/Queue;
        77: aload_1
        78: areturn
      LineNumberTable:
        line 18: 0
        line 19: 12
        line 21: 14
        line 22: 27
        line 24: 49
        line 25: 59
        line 26: 64
        line 27: 72
        line 28: 77
      StackMapTable: number_of_entries = 2
        frame_type = 14 /* same */
        frame_type = 34 /* same */
    Signature: #27                          // ()TT;

```

查看 60-69：做了几件事

- 读取 inQueue 到栈顶
- 保存到局部变量 2
- 读取 swapQueue 到栈顶
- 赋值给 inQueue 变量
- 把局部变量 2 （原 inQueue）推至栈顶
- 赋值给 swapQueue
  
至此，完成两个队列的交换

这么看一下，确实也就是一个局部变量的事儿，也谈不上有啥内存开销啊 ( ╯▽╰)

所以面试官到底是啥意思。。。
以后再研究一下
