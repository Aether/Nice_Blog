---
layout: post
title: "软件构造课程总结——并行与分布式程序"
subtitle: ""
date: 2019-06-18
author: Aether
category: coding
tags: SOFTWARE-CONSTRUCTION JAVA
finished: true
---

## Concurrency
###  Two Models for Concurrent Programming
- Shared memory --- 并发模块通过在内存中读写共享对象进行交互。
- Message passing --- sending messages to each other through a communication channel. Modules send off messages, and incoming messages to each module are queued up for handling.

### 应用
- 网络上的两台计算机通过网络连接通讯
- 浏览器和Web服务器，A请求页面，B发送页面数据给A
- 即时通讯软件的客户端和服务器
- 同一台计算机上的两个程序通过管道连接进行通讯

## Process & Thread
### Process
- 在同一机器上独立于其它进程的一个运行中程序的实例。它拥有自己独立的内存空间。可视为一台虚拟机。
-  拥有整台计算机的资源
-  私有空间 彼此隔离 不共享内存
-  通过消息传递进行协作
-  JVM通常运行单一进程，但也可以创建新的进程
-  共享可变对象时需要同步

### Thread
-  一个运行中程序的控制轨迹，每个线程在程序中某个位置执行，拥有自己独立的运行时栈。可视为一个虚拟处理器。
-  共享内存 消息队列
-  轻量级
-  共享可变对象时需要同步
-  unsafe to kill

#### new Thread

- (Seldom used) Subclassing Thread.
~~~
class MyThread extends Thread {
  @Override
  public void run() {
    ...
  }
}
(new MyThread()).start();
~~~
~~~
(new MyThread()).run();可运行，未启动线程
~~~
- (More generally used) 从Runnable接口构造Thread对象
~~~
new Thread(new Runnable() {
   public void run() {
       System.out.println("Hello");
   }
}).start();
~~~
~~~
class MyThread implements Runnable {
  @Override
  public void run() {
    ...
  }
(new Thread(new MyThread())).start();
~~~

### Heisenbugs
Nondeterministic and hard to reproduce <br/>
A heisenbug may even disappear when you try to look at it with println or debugger
### Bohrbugs
Showing up repeatedly whenever you look at it.
## Interleaving and Race Condition
### Time slicing
- 在单核计算机中，每一时刻仅有一个线程在实际运行。单个核心的运行时间通过操作系统的特性被各个进程和线程共享，这种行为称为时间分片。
- 在多核计算机中，当线程数大于核心数时，也通过时间分片来模拟线程并行运行。
- 时间切片不可预测、不确定，这意味着线程可以随时暂停或恢复

### Shared Memory among Threads
- Interleaving 交叉存取 --- 当两线程并行运行时，底层指令可能交错运行

### Race Condition
- The correctness of the program (the satisfaction of postconditions and invariants) depends on the relative timing of events in concurrent computations A and B. When this happens, we say “A is in a race with B.”
- 单行、单条语句都未必是原子的。是否原子由JVM确定
- also called “Thread Interference”

## Some operations for interfering automatic interleaving of threads
### Thread.sleep()
- 不会失去对现有monitor锁的所有权
- 在sleep()的时候检测是否收到别人的中断信号, catch(IntterruptedException e)

### Thread.interrupt()
- t.interrupt()
- t.isInterrupted()
-正常运行期间，即使受到中断信号也不理会

###  Thread.yield() (避免使用)
- 线程会告诉调度器可以放弃CPU的占用权, 从而可能引起调度器唤醒其他线程

### Thread.join()
- 让当前线程保持执行，直到其执行结束

### object.wait()
- 使用前提为线程已获得被调用对象的对象锁。当前线程放弃对象锁并进入等待状态，直到其它获得该对象锁的线程对该对象调用notify()或notifyAll()方法。当前线程被通知或被中断时，尝试重新获得锁并继续运行。若被中断，InterruptedException将在获得对象锁之后抛出。

## Thread Safety
### Confinement
- Don’t share: isolate mutable state in individual threads
- 将可变数据限制在单一线程内部，避免竞争
- 不允许任何线程直接读写该数据
- 避免全局变量 如Singleton设计模式 两个线程同时调用getInstance()违背设计

### Immutability
- Don’t mutate: share only immutable state
- 共享不可变的引用和数据类型
- 如果使用了beneficent mutation还是要加锁
-  不含mutator方法
-  所有字段必须被private final修饰
-  子类不可重写方法
-  没有表示泄露
-  表示中的可变对象不得变化

### Using Threadsafe Data Types
-  Threadsafe Collections
-  private static Map<Integer,Boolean> cache = Collections.synchronizedMap(new HashMap<>());
-  多个操作间不保证安全 如(!list.isEmpty()) list.get(0)

### Locks and Synchronization
-  在使用synchronizedMap(hashMap)之后，不要再把参数hashMap共享给其他线程，不要保留别名，一定要彻底销毁
-  即使在线程安全的集合类上，使用iterator也是不安全的（除非使用lock机制）
-  即使是线程安全的collection类，仍可能产生竞争

### Volatile
-  volatile也可保证基本类型变量变化对于所有线程的可见性（同步缓存），但不一定保证操作的原子性。
-  当且仅当对volatile修饰的变量进行的修改与任何变量（包括自身）的值无关时，其才是线程安全的。

### 线程安全策略撰写
-  采取了四种方法中的哪一种
-  如果是后两种，还需考虑对数据的访问 都是原子的，不存在interleaving

#### Confinement
-  局限访问策略通常不适用于仅仅讨论一个数据类型的情况，因为你需要知道系统中存在哪些线程，每个线程可访问哪些对象。
-  若访问该数据类型的线程均由自身创建，则可讨论这些线程间如何保持访问的局限性
-  若线程来自于外部，则无法讨论
-  通常在将系统作为一个整体，讨论一些模块或数据类型不需要线程安全的原因时，会从设计的角度论证线程间不会共享数据。

#### Immutability
-  说明对象中数据为何能够保持不可变性即可

#### Using Threadsafe Data Types
-  表明该数据类型的所有原子操作都被恰当地设计，使得该数据类型依赖的不变式不受原子操作的交错执行影响。
-  nodes和edges举例

~~~
private final Set<Node> nodes = Collections.synchronizedSet(new HashSet<>());
private final Map<Node,Set<Node>> edges = Collections.synchronizedMap(new HashMap<>());
// 表示不变式：
// 对于所有满足y是edges.get(x)的成员的二元组x和y，x,y都是nodes的成员。
该类是线程安全的，因为：nodes和edges都声明为final，因此它们都是不可变的且是线程安全的。
~~~

> nodes和edges都指向线程安全的Set和Map数据类型。
该论证仅防止了某些竞争条件，但不是全部。由于该类的表示不变式中要求了nodes与edges成员间的关系，因此还应论证两者成员的改变应当是同时的、原子的。若该点无法证明，则可以构造竞争条件。

#### Locks and Synchronization
-  表明该数据类型的所有原子操作都被恰当地设计，使得该数据类型依赖的不变式不受原子操作的交错执行影响。

## Locks and Synchronization
### Monitor模式:ADT所有方法都是互斥访问
-  reads must be guarded as well as writes
-  public synchronized void fun(){…} == synchronized(this)

### fine-grained synchronization.
synchronized(object)
~~~
public synchronized Constructor(){
  ...
} // wrong
~~~
-  用ADT自己做lock

### Lock
~~~
Object lock = new Object();
synchronized (lock) {
  ...
}
~~~

## Liveness
### Deadlock
多个线程竞争lock，相互等待对方释放lock

#### 解决方案：

1. 两线程均按顺序征用锁-两个线程均先获得A锁再获得B锁
    - 缺点1:不是模块化的——代码必须知道系统中的所有锁，或者至少知道子系统中的所有锁
    - 缺点2:代码可能很难或不可能在获取第一个锁之前准确地知道它需要哪些锁 <br/>
    
2. 粗粒度的lock
    - 缺点：性能下降
    
### Starvation
-  因为其他线程lock时间太长，一个线程长时间无法获取其所需的资源访问权(lock)，导致无法往下进行

### Livelock
-  两线程的行为互相由对方的行为决定（通常是为了避免死锁），线程没有锁死，仍在正常运行，但无法正常工作。
