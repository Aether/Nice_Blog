---
layout: post
title: "面向性能的构造技术"
subtitle: ""
date: 2019-06-20
author: Aether
category: coding
tags: CODE SOFTWARECONSTRUCTION JAVA
finished: true
---

## Metrics
### Time performance
-  get execution time

        System.currentTimeMillis()
        
        
- Basic statements
- Algorithms
- Data structure
- I/O (file, database, network communication, etc)
- Concurrency / multi-thread / lock
### Space performance
- Get memory consumption
~~~
// Get the Java runtime
Runtime runtime = Runtime.getRuntime();
// Run the garbage collector
runtime.gc();
// Calculate the used memory
long memory = runtime.totalMemory() - runtime.freeMemory();
~~~
- Algorithms
- Data structure
- Memory allocation
- Garbage collection

******

## Memory Performance and Garbage Collection
### Memory Management in OS and Application Level
- memory allocation 内存分配
- garbage collection 垃圾回收

### Object Model 内存对象模型
- Create a new object x
- Attach it to the reference x;
- Initialize its fields.

### Three modes of object management
- Static mode ——在编译阶段就已经确定好了内存分配
- Dynamic allocation
    - Stack-based mode
        - 存储方法调和局部变量
        - 多线程之间传递数据，是通过复制而非引用局部的基本数据类型
    - Heap-based mode  (free mode)
        - 在一块内存里分为多个小块，每块包含 一个对象，或者未被占用
        - 所有对象
        - 即使是局部变量的object，也是在堆上创建
        - 堆上创建的对象可被所有线程共享引用
        
### Java Memory Model
- Thread Stack --- All local variables of primitive types
- Heap --- All objects

### Memory Structure of JVM
1. Stacks
- Reference/Local variable
    
2. Heap
- Object/Value
- An object's primitive member variables
- a member variable is a reference to an objectf
3. Native Stacks 本地方法栈
- manage native methods (coded in C) used by JVM
4. Program Counter Register (PC)
- 代码行号指示器，用于指示， 跳转下一条需要执行的命令
5. Method Area
- 用于存储被VM加载的类信息、常量、静态变量等 class Static variable
- HotSpot JVM中用Permanent Area (Perm)实现该区域，并作为heap的一部分
- Java 8之后改名为Metaspace (使用native memory)

###  Garbage Collection
1. static modes
无需进行内存回收:所有都是已确定的
2. stack-based modes
在栈上进行内存空间回收:按block(某个方法)整体进行
3. heap-based (free) modes
- Reachable Objects vs. Unreachable Objects --- Live vs. Dead <br/>
- 从root对象开始进行有向图的搜索，将图分为root可达部分和root不可达部分 <br/>
- **roots** 包括：<br/>
1) Words in the static area 静态区域的数据<br/>
2) Registers 寄存器<br/>
3) Words on the execution stack that point into the heap. 目前的执行栈中的数据所指向的内存对象

#### Help the GC that you are finished with an object

    r = new FileReader(filename)
    // use the reader
    ...
    reader.close();
    reader = null;

#### Without Automatic GC
内存泄露，很多dead objects仍占据着内存空间 <br/>
原本live的对象却被回收了 (Dangling Pointers 悬空指针)，导致程序执行失效。

********
## Basic Algorithms of Garbage Collection
### Reference counting 引用计数
> 为每个object存储一个计数RC，当有其他 reference指向它时，RC++;当其他reference与其断开时，RC--;如 果RC==0，则回收它 (及其所有指向的object)。

Advantages
* 简单
* 计算代价分散
* “幽灵时间”短->0

Disadvantages
* 不全面(容易漏掉循环引用的对象cyclic)
* 并发支持较弱
* 占用额外内存空间等

### Mark-Sweep 标记-清除
> 标记-清除:为每个object设定状态位(live/dead)并记录，即mark阶段;将标记为dead的对象进行清理，即sweep阶段。

Disadvantages
* 需要停止程序执行来Mark和Sweep，导致幽灵时间过长，也影响程序本身的性能
* 碎片化

### Mark-Compact 标记-整理
Advantages
* 避免碎片化

Disadvantages
* 时间消耗太长，影响程序本身

###  Fragmentation and Copying 复制

> 该GC策略与mark-compact的区别在于:不是在同一个区域内进行整理，而是将live对象全部复制到另一个区域。<br/> 将堆分为两部分 Fromspace and Tospace

*******

## Garbage Collection in JVM
Java GC将堆分为不同的区域，各区域采用不同的GC策略，以提高GC的效率
~~~
-verbose:gc
~~~

### Three major space in HotSpot VM (Sun JVM)
#### young generation
- Eden
- From survivor space
- To survivor space
- 只有一小部分对象可较长时间存活，故采用**copy**算法减少GC代价
- 使用**minor GC**进行垃圾收集
*  Minor GC所需时间较短
* 如果历经多次minor GC仍存活下来，将其copy到old generation

#### old generation space (tenured)
- 这里的对象有很高的幸存度，使用**Mark-Sweep**或**Mark-Compact**算法
- 启动**full GC**
* Old generation满，意味着无法进行下一次minor GC
* Minor GC和full GC独立进行，减小代价

#### Permanent generation space (perm)
- 当perm generation满了之后，无法存储更多的元数据，也启动full GC

只有当某个区域不能再为对象分配内存时(满)才启动GC
`java.lang.OutOfMemoryError`

*******

## Garbage Collection Tuning in JVM
### Specifying VM heap size 确定堆的大小
#### 控制Young generation的大小
- -XX: NewSize=<n>[g|m|k]
- -XX: MaxNewSize=<n>[g|m|k]
- -Xmn<n>[g|m|k]
-三个参数依次为最小值、最大值和固定值。

- -Xms and -Xmx 初始和最大的heap尺寸
heap尺寸变化时需 要full GC
- -XX:MetaspaceSize -XX:MaxMetaspaceSize for Java 8 and later
- -XX:SurvivorRatio=<n>
- -XX:NewRatio=<n> --- young and old generation is 1:n

old generation的尺寸不需要设置，根据其他各参数的取值可计算得到

### Choosing a garbage collection scheme 选择GC模式
- -XX:+UseSerialGC：使用单线程GC。
- -XX:+UseParallelGC：对MinorGC使用多线程进行回收，但对MajorGC仍采用单线程。
- -XX:+UseConcMarkSweepGC：与程序的运行同步地进行垃圾回收，回收时会短暂地暂停程序。
- -XX:+UseTrainGC：每次MinorGC回收一部分老年代对象，以尝试最小化MajorGC的长时暂停。
#### Garbage-First collector
> designed for heap sizes greater than 4GB. It divides the heap size into regions. After the marking phase is complete, G1 collects the region with lesser live data, which usually yields a large amount of free space. So G1 collects these regions first.
<br/>Default in Java 9.

Automatically logging low memory conditions 自动记录内存将要不足的情况
- -Xloggc:../../logs/gc-console.log

Manually requesting garbage collection 手工请求GC
- System.gc()

******

## I/O and Algorithm Performance
### Buffer：对数据进行缓冲
- BufferedReader
- BufferedWriter

### Stream：将数据视作流
- FileInputStream
- FileOutputStream

### NIO：New I/O
- Buffers（包括CharBuffer、ByteBuffer、IntBuffer、DoubleBuffer等）
- Channels（包括FileChannel、SockerChannel等，可向Buffer提供数据）

### Lamda
~~~
Files.lines(Paths.get(args[0])).forEach(System.out::println);
~~~

***********

## Dynamic Program Analysis Methods and Tools
### 80-20 rule
20% of program responsible for 80% of execution time
### Dynamic Program Analysis
- Program Hot Spots --- 每个程序实体 (语句、分支、路径、方法等)的执行概率/频度是多少
- Path profiling

### Profiling Approaches
- Insertion/Instrumentation 代码注入/代码插入

在原始程序中加入某些语句来收集运行时数据，这些语句不改变原程序的语义，但对原程序的性能有了轻微变化
- Sampling: observation of behavior 采样

以特定的频率观察程序执行的特定时刻所展现出的行为与状态

-  Instrumented virtual machine 借助于虚拟机获取程序性能数据

### Program Profiling Tools in Java

- Command-line profiling tools
    - jhat --- 在Web浏览器中查看heap dump文件的内容
    - jmap --- jmap -dump 生成 java heap dump
     - jstat
     - jstack --- 打印 stack traces
- JConsole
- Visual VM
- Memory Analyzer (MAT) --- 显示堆转储的内容分析

*******

## Code Tuning for Performance Optimization
### Java代码调优的设计模式
#### Singleton Pattern
> 逻辑上仅能存在唯一一个实例，为了防止多个该对象实例被创建，将构造方法设为私有，并提供获取该类唯一实例的方法（ `getInstance()` ）

- static final instance
> 设置静态变量来存储单一实例对象
- private constructor
> 将构造器设置为private，从而client无法new <br/>  在构造器中new新实例

- static getInstance()
> 提供静态方法来获取单一实例对象 <br/>  进一步提升性能:在需要的时候再new，而非提前构造

### Flyweight Pattern
> 当某对象由大量对象组成，而部分Flyweight对象可被共享以减少资源消耗时，可创建一个工厂类以方便外界获得可被共享的对象，对于目前不存在的ConcreteFlyweight对象，工厂类会创建之，而对于已存在的ConcreteFlyweight对象，工厂类会直接返回之。

- 固定 --- 内部rep
- 可变 --- 外部特征
- Immutable

#### Prototype Pattern /cloneable
> 当需要客户端指定要创建的对象的类型，且之后需创建的对象与先前指定的完全相同时，可通过给出第一个对象（称为原型对象）以指明之后所有对象的类型，之后所有对象通过复制第一个对象来创建。

implement `Cloneable`, override `clone()`
- reference copy --- shallow copy
- object copy --- deep copy
    
#### Object Pool Pattern
> 当某类对象（图中为Object类对象）的创建代价极大，而该类对象可以以较小的代价被重复利用时，可以预先建立一个对象池用于保存该类对象。对象池中可预先创建几个对象等待使用，使用完的对象会回到对象池中被重新初始化并被再利用，减少对象的创建操作。

代价:原本可被GC的对象，现在要留在pool中，导致内存浪费 —— 用空间换时间
#### Canonicalizing Objects

### String Constant Pool
 堆中包含字符串对象引用的特殊内存区域
~~~
String s = “java”;
String s = new String(“java”);
~~~
