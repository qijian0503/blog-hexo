---
title: JVM运行时数据区
date: 2018-10-15 23:51:24
categories: 深入JVM内核
tags: [jvm]
---
### JVM运行时数据区域
根据 JVM 规范，JVM 内存共分为**虚拟机栈、堆、方法区、程序计数器、本地方法栈**五个部分。  
> 内存空间(Runtime Data Area)中可以按照是否线程共享分为两块，线程共享的是方法区(Method Area)和堆(Heap)，线程独享的是虚拟机栈(VM Stack)，本地方法栈(Native Method Stack)和PC寄存器(Program Counter Register)。  

具体参见下图：  

![JVM运行时数据区](https://upload-images.jianshu.io/upload_images/8760038-0fbf4b052e52c4f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!-- more -->

区域 | 是否线程共享 | 是否会内存溢出
---|---|---
程序计数器 | 否 | 不会
虚拟机栈 | 否 | 会
本地方法栈 | 否 | 会
堆 | 是 | 会
方法区 | 是 | 会

- 虚拟机栈（线程私有）  
每个线程有一个私有的栈，随着线程的创建而创建。栈里面存放着一种叫做“栈帧”的东西，每个方法在执行的时候会创建一个栈帧，存储了局部变量表(基本数据类型和对象引用)，操作数栈，动态连接，方法出口等信息。
每个方法从调用到执行完毕，对应一个栈帧在虚拟机栈中的入栈和出栈。
通常所说的栈，一般是指虚拟机栈中的局部变量表部分。局部变量表所需的内存在编译期间完成分配。
栈的大小可以固定也可以动态扩展，当扩展到无法申请足够的内存，则OutOfMemoryError。
当栈调用深度大于JVM所允许的范围，会抛出StackOverflowError的错误，不过这个深度范围不是一个恒定的值，我们通过下面这段程序可以测试一下这个结果：
    ```
    // 栈溢出测试源码
    package com.paddx.test.memory;
    /**
     * Created by root on 2/28/17.
     */
    public class StackErrorMock {
        private static int index = 1;
    
        public void call() {
            index++;
            call();
        }
    
        public static void main(String[] args) {
            StackErrorMock mock = new StackErrorMock();
            try {
                mock.call();
            } catch(Throwable e) {
                System.out.println("Stack deep: " + index);
                e.printStackTrace();
            }
        }
    }
    ```
    运行三次，可以看出每次栈的深度都是不一样的，输出结果如下：
    ![image.png](https://upload-images.jianshu.io/upload_images/8760038-a0f0b393a34ec6eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    ![image.png](https://upload-images.jianshu.io/upload_images/8760038-4ed267fdda0323cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    ![image.png](https://upload-images.jianshu.io/upload_images/8760038-5dd59dde81863c09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
    查看三张结果图，可以看出每次的Stack deep值都有所不同。究其原因，就需要深入到JVM的源码中才能探讨，这里不作赘述。
    虚拟机栈除了上述错误外，还有另一种错误，那就是当申请不到空间时，会抛出OutOfMemoryError。这里有一个小细节需要注意，catch捕获的是Throwable，而不是Exception，这是因为StackOverflowError和OutOfMemoryError都不属于Exception的子类。

- 本地方法栈（线程私有）  
和虚拟机栈类似，主要为虚拟机使用到的Native方法服务。
也会抛出StackOverflowError和OutOfMemoryError。


- PC寄存器（线程私有）  
PC寄存器，也叫程序计数器。JVM支持多个线程同时运行，每个线程都有自己的程序计数器。倘若当前执行的是JVM方法，则该寄存器中保存当前执行指令的地址；倘若执行的是native方法，则PC寄存器为空。
这个内存区域是唯一一个在虚拟机中没有规定任何OutOfMemoryError情况的区域。

- 堆（线程共享）  
堆内存是JVM所有线程共享的部分，在虚拟机启动的时候就已经创建。
和程序开发密切相关，应用系统对象都保存在Java堆中。所有的对象和数组都在堆上进行分配。这部分空间可通过GC进行回收。
对分代GC来说，堆也是分代的，是GC的主要工作区间。
当申请不到空间时，会抛出OutOfMemoryError。下面我们简单的模拟一个堆内存溢出的情况：
    ```
    package com.paddx.test.memory;
    import java.util.ArrayList;
    import java.util.List;
    /**
     * Created by root on 2/28/17.
     */
    public class HeapOomMock {
        public static void main(String[] args) {
            List<byte[]> list = new ArrayList<byte[]>();
            int i = 0;
            boolean flag = true;
            while(flag) {
                try {
                    i++;
                    list.add(new byte[1024 * 1024]); // 每次增加1M大小的数组对象
                }catch(Throwable e) {
                    e.printStackTrace();
                    flag = false;
                    System.out.println("Count = " + i); // 记录运行的次数
                }
            }
        }
    }
    ```
    首先配置运行时虚拟机的启动参数：  
    ![image.png](https://upload-images.jianshu.io/upload_images/8760038-fcba7402e9643c67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
    然后运行代码，输出结果如下：  
    ![image.png](https://upload-images.jianshu.io/upload_images/8760038-fc81a4361af7456c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
    注意，这里我们指定了堆内存的大小为16M，所以这个地方显示的Count=13(这个数字不是固定的)，至于为什么会是13或其他数字，需要根据GC日志来判断。

- 方法区（线程共享）  
方法区也是所有线程共享的。主要用于存储类的信息、常量池、方法数据、方法代码等。方法区逻辑上属于堆的一部分，但是为了与堆进行区分，通常又叫“非堆”。   
这个区域的内存回收目标主要针对常量池的回收和对类型的卸载。  
当方法区无法满足内存分配需求时，则抛出OutOfMemoryError异常。  
在HotSpot虚拟机中，用永久代来实现方法区，将GC分代收集扩展至方法区，但是这样容易遇到内存溢出的问题。  
JDK1.7中，已经把放在永久代的字符串常量池移到堆中。  
JDK1.8撤销永久代，引入元空间。  


附件（栈、堆、方法区交互）：
![image.png](https://upload-images.jianshu.io/upload_images/8760038-1986aac726fa8f97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参考链接：  
https://segmentfault.com/a/1190000008134012
https://blog.csdn.net/universe_ant/article/details/58585854
https://www.cnblogs.com/mengchunchen/p/7819370.html
