# 1 线程安全

我们可以将Java语言中各种操作共享的数据分为以下5类：

- 不可变
- 绝对线程安全
- 相对线程安全
- 线程兼容
- 线程对立



### 1.1 不可变

不可变的对象一定是线程安全的。举例：

1. 用final修饰的基本数据类型

2. 若共享数据是一个对象，那就需要保证对象的行为不会对其状态产生任何影响才行，如java.lang.String类的对象，它是一个典型的不可变对象，我们调用它的substring()、replace()和concat()这些方法都不会影响它原来的值，只会返回一个新构造的字符串对象。

   保证对象行为不影响自己状态的途径有很多种，其中最简单的就是把对象中带有状态的变量都声明为final，这样在构造函数结束之后，它就是不可变的，  如下代码：

   JDK中Integer类的构造函数

```
private final int value;

public Integer(int value) {
  this.value = value;
}

```

3. 枚举类型
4. java.lang.Number的部分子类，如Long和Double等数值包装类型，BigInteger和BigDecimal等大数据类型；但同为Number的子类型的原子类AtomicInteger和AtomicLong则并非不可变的。



###1.2 绝对线程安全

java中标注自己是线程安全的类，大多数都不是绝对的线程安全。



### 1.3 相对线程安全

它需要保证对这个对象单独的操作是线程安全的，我们在调用的时候不需要做额外的保障措施，但是对于一些特定顺序的连续调用，就可能需要在调用端使用额外的同步手段来保证调用的正确性。如Vector、HashTable、Collections的synchronizedCollection()方法包装的集合等。





### 1.4 线程兼容

 线程兼容是指对象本身并不是线程安全的，但是可以通过在调用端正确地使用同步手段来保证对象在并发环境中可以安全地使用，我们平常说一个类不是线程安全的，绝大多数时候指的是这一种情况。Java API中大部分的类都是属于线程兼容的，如与前面的Vector和HashTable相对应的集合类ArrayList和HashMap等。 



### 1.5 线程对立 

线程对立是指无论调用端是否采取了同步措施，都无法在多线程环境中并发使用的代码。由于Java语言天生就具备多线程特性，线程对立这种排斥多线程的代码是很少出现的，而且通常都是有害的，应当尽量避免。





# 2 可重入锁

指的是同一线程外层函数获得锁之后，内层递归函数仍然能获取该锁的代码，在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁，也就是说，线程可以进入任何一个它已经拥有的锁所同步着的代码块

**可重入锁最大的作用是避免死锁**

eg：

sychronized和ReentrantLock 



# 3 公平锁&非公平锁

### 3.1 公平锁（Fair） 

加锁前检查是否有排队等待的线程，优先排队等待的线程，先来先得 

### 3.2 非公平锁（Nonfair） 

加锁时不考虑排队等待问题，直接尝试获取锁，获取不到自动到队尾等待 

1. 非公平锁性能比公平锁高 5~10 倍，因为公平锁需要在多核的情况下维护一个队列 
2. Java 中的 `synchronized 是非公平锁`，`ReentrantLock 默认的 lock()方法采用的是非公平锁`，但是也可以启用公平锁。



# 4 共享锁&独占锁&互斥锁 

java 并发包提供的加锁模式分为独占锁和共享锁

### 4.1 独占锁(写锁)

独占锁模式下，每次只能有一个线程能持有锁，**ReentrantLock 和sychronized就是以独占方式实现的互斥锁**。 

独占锁是一种悲观保守的加锁策略，它避免了读/读冲突，如果某个只读线程获取锁，则其他读线程都只能等待，这种情况下就限制了不必要的并发性，因为读操作并不会影响数据的一致性。 



### 4.2 共享锁(读锁)

共享锁则允许多个线程同时获取锁，并发访问共享资源，如：ReadWriteLock。共享锁则是一种乐观锁，它放宽了加锁策略，允许多个执行读操作的线程同时访问共享资源。 

1. AQS 的内部类 Node 定义了两个常量 SHARED 和 EXCLUSIVE，他们分别标识 AQS 队列中等待线程的锁获取模式。 
2. java 的并发包中提供了 **ReadWriteLock，读-写锁**。它允许一个资源可以被多个读操作访问， 或者被一个写操作访问，但两者不能同时进行。



### 4.3 应用

[博客](https://www.cnblogs.com/xiaoxi/p/9140541.html)

读写锁**ReentrantReadWriteLock** 

它表示两个锁，一个是读操作相关的锁，称为共享锁；一个是写相关的锁，称为排他锁 ,读写、写读、写写的过程是互斥的。

读写锁有以下三个重要的特性：

（1）公平选择性：支持非公平（默认）和公平的锁获取方式，吞吐量还是非公平优于公平。

（2）重进入：读锁和写锁都支持线程重进入。

（3）锁降级：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级成为读锁。



##### 4.3.1 源码

![](./assets/13.1.png)

```java
//ReadWriteLock接口定义了获取读锁和写锁的规范
public class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable {

    /** 读锁 */
    private final ReentrantReadWriteLock.ReadLock readerLock;

    /** 写锁 */
    private final ReentrantReadWriteLock.WriteLock writerLock;

    final Sync sync;
    
    /** 使用默认（非公平）的排序属性创建一个新的 ReentrantReadWriteLock */
    public ReentrantReadWriteLock() {
        this(false);
    }

    /** 使用给定的公平策略创建一个新的 ReentrantReadWriteLock */
    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }

    /** 返回用于写入操作的锁 */
    public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
    
    /** 返回用于读取操作的锁 */
    public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }


    abstract static class Sync extends AbstractQueuedSynchronizer {}

    static final class NonfairSync extends Sync {}

    static final class FairSync extends Sync {}

    public static class ReadLock implements Lock, java.io.Serializable {}

    public static class WriteLock implements Lock, java.io.Serializable {}
}


//Sync抽象类继承自AQS抽象类，Sync类提供了对ReentrantReadWriteLock的支持
abstract static class Sync extends AbstractQueuedSynchronizer {
          
        static final class HoldCounter {
            //计数：表示某个读线程重入的次数
            int count = 0;
           // tid：表示该线程的tid字段的值，该字段可以用来唯一标识一个线程。
            final long tid = getThreadId(Thread.currentThread());
        }
    
    // 本地线程计数器
    //ThreadLocalHoldCounter重写了ThreadLocal的initialValue方法，ThreadLocal类可以将线程与对象相关联。在没有进行set的情况下，get到的均是initialValue方法里面生成的那个HolderCounter对象
       static final class ThreadLocalHoldCounter
            extends ThreadLocal<HoldCounter> {
            public HoldCounter initialValue() {
                return new HoldCounter();
            }
        }

      // 版本序列号
    private static final long serialVersionUID = 6317671515068378041L;        
    // 高16位为读锁，低16位为写锁
    static final int SHARED_SHIFT   = 16;
    // 读锁单位
    static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
    // 读锁最大数量
    static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
    // 写锁最大数量
    static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
    // 本地线程计数器
    private transient ThreadLocalHoldCounter readHolds;
    // 缓存的计数器
    private transient HoldCounter cachedHoldCounter;
    // 第一个读线程
    private transient Thread firstReader = null;
    // 第一个读线程的计数
    private transient int firstReaderHoldCount;
    

// 构造函数
Sync() {
    // 本地线程计数器
    readHolds = new ThreadLocalHoldCounter();
    // 设置AQS的状态
    setState(getState()); 
}

        abstract boolean readerShouldBlock();

        abstract boolean writerShouldBlock();

    protected final boolean tryRelease(int releases) {
        //若锁的持有者不是当前线程，抛出异常
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        //写锁的新线程数
        int nextc = getState() - releases;
        //如果独占模式重入数为0了，说明独占模式被释放
        boolean free = exclusiveCount(nextc) == 0;
        if (free)
            //若写锁的新线程数为0，则将锁的持有者设置为null
            setExclusiveOwnerThread(null);
        //设置写锁的新线程数
        //不管独占模式是否被释放，更新独占重入数
        setState(nextc);
        return free;
    }

    protected final boolean tryAcquire(int acquires) {
        //当前线程
        Thread current = Thread.currentThread();
        //获取AQS状态
        int c = getState();
        //写线程数量（即获取独占锁的重入数）
        int w = exclusiveCount(c);

        //当前同步状态state != 0，说明已经有其他线程获取了读锁或写锁
        if (c != 0) {
            //此时：如果写锁状态为0说明读锁此时被占用返回false；
            // 如果写锁状态不为0且写锁没有被当前线程持有返回false
            if (w == 0 || current != getExclusiveOwnerThread())
                return false;

            //判断同一线程获取写锁是否超过最大次数（65535），支持可重入
            if (w + exclusiveCount(acquires) > MAX_COUNT)
                throw new Error("Maximum lock count exceeded");
            //更新状态
            //此时当前线程已持有写锁，现在是重入，所以只需要修改锁的数量即可。
            setState(c + acquires);
            return true;
        }

        //到这里说明此时c=0,读锁和写锁都没有被获取
        //writerShouldBlock表示是否阻塞
        if (writerShouldBlock() ||
            !compareAndSetState(c, c + acquires))
            return false;

        //设置锁为当前线程所有
        setExclusiveOwnerThread(current);
        return true;
    }
        }
    
}
```



**tryAcquire（获取写锁）：**

![](./assets/13.3.png)



**tryRelease（释放写锁）：**

****![](./assets/13.2.png)



读锁暂时放一下。



# 5 悲观锁&乐观锁

锁从宏观上分类，分为悲观锁与乐观锁。 

### 5.1 乐观锁

- 乐观锁是一种乐观思想，即认为读多写少，遇到并发写的可能性低，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，采取在写时先读出当前版本号，然后加锁操作（比较跟上一次的版本号，如果一样则更新），如果失败则要重复读-比较-写的操作。

  java中的乐观锁**基本都是通过CAS操作实现的**，CAS是一种更新的原子操作，比较当前值跟传入值是否一样，一样则更新，否则失败。

### 5.2 悲观锁

- 悲观锁是就是悲观思想，即认为写多，遇到并发写的可能性高，每次去拿数据的时候都认为别人会修改，所以每次在读写数据的时候都会上锁，这样别人想读写这个数据就会block直到拿到锁。**java中的悲观锁就是Synchronized,AQS框架下的锁则是先尝试cas乐观锁去获取锁，获取不到，才会转换为悲观锁，如RetreenLock。**





# 6 锁的状态

[博客](https://www.jianshu.com/p/36eedeb3f912)

锁的状态总共有四种：无锁状态、偏向锁、轻量级锁和重量级锁。 

### 6.1 重量级锁（Mutex Lock） 

Synchronized 是通过对象内部的一个叫做监视器锁（monitor）来实现的。但是监视器锁本质又是依赖于底层的操作系统的 Mutex Lock 来实现的。而操作系统实现线程之间的切换这就需要从用户态转换到核心态，这个成本非常高。

JDK 中对 Synchronized 做的种种优化，其核心都是为了减少这种重量级锁的使用。 JDK1.6 以后，为了减少获得锁和释放锁所带来的性能消耗，提高性能，引入了“轻量级锁”和 “偏向锁”。 



### 6.2 锁升级 

随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁（但是锁的升级是单向的，也就是说只能从低到高升级，不会出现锁的降级）。 



### 6.3 轻量级锁 

“轻量级”是相对于使用操作系统互斥量来实现的传统锁而言的。它的本意是在没有多线程竞争的前提下，减少传统的重量级锁使用产生的性能消耗。轻量级锁所适应的场景是线程交替执行同步块的情况，如果存在同一时间访问同一锁的情况，就会导致轻量级锁膨胀为重量级锁。 



### 6.4 偏向锁

##### 6.4.1 介绍

Java偏向锁(Biased Locking)是Java6引入的一项多线程优化。

偏向锁，顾名思义，它会偏向于第一个访问锁的线程，如果在运行过程中，同步锁只有一个线程访问，不存在多线程争用的情况，则线程是不需要触发同步的，这种情况下，就会给线程加一个偏向锁。
如果在运行过程中，遇到了其他线程抢占锁，则持有偏向锁的线程会被挂起，JVM会消除它身上的偏向锁，将锁恢复到标准的轻量级锁。



##### 6.4.2 偏向锁的实现

偏向锁获取过程：

1. 访问Mark Word中偏向锁的标识是否设置成1，锁标志位是否为01，确认为可偏向状态。
2. 如果为可偏向状态，则测试线程ID是否指向当前线程，如果是，进入步骤5，否则进入步骤3。
3. 如果线程ID并未指向当前线程，则通过CAS操作竞争锁。如果竞争成功，则将Mark Word中线程ID设置为当前线程ID，然后执行5；如果竞争失败，执行4。
4. 如果CAS获取偏向锁失败，则表示有竞争。当到达全局安全点（safepoint）时获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。（撤销偏向锁的时候会导致stop the word）
5. 执行同步代码。



##### 6.4.3 偏向锁的释放

偏向锁的撤销在上述第四步骤中有提到。偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动去释放偏向锁。



##### 6.4.4 偏向锁的适用场景

始终只有一个线程在执行同步块，在它没有执行完释放锁之前，没有其它线程去执行同步块，在锁无竞争的情况下使用，一旦有了竞争就升级为轻量级锁，升级为轻量级锁的时候需要撤销偏向锁，撤销偏向锁的时候会导致stop the word操作；
在有锁的竞争时，偏向锁会多做很多额外操作，尤其是撤销偏向锁的时候会导致进入安全点，安全点会导致stw，导致性能下降，这种情况下应当禁用；

##### 6.4.5 开启/关闭偏向锁

- 开启偏向锁：-XX:+UseBiasedLocking -XX:BiasedLockingStartupDelay=0   
- 关闭偏向锁：-XX:-UseBiasedLocking  





# 7 自旋锁

是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```
### 7.1 原理

自旋锁原理非常简单，如果持有锁的线程能在很短时间内释放锁资源，那么那些等待竞争锁的线程就不需要做内核态和用户态之间的切换进入阻塞挂起状态，它们只需要等一等（自旋），等持有锁的线程释放锁后即可立即获取锁，这样就避免用户线程和内核的切换的消耗。

但是线程自旋是需要消耗cup的，所以需要设定一个自旋等待的最大时间。



### 7.2 自旋锁的优缺点

自旋锁尽可能的减少线程的阻塞，这对于锁的竞争不激烈，且占用锁时间非常短的代码块来说性能能大幅度的提升，因为自旋的消耗会小于线程阻塞挂起再唤醒的操作的消耗，这些操作会导致线程发生两次上下文切换。

但是如果锁的竞争激烈，或者持有锁的线程需要长时间占用锁执行同步块，这时候就不适合使用自旋锁了，因为自旋锁在获取锁前一直都是占用cpu做无用功，同时有大量线程在竞争一个锁，会导致获取锁的时间很长，线程自旋的消耗大于线程阻塞挂起操作的消耗，其它需要cup的线程又不能获取到cpu，造成cpu的浪费。所以这种情况下我们要关闭自旋锁；



### 7.3 自旋锁时间阈值

JVM对于自旋周期的选择，jdk1.5这个限度是一定的写死的，在1.6引入了适应性自旋锁，适应性自旋锁意味着自旋的时间不在是固定的了，而是由前一次在同一个锁上的自旋时间以及锁的拥有者的状态来决定，基本认为一个线程上下文切换的时间是最佳的一个时间，同时JVM还针对当前CPU的负荷情况做了较多的优化

- 如果平均负载小于CPUs则一直自旋
- 如果有超过(CPUs/2)个线程正在自旋，则后来线程直接阻塞
- 如果正在自旋的线程发现Owner发生了变化则延迟自旋时间（自旋计数）或进入阻塞
- 如果CPU处于节电模式则停止自旋
- 自旋时间的最坏情况是CPU的存储延迟（CPU A存储了一个数据，到CPU B得知这个数据直接的时间差）
- 自旋时会适当放弃线程优先级之间的差异



### 7.4 手写自旋锁（重点）

```java
package com.jian8.juc.lock;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;

/**
 * 实现自旋锁
 * 自旋锁好处，循环比较获取知道成功位置，没有类似wait的阻塞
 *
 * 通过CAS操作完成自旋锁，A线程先进来调用mylock方法自己持有锁5秒钟，B随后进来发现当前有线程持有锁，不是null，所以只能通过自旋等待，直到释放锁后B随后抢到
 */
public class SpinLockDemo {
    public static void main(String[] args) {
        SpinLockDemo spinLockDemo = new SpinLockDemo();
        new Thread(() -> {
            spinLockDemo.mylock();
            try {
                TimeUnit.SECONDS.sleep(3);
            }catch (Exception e){
                e.printStackTrace();
            }
            spinLockDemo.myUnlock();
        }, "Thread 1").start();

        try {
            TimeUnit.SECONDS.sleep(3);
        }catch (Exception e){
            e.printStackTrace();
        }

        new Thread(() -> {
            spinLockDemo.mylock();
            spinLockDemo.myUnlock();
        }, "Thread 2").start();
    }

    //原子引用线程
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void mylock() {
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName() + "\t come in");
        while (!atomicReference.compareAndSet(null, thread)) {

        }
    }

    public void myUnlock() {
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread, null);
        System.out.println(Thread.currentThread().getName()+"\t invoked myunlock()");
    }
}

```



# 8 synchronized

属于独占式的悲观锁，同时是可重入锁。



### 8.1 实现原理

synchronized关键字经过编译之后，会在同步块的前后分别形成**monitorenter和monitorexit这两个字节码指令**，这两个字节码都需要一个reference类型的参数来指明要锁定和解锁的对象。在执行monitorenter指令时，首先要尝试获取对象的锁。如果这个对象没被锁定，或者当前线程已经拥有了那个对象的锁，把锁的计数器加1，相应的，在执行monitorexit指令时会将锁计数器减1，当计数器为0时，锁就被释放。

也就是说当两个线程同时对一个对象的一个方法进行操作，只有一个线程能够抢到锁。因为**一个对象只有一把锁**，一个线程获取了该对象的锁之后，其他线程无法获取该对象的锁，就不能访问该对象的其他synchronized实例方法，但是可以访问非synchronized修饰的方法 

注意：synchronized是Java语言中一个重量级（Heavyweight）的操作，有经验的程序员都会在确实必要的情况下才使用这种操作。





### 8.2 作用范围

1. 作用于方法时，锁住的是对象的实例(this)； 
2. 当作用于静态方法时，锁住的是Class实例，又因为Class的相关数据存储在永久带PermGen （jdk1.8 则是 metaspace），永久代是全局共享的，因此静态方法锁相当于类的一个全局锁， 会锁所有调用该方法的线程； 

3. synchronized 作用于一个对象实例时，锁住的是所有以该对象为锁的代码块。它有多个队列， 当多个线程一起访问某个对象监视器的时候，对象监视器会将这些线程存储在不同的容器中



### 8.3 局限性

- 当线程尝试获取锁的时候，如果获取不到锁会一直阻塞。
- 如果获取锁的线程进入休眠或者阻塞，除非当前线程异常，否则其他线程尝试获取锁必须一直等待。



# 9 ReentrantLock

可以使用java.util.concurrent（下文称J.U.C）包中的重入锁(即一个线程可以多次获取锁)来实现同步。



### 9.1 ReenreantLock与synchronized的区别

1. 原始构成

   Lock 是一个接口，而 synchronized 是 Java 中的关键字，synchronized 是内置的语言实现。 

   - synchronized时关键字属于jvm

     **monitorenter**，底层是通过monitor对象来完成，其实wait/notify等方法也依赖于monitor对象只有在同步或方法中才能掉wait/notify等方法

     **monitorexit**

   - Lock是具体类，是api层面的锁（java.util.）

2. 使用方法

   - sychronized不需要用户取手动释放锁，当synchronized代码执行完后系统会自动让线程释放对锁的占用
   - ReentrantLock则需要用户去手动释放锁若没有主动释放锁，就有可能导致出现死锁现象，需要lock()和unlock()方法配合try/finally语句块来完成

3. 等待是否可中断

   - synchronized不可中断，除非抛出异常或者正常运行完成
   - ReentrantLock可中断，设置超时方法tryLock(long timeout, TimeUnit unit)，或者lockInterruptibly()放代码块中，调用interrupt()方法可中断。

4. 加锁是否公平

   - synchronized非公平锁
   - ReentrantLock两者都可以，默认公平锁，构造方法可以传入boolean值，true为公平锁，false为非公平锁

5. 锁绑定多个条件Condition

   - synchronized没有
   - ReentrantLock用来实现分组唤醒需要要唤醒的线程们，可以精确唤醒，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。

6. Lock 可以提高多个线程进行读操作的效率，既实现读写锁等。



Condition 类和 Object 类锁方法区别区别 :

1. Condition 类的 awiat 方法和 Object 类的 wait 方法等效 
2. Condition 类的 signal 方法和 Object 类的 notify 方法等效 
3. Condition 类的 signalAll 方法和 Object 类的 notifyAll 方法等效 
4. ReentrantLock 类可以唤醒指定条件的线程，而 object 的唤醒是随机的






# 10 锁优化


### 10.1 java线程阻塞的代价

java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统介入，需要在用户态与核心态之间切换，这种切换会消耗大量的系统资源，因为用户态与内核态都有各自专用的内存空间，专用的寄存器等，用户态切换至内核态需要传递给许多变量、参数给内核，内核也需要保护好用户态在切换时的一些寄存器值、变量等，以便内核态调用结束后切换回用户态继续工作。

如果线程状态切换是一个高频操作时，这将会消耗很多CPU处理时间；
如果对于那些需要同步的简单的代码块，获取锁挂起操作消耗的时间比用户代码执行的时间还要长，这种同步策略显然非常糟糕的。

synchronized会导致争用不到锁的线程进入阻塞状态，所以说它是java语言中一个重量级的同步操纵，被称为重量级锁，为了缓解上述性能问题，JVM从1.5开始，引入了轻量锁与偏向锁，默认启用了自旋锁，他们都属于乐观锁。



### 10.2 markword

markword是java对象数据结构中的一部分，markword数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，它的**最后2bit是锁状态标志位**，用来标记当前对象的状态，对象的所处的状态。

![10.1](./assets/10.1.png)





### 10.3 常见锁优化策略

##### 10.3.1 减少锁的时间

不需要同步执行的代码，能不放在同步快里面执行就不要放在同步快内，可以让锁尽快释放；



##### 10.3.2 减少锁的粒度

它的思想是将物理上的一个锁，拆成逻辑上的多个锁，增加并行度，从而降低锁竞争。它的思想也是用空间来换时间；

##### 10.3.3 锁粗化

大部分情况下我们是要让锁的粒度最小化，锁的粗化则是要增大锁的粒度;
在以下场景下需要粗化锁的粒度：
假如有一个循环，循环内的操作需要加锁，我们应该把锁放到循环外面，否则每次进出循环，都进出一次临界区，效率是非常差的；



##### 10.3.4 锁消除 

锁消除是在编译器级别的事情。在即时编译器时，如果发现不可能被共享的对象，则可以消除这些对象的锁操作，多数是因为程序员编码不规范引起的。



##### 10.3.4 使用读写锁

ReentrantReadWriteLock 是一个读写锁，读操作加读锁，可以并发读，写操作使用写锁，只能单线程写；



##### 10.3.5 读写分离

CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。







参考博客：https://blog.csdn.net/zzti_erlie/article/details/79964590

​		    https://blog.csdn.net/zqz_zqz/article/details/70233767