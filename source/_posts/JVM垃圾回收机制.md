---
title: JVM垃圾回收机制
date: 2018-10-16 20:17:34
categories: 深入JVM内核
tags: [jvm]
---
> Java开发有个很基础的问题，虽然我们平时接触的不多，但是了解它却成为Java开发的必备基础——这就是JVM。  
在Java中JVM内置了垃圾回收的机制，以守护进程的形式在后台自动回收垃圾，它让开发者无需关注空间的创建和释放，帮助开发者承担对象的创建和释放的工作，极大的减轻了开发的负担。那是不是我们就不需要了解JVM了，显然在做一些优化或者深入研究应用性能的时候，JVM还是起了很关键的作用的。因此本篇就总结性的描述下垃圾回收相关的知识。

<!-- more -->

### 哪些内存需要回收
**回收区域主要集中在java堆和方法区。**  
程序计数器、虚拟机栈、本地方法栈3个区域随线程而生，随线程而灭；栈中的栈帧随着方法的进入和退出而有条不紊地执行着出栈和入栈操作。**每一个栈帧中分配多少内存基本上是在类结构确定下来时就已知的，因此这几个区域的内存分配和回收都具备确定性**，所以不需要考虑回收，而Java堆和方法区则不一样，一个接口中的多个实现类需要的内存可能不一样，一个方法中的多个分支需要的内存也可能不一样，我们只有在程序处于运行期间时才能知道会创建哪些对象，这部分内存的分配和回收都是动态的，垃圾收集器所关注的是这部分内存。

### 什么时候回收  
- 对象没有引用 
- 作用域发生未捕获异常 
- 程序在作用域正常执行完毕 
- 程序执行了System.exit() 
- 程序发生意外终止（被杀进程等）

### 如何回收
所谓“垃圾”，就是指所有不再存活的对象。常见的判断是否存活有两种方法：**引用计数法和可达性分析。**
- 引用计数法  
为每一个创建的对象分配一个引用计数器，用来存储该对象被引用的个数。当该个数为零，意味着没有人再使用这个对象，可以认为“对象死亡”。但是，这种方案存在严重的问题，就是无法检测“循环引用”：当两个对象互相引用，即时它俩都不被外界任何东西引用，它俩的计数都不为零，因此永远不会被回收。而实际上对于开发者而言，这两个对象已经完全没有用处了。
因此，Java 里没有采用这样的方案来判定对象的“存活性”。
- 可达性分析  
这种方案是目前主流语言里采用的对象存活性判断方案。基本思路是把所有引用的对象想象成一棵树，从树的根结点 GC Roots 出发，持续遍历找出所有连接的树枝对象，这些对象则被称为“可达”对象，或称“存活”对象。其余的对象则被视为“死亡”的“不可达”对象，或称“垃圾”。
参考下图，object5,object6 和 object7 便是不可达对象，视为“死亡状态”，应该被垃圾回收器回收。
![可达性分析](https://upload-images.jianshu.io/upload_images/8760038-063e47407195c745.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    **可作为GC root的对象**  
    我们可以猜测，GC Roots 本身一定是可达的，这样从它们出发遍历到的对象才能保证一定可达。那么，Java 里有哪些对象是一定可达呢？主要有以下四种：  
    - 虚拟机栈（帧栈中的本地变量表）中引用的对象。
    - 方法区中静态属性引用的对象。
    - 方法区中常量引用的对象。
    - 本地方法栈中 JNI 引用的对象。  
    这里只要知道有这么几种类型的 GC Roots，每次垃圾回收器会从这些根结点开始遍历寻找所有可达节点。

### 垃圾回收算法 
上面已经知道，所有 GC Roots不可达的对象都称为垃圾，参考下图，黑色的表示垃圾，灰色表示存活对象，绿色表示空白空间。
![image](http://upload-images.jianshu.io/upload_images/8760038-dd9a47ead95ebca2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
那么，我们如何来回收这些垃圾呢？

- Mark-Sweep标记-清除算法   
第一步，所谓“标记”就是利用可达性遍历堆内存，把“存活”对象和“垃圾”对象进行标记，得到的结果如上图；  
第二步，既然“垃圾”已经标记好了，那我们再遍历一遍，把所有“垃圾”对象所占的空间直接清空即可。结果如下：
![Mark-Sweep标记-清除算法](http://upload-images.jianshu.io/upload_images/8760038-5686115b38932e8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这便是“标记－清理”方案，简单方便 ，但是容易产生内存碎片。

- Mark-Compact标记-整理算法  
既然上面的方法会产生内存碎片，那好，我在清理的时候，把所有 存活 对象扎堆到同一个地方，让它们待在一起，这样就没有内存碎片了。
结果如下：
![Mark-Compact标记-整理算法](http://upload-images.jianshu.io/upload_images/8760038-8c8a286138015d0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这两种方案适合存活对象多，垃圾少的情况，它只需要清理掉少量的垃圾，然后挪动下存活对象就可以了。

- Copying复制算法  
这种方法比较粗暴，直接把堆内存分成两部分，一段时间内只允许在其中一块内存上进行分配，当这块内存被分配完后，则执行垃圾回收，把所有存活对象全部复制到另一块内存上，当前内存则直接全部清空。  
参考下图：
![Copying复制算法](http://upload-images.jianshu.io/upload_images/8760038-8211a4a8ba58067d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
起初时只使用上面部分的内存，直到内存使用完毕，才进行垃圾回收，把所有存活对象搬到下半部分，并把上半部分进行清空。  
这种做法不容易产生碎片，也简单粗暴；但是，它意味着你在一段时间内只能使用一部分的内存，超过这部分内存的话就意味着堆内存里频繁的 复制清空。  
这种方案适合 存活对象少，垃圾多   的情况，这样在复制时就不需要复制多少对象过去，多数垃圾直接被清空处理。  

- Generational Collection 分代收集  
最后的这种方法是前面几种的合体，即目前JVM主要采取的一种方法，思想就是把JVM分成不同的区域。每种区域使用不同的垃圾回收方法。 

    ![分代收集](http://upload-images.jianshu.io/upload_images/8760038-5c70ecc1228bd230.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

    上面可以看到堆分成三个区域：  
    **新生代(Young Generation)**：用于存放新创建的对象，采用复制回收方法，如果在s0和s1之间复制一定次数后，转移到年老代中。这里的垃圾回收叫做minor GC;  
    **年老代(Old Generation)**：这些对象垃圾回收的频率较低，采用的标记整理方法，这里的垃圾回收叫做major GC。   
    **永久代(Permanent Generation)**：存放Java本身的一些数据，当类不再使用时，也会被回收。  
    
    这里可以详细的说一下新生代复制回收的算法流程：  
    在新生代中，分为三个区：Eden, from survivor, to survior。  
    - 当触发minor GC时，会先把Eden中存活的对象复制到to Survivor中；
    - 然后再看from survivor，如果次数达到年老代的标准，就复制到年老代中；如果没有达到则复制到to - survivor中，如果to survivor满了，则复制到年老代中。
    - 然后调换from survivor 和 to survivor的名字，保证每次to survivor都是空的等待对象复制到那里的。

### 垃圾回收器
![HotSpot 虚拟机的垃圾收集器](http://upload-images.jianshu.io/upload_images/8760038-d1e46537466e954e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 串行收集器 Serial  
这种收集器就是以单线程的方式收集，垃圾回收的时候其他线程也不能工作。
![串行收集器 Serial](http://upload-images.jianshu.io/upload_images/8760038-e69dc04894ce9875.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 并行收集器 Parallel  
以多线程的方式进行收集  
![并行收集器 Parallel](http://upload-images.jianshu.io/upload_images/8760038-83a273bb6eed2133.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 并发标记清除收集器 Concurrent Mark Sweep Collector, CMS  
大致的流程为：初始标记--并发标记--重新标记--并发清除  
![并发标记清除收集器 Concurrent Mark Sweep Collector, CMS](http://upload-images.jianshu.io/upload_images/8760038-420c265247c679bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- G1收集器 Garbage First Collector  
大致的流程为：初始标记--并发标记--最终标记--筛选回收  
![G1收集器 Garbage First Collector](http://upload-images.jianshu.io/upload_images/8760038-5271671c121bf143.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### GC什么时候触发
由于对象进行了分代处理，因此垃圾回收区域、时间也不一样。GC有两种类型：Scavenge GC和Full GC。

- Scavenge GC  
一般情况下，当新对象生成，并且在Eden申请空间失败时，就会触发Scavenge GC，对Eden区域进行GC，清除非存活对象，并且把尚且存活的对象移动到Survivor区。然后整理Survivor的两个区。这种方式的GC是对年轻代的Eden区进行，不会影响到年老代。因为大部分对象都是从Eden区开始的，同时Eden区不会分配的很大，所以Eden区的GC会频繁进行。因而，一般在这里需要使用速度快、效率高的算法，使Eden去能尽快空闲出来。

- Full GC  
对整个堆进行整理，包括Young、Tenured和Perm。Full GC因为需要对整个堆进行回收，所以比Scavenge GC要慢，因此应该尽可能减少Full GC的次数。在对JVM调优的过程中，很大一部分工作就是对于Full GC的调节。有如下原因可能导致Full GC：  
    - 年老代（Tenured）被写满
    - 持久代（Perm）被写满 
    - System.gc()被显示调用  
    - 上一次GC之后Heap的各域分配策略动态变化 
    
参考链接：  
http://www.importnew.com/26821.html  
https://www.cnblogs.com/xing901022/p/7725961.html  
https://www.cnblogs.com/1024Community/p/honery.html  
https://blog.csdn.net/sinat_33087001/article/details/77030118  
