---
title: java虚拟机的锁优化技术
date: 2020-03-21 09:19:32
tags: java
---
# java虚拟机的锁优化技术
> 这篇文章也是从hollis那里看到的，是一个系列文章之一。其中未转载的文章是因为看了以后不太明白，就没有记录。

<!--more-->

## 原文前言

### Synchronized的实现原理
1、同步方法通过ACC_SYNCHRONIZED关键字隐式的对方法进行加锁。当线程要执行的方法被标注上ACC_SYNCHRONIZED时，需要先获得锁才能执行该方法。《深入理解多线程（一）——Synchronized的实现原理》

> 上面这个文章，去除了源码和反编译部分，已经将作者的总结放在了上一篇《从StringBuffer和StringBuilder扩展开的问题》文章中。

### Moniter的实现原理
2、同步代码块通过monitorenter和monitorexit执行来进行加锁。当线程执行到monitorenter的时候要先获得所锁，才能执行后面的方法。当线程执行到monitorexit的时候则要释放锁。《深入理解多线程（四）—— Moniter的实现原理》

> 这篇文章，我将`moniter`的概念放在了我的那篇文章中。具体的源码以及流程图并没有拿过来。

#### 原文摘抄
这里再把原文中的流程复制过来记录一下：

ObjectMonitor中有几个关键属性：
>_owner：指向持有ObjectMonitor对象的线程
>
>_WaitSet：存放处于wait状态的线程队列
>
>_EntryList：存放处于等待锁block状态的线程队列
>
>_recursions：锁的重入次数
>
>_count：用来记录该线程获取锁的次数

当多个线程同时访问一段同步代码时，首先会进入`_EntryList`队列中，当某个线程获取到对象的monitor后进入`_Owner`区域并把monitor中的`_owner`变量设置为当前线程，同时monitor中的计数器`_count`加1。即获得对象锁。

若持有monitor的线程调用wait()方法，将释放当前持有的monitor，`_owner`变量恢复为null，`_count`自减1，同时该线程进入`_WaitSet`集合中等待被唤醒。若当前线程执行完毕也将释放monitor(锁)并复位变量的值，以便其他线程进入获取monitor(锁)。如下图所示：
![](b1.png)

ObjectMonitor类中提供了几个方法：
##### 获得锁：
```java
void ATTR ObjectMonitor::enter(TRAPS) {
  Thread * const Self = THREAD ;
  void * cur ;
  //通过CAS尝试把monitor的`_owner`字段设置为当前线程
  cur = Atomic::cmpxchg_ptr (Self, &_owner, NULL) ;
  //获取锁失败
  if (cur == NULL) {
  	 assert (_recursions == 0   , "invariant") ;
     assert (_owner      == Self, "invariant") ;
     // CONSIDER: set or assert OwnerIsThread == 1
     return ;
  }
  // 如果旧值和当前线程一样，说明当前线程已经持有锁，此次为重入，_recursions自增，并获得锁。
  if (cur == Self) { 
     // TODO-FIXME: check for integer overflow!  BUGID 6557169.
     _recursions ++ ;
     return ;
  }

  // 如果当前线程是第一次进入该monitor，设置_recursions为1，_owner为当前线程
  if (Self->is_lock_owned ((address)cur)) { 
    assert (_recursions == 0, "internal state error");
    _recursions = 1 ;
    // Commute owner from a thread-specific on-stack BasicLockObject address to
    // a full-fledged "Thread *".
    _owner = Self ;
    OwnerIsThread = 1 ;
    return ;
  }

  // 省略部分代码。
  // 通过自旋执行ObjectMonitor::EnterI方法等待锁的释放
  for (;;) {
      jt->set_suspend_equivalent();
      // cleared by handle_special_suspend_equivalent_condition()
      // or java_suspend_self()

      EnterI (THREAD) ;

      if (!ExitSuspendEquivalent(jt)) break ;

      //
      // We have acquired the contended monitor, but while we were
      // waiting another thread suspended us. We don't want to enter
      // the monitor while suspended because that would surprise the
      // thread that suspended us.
      //
          _recursions = 0 ;
      _succ = NULL ;
      exit (Self) ;

      jt->java_suspend_self();
	}
}
```

![获得锁](b2.png)

##### 释放锁：
```java
void ATTR ObjectMonitor::exit(TRAPS) {
   Thread * Self = THREAD ;
   //如果当前线程不是Monitor的所有者
   if (THREAD != _owner) { 
     if (THREAD->is_lock_owned((address) _owner)) { // 
       // Transmute _owner from a BasicLock pointer to a Thread address.
       // We don't need to hold _mutex for this transition.
       // Non-null to Non-null is safe as long as all readers can
       // tolerate either flavor.
       assert (_recursions == 0, "invariant") ;
       _owner = THREAD ;
       _recursions = 0 ;
       OwnerIsThread = 1 ;
     } else {
       // NOTE: we need to handle unbalanced monitor enter/exit
       // in native code by throwing an exception.
       // TODO: Throw an IllegalMonitorStateException ?
       TEVENT (Exit - Throw IMSX) ;
       assert(false, "Non-balanced monitor enter/exit!");
       if (false) {
          THROW(vmSymbols::java_lang_IllegalMonitorStateException());
       }
       return;
     }
   }
    // 如果_recursions次数不为0.自减
   if (_recursions != 0) {
     _recursions--;        // this is simple recursive enter
     TEVENT (Inflated exit - recursive) ;
     return ;
   }

   //省略部分代码，根据不同的策略（由QMode指定），从cxq或EntryList中获取头节点，通过ObjectMonitor::ExitEpilog方法唤醒该节点封装的线程，唤醒操作最终由unpark完成。

```
![](b3.png)

#### HotSpot虚拟机中Moniter的的加锁以及解锁的原理总结
通过这篇文章我们知道了sychronized加锁的时候，会调用objectMonitor的enter方法，解锁的时候会调用exit方法。事实上，只有在JDK1.6之前，synchronized的实现才会直接调用ObjectMonitor的enter和exit，这种锁被称之为重量级锁。为什么说这种方式操作锁很重呢？

- Java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统的帮忙，这就要从用户态转换到核心态，因此状态转换需要花费很多的处理器时间，对于代码简单的同步块（如被synchronized修饰的get 或set方法）状态转换消耗的时间有可能比用户代码执行的时间还要长，所以说synchronized是java语言中一个重量级的操纵。

所以，在JDK1.6中出现对锁进行了很多的优化，进而出现轻量级锁，偏向锁，锁消除，适应性自旋锁，锁粗化(自旋锁在1.4就有 只不过默认的是关闭的，jdk1.6是默认开启的)，这些操作都是为了在线程之间更高效的共享数据 ，解决竞争问题。后面的文章会继续介绍这几种锁以及他们之间的关系。

### Java的对象模型
3、在HotSpot虚拟机中，使用oop-klass模型来表示对象。每一个Java类，在被JVM加载的时候，JVM会给这个类创建一个instanceKlass，保存在方法区，用来在JVM层表示该Java类。当我们在Java代码中，使用new创建一个对象的时候，JVM会创建一个instanceOopDesc对象，这个对象中包含了对象头以及实例数据。《深入理解多线程（二）—— Java的对象模型》
> 这里先略过原文复制，有点没看懂原文作者写的东西，之后找别的资料互相理解后加上。

### java的对象头
4、对象头中主要包含了GC分代年龄、锁状态标记、哈希码、epoch等信息。对象的状态一共有五种，分别是无锁态、轻量级锁、重量级锁、GC标记和偏向锁。《深入理解多线程（三）—— Java的对象头》
> 这里是和上面关联的，我需要阅读一下书。再来补充

## 正文
事实上，只有在JDK1.6之前，synchronized的实现才会直接调用ObjectMonitor的enter和exit，这种锁被称之为重量级锁。

高效并发是从JDK 1.5 到 JDK 1.6的一个重要改进，HotSpot虚拟机开发团队在这个版本中花费了很大的精力去对Java中的锁进行优化，如适应性自旋、锁消除、锁粗化、轻量级锁和偏向锁等。这些技术都是为了在线程之间更高效的共享数据，以及解决竞争问题。

本文，主要先来介绍一下自旋、锁消除以及锁粗化等技术。

这里简单说明一下，本文要介绍的这几个概念，以及后面要介绍的轻量级锁和偏向锁，其实对于使用他的开发者来说是屏蔽掉了的，也就是说，作为一个Java开发，你只需要知道你想在加锁的时候使用synchronized就可以了，具体的锁的优化是虚拟机根据竞争情况自行决定的。

也就是说，在JDK 1.5 以后，我们即将介绍的这些概念，都被封装在synchronized中了。

### 线程状态
要想把锁说清楚，一个重要的概念不得不提，那就是线程和线程的状态。锁和线程的关系是怎样的呢，举个简单的例子你就明白了。

比如，你今天要去银行办业务，你到了银行之后，要先取一个号，然后你坐在休息区等待叫号，过段时间，广播叫到你的号码之后，会告诉你去哪个柜台办理业务，这时，你拿着你手里的号码，去到对应的柜台，找相应的柜员开始办理业务。当你办理业务的时候，这个柜台和柜台后面的柜员只能为你自己服务。当你办完业务离开之后，广播再喊其他的顾客前来办理业务。
![](b4.png)
> 这个例子中，每个顾客是一个线程。 柜台前面的那把椅子，就是锁。 柜台后面的柜员，就是共享资源。 你发现无法直接办理业务，要取号等待的过程叫做阻塞。 当你听到叫你的号码的时候，你起身去办业务，这就是唤醒。 当你坐在椅子上开始办理业务的时候，你就获得锁。 当你办完业务离开的时候，你就释放锁。

对于线程来说，一共有五种状态，分别为：初始状态(New) 、就绪状态(Runnable) 、运行状态(Running) 、阻塞状态(Blocked) 和死亡状态(Dead) 。

![](b5.png)
> 想想第一次听说这个概念还是在学校的时候，讲到计算机组成原理

### 自旋锁
在[深入理解多线程（四）—— Moniter的实现原理](http://www.hollischuang.com/archives/2030)文章中，我们介绍的synchronized的实现方式中使用Monitor进行加锁，这是一种互斥锁，为了表示他对性能的影响我们称之为重量级锁。

这种互斥锁在互斥同步上对性能的影响很大，Java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统的帮忙，这就要从用户态转换到内核态，因此状态转换需要花费很多的处理器时间。

就像去银行办业务的例子，当你来到银行，发现柜台前面都有人的时候，你需要取一个号，然后再去等待区等待，一直等待被叫号。这个过程是比较浪费时间的，那么有没有什么办法改进呢？

有一种比较好的设计，那就是银行提供自动取款机，当你去银行取款的时候，你不需要取号，不需要去休息区等待叫号，你只需要找到一台取款机，排在其他人后面等待取款就行了。
![形象的图](b6.png)

之所以能这样做，是因为取款的这个过程相比较之下是比较节省时间的。如果所有人去银行都只取款，或者办理业务的时间都很短的话，那也就可以不需要取号，不需要去单独的休息区，不需要听叫号，也不需要再跑到对应的柜台了。

而在程序中，Java虚拟机的开发工程师们在分析过大量数据后发现：共享数据的锁定状态一般只会持续很短的一段时间，为了这段时间去挂起和恢复线程其实并不值得。

如果物理机上有多个处理器，可以让多个线程同时执行的话。我们就可以让后面来的线程“稍微等一下”，但是并不放弃处理器的执行时间，看看持有锁的线程会不会很快释放锁。这个“稍微等一下”的过程就是自旋。

自旋锁在JDK 1.4中已经引入，在JDK 1.6中默认开启。

很多人在对于自旋锁的概念不清楚的时候可能会有以下疑问：这么听上去，自旋锁好像和阻塞锁没啥区别，反正都是等着嘛。

- 对于去银行取钱的你来说，站在取款机面前等待和去休息区等待叫号有一个很大的区别：

	- 那就是如果你在休息区等待，这段时间你什么都不需要管，随意做自己的事情，等着被唤醒就行了。

	- 如果你在取款机面前等待，那么你需要时刻关注自己前面还有没有人，因为没人会唤醒你。

	- 很明显，这种直接去取款机前面排队取款的效率是比较高。

所以呢，自旋锁和阻塞锁最大的区别就是，到底要不要放弃处理器的执行时间。对于阻塞锁和自旋锁来说，都是要等待获得共享资源。但是阻塞锁是放弃了CPU时间，进入了等待区，等待被唤醒。而自旋锁是一直“自旋”在那里，时刻的检查共享资源是否可以被访问。

由于自旋锁只是将当前线程不停地执行循环体，不进行线程状态的改变，所以响应速度更快。但当线程数不停增加时，性能下降明显，因为每个线程都需要执行，占用CPU时间。如果线程竞争不激烈，并且保持锁的时间段。适合使用自旋锁。

### 锁消除
除了自旋锁之后，JDK中还有一种锁的优化被称之为锁消除。还拿去银行取钱的例子说。

你去银行取钱，所有情况下都需要取号，并且等待吗？其实是不用的，当银行办理业务的人不多的时候，可能根本不需要取号，直接走到柜台前面办理业务就好了。
![形象的图](b7.png)
能这么做的前提是，没有人和你抢着办业务。

上面的这种例子，在锁优化中被称作“锁消除”，是JIT编译器对内部锁的具体实现所做的一种优化。

在动态编译同步块的时候，JIT编译器可以借助一种被称为逃逸分析（Escape Analysis）的技术来判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程。

如果同步块所使用的锁对象通过这种分析被证实只能够被一个线程访问，那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。

如以下代码：
```java
public void f() {
    Object hollis = new Object();
    synchronized(hollis) {
        System.out.println(hollis);
    }
}

```
代码中对hollis这个对象进行加锁，但是hollis对象的生命周期只在f()方法中，并不会被其他线程所访问到，所以在JIT编译阶段就会被优化掉。优化成：
```java
public void f() {
    Object hollis = new Object();
    System.out.println(hollis);
}
```
> 这里，可能有读者会质疑了，代码是程序员自己写的，程序员难道没有能力判断要不要加锁吗？就像以上代码，完全没必要加锁，有经验的开发者一眼就能看的出来的。其实道理是这样，但是还是有可能有疏忽，比如我们经常在代码中使用StringBuffer作为局部变量，而StringBuffer中的append是线程安全的，有synchronized修饰的，这种情况开发者可能会忽略。这时候，JIT就可以帮忙优化，进行锁消除。

总之，读者只需要知道，在使用synchronized的时候，如果JIT经过逃逸分析之后发现并无线程安全问题的话，就会做锁消除。

### 锁粗化
很多人都知道，在代码中，需要加锁的时候，我们提倡尽量减小锁的粒度，这样可以避免不必要的阻塞。

这也是很多人原因是用同步代码块来代替同步方法的原因，因为往往他的粒度会更小一些，这其实是很有道理的。

还是我们去银行柜台办业务，最高效的方式是你坐在柜台前面的时候，只办和银行相关的事情。如果这个时候，你拿出手机，接打几个电话，问朋友要往哪个账户里面打钱，这就很浪费时间了。最好的做法肯定是提前准备好相关资料，在办理业务时直接办理就好了。

加锁也一样，把无关的准备工作放到锁外面，锁内部只处理和并发相关的内容。这样有助于提高效率。

那么，这和锁粗化有什么关系呢？可以说，大部分情况下，减小锁的粒度是很正确的做法，只有一种特殊的情况下，会发生一种叫做锁粗化的优化。

就像你去银行办业务，你为了减少每次办理业务的时间，你把要办的五个业务分成五次去办理，这反而适得其反了。因为这平白的增加了很多你重新取号、排队、被唤醒的时间。

如果在一段代码中连续的对同一个对象反复加锁解锁，其实是相对耗费资源的，这种情况可以适当放宽加锁的范围，减少性能消耗。

当JIT发现一系列连续的操作都对同一个对象反复加锁和解锁，甚至加锁操作出现在循环体中的时候，会将加锁同步的范围扩散（粗化）到整个操作序列的外部。

如以下代码：
```java
for(int i=0;i<100000;i++){  
    synchronized(this){  
        do();  
}  
```
会被粗化成：
```java
synchronized(this){  
    for(int i=0;i<100000;i++){  
        do();  
}  
```
这其实和我们要求的减小锁粒度并不冲突。减小锁粒度强调的是不要在银行柜台前做准备工作以及和办理业务无关的事情。而锁粗化建议的是，同一个人，要办理多个业务的时候，可以在同一个窗口一次性办完，而不是多次取号多次办理。

## 总结
自Java 6/Java 7开始，Java虚拟机对内部锁的实现进行了一些优化。这些优化主要包括锁消除（Lock Elision）、锁粗化（Lock Coarsening）、偏向锁（Biased Locking）以及适应性自旋锁（Adaptive Locking）。这些优化仅在Java虚拟机server模式下起作用（即运行Java程序时我们可能需要在命令行中指定Java虚拟机参数“-server”以开启这些优化）。

本文主要介绍了自旋锁、锁粗化和锁消除的概念。在JIT编译过程中，虚拟机会根据情况使用这三种技术对锁进行优化，目的是减少锁的竞争，提升性能。

其实原本不想写这么多，因为很容易让我自己在看的时候找不到重点。但是现在我还是需要这些来想明白整个过程。大概之后理解的深了可以精简一下吧



> 题外话
>
> 记录一下面试题
>
>  小米二面（1h30min）
> 1.怼项目，各种细节，扩展各个功能怎么做？
> 2.redis有哪几种数据类型，各有什么特点，各有哪些应用场景
> 3.redis的持久化哪几种？分别讲一下RDB和AOF的原理和AOF重写的过程
> 4.redis分布式锁哪几种？分别讲一下setnx和redlock原理？setnx怎么释放锁？释放锁的LUA脚本会写吗？
> 5.redis的主从复制了解吗？
> 6.mysql的隔离级别哪几种？分别讲一下原理
> 7.讲一下mysql的索引？B+树和B树有什么区别？为什么要用B+树而不用B树？B+树的查询时间复杂度多少
> 8.并发和并行的区别
> 9.volatile原理？他可以同步操作吗？它用在什么地方？
> 10.了解什么消息队列？讲一下rabbitmq的原理？怎么保证它消息不丢失和不重复消费？
> 11.讲一下分布式事务
> \12. zookeeper了解吗？讲一下它的原理？哪些应用场景？它怎么实现分布式锁
> 13.讲一下2pc、3pc、CAP理论、BASE理论、paxos算法、raft算法
> 14.写代码：单链表交换相邻的两个节点 

