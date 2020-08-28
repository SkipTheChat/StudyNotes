# [交替打印FooBar](https://leetcode-cn.com/problems/print-foobar-alternately/)

### 信息卡片

- 时间： 2020-3-7
- 难度：中等
- 题目描述：

```
我们提供一个类：

class FooBar {
  public void foo() {
    for (int i = 0; i < n; i++) {
      print("foo");
    }
  }

  public void bar() {
    for (int i = 0; i < n; i++) {
      print("bar");
    }
  }
}

两个不同的线程将会共用一个 FooBar 实例。其中一个线程将会调用 foo() 方法，另一个线程将会调用 bar() 方法。

请设计修改程序，以确保 "foobar" 被输出 n 次。

 

示例 1:

输入: n = 1
输出: "foobar"
解释: 这里有两个线程被异步启动。其中一个调用 foo() 方法, 另一个调用 bar() 方法，"foobar" 将被输出一次。

示例 2:

输入: n = 2
输出: "foobarfoobar"
解释: "foobar" 将被输出两次。

```



### 参考答案

> 思路

Semaphore



> 代码

```java
class FooBar {
    private int n;

    public FooBar(int n) {
        this.n = n;
    }

    Semaphore foo = new Semaphore(1);
    Semaphore bar = new Semaphore(0);

    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            foo.acquire();
            printFoo.run();
            bar.release();
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            bar.acquire();
            printBar.run();
            foo.release();
        }
    }
}
```





### 其他优秀解答

#### 1 Lock（公平锁）

```java
class FooBar {
    private int n;

    public FooBar(int n) {
        this.n = n;
    }

    Lock lock = new ReentrantLock(true);
    volatile boolean permitFoo = true;

    public void foo(Runnable printFoo) throws InterruptedException {
        
        for (int i = 0; i < n; ) {
            lock.lock();
            try {
            	if(permitFoo) {
            	    printFoo.run();
                    i++;
                    permitFoo = false;
            	}
            }finally {
            	lock.unlock();
            }
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; ) {
            lock.lock();
            try {
            	if(!permitFoo) {
            	    printBar.run();
            	    i++;
            	    permitFoo = true;
            	}
            }finally {
            	lock.unlock();
            }
        }
    }
}
```



#### 2 无锁

> 代码

```java
class FooBar {
    private int n;

    public FooBar(int n) {
        this.n = n;
    }

    volatile boolean permitFoo = true;

    public void foo(Runnable printFoo) throws InterruptedException {     
        for (int i = 0; i < n; ) {
            if(permitFoo) {
        	printFoo.run();
            	i++;
            	permitFoo = false;
            }
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {       
        for (int i = 0; i < n; ) {
            if(!permitFoo) {
        	printBar.run();
        	i++;
        	permitFoo = true;
            }
        }
    }
}

```



#### 3 CyclicBarrier

用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

可循环（Cyclic）使用的屏障。让一组线程到达一个屏障（也可叫同步点）时被阻塞，知道最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活，线程进入屏障通过CycliBarrier的await()方法

> 代码

```java
class FooBar {
    private int n;

    public FooBar(int n) {
        this.n = n;
    }

    CyclicBarrier cb = new CyclicBarrier(2);
    volatile boolean fin = true;

    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            while(!fin);
            printFoo.run();
            fin = false;
            try {
		cb.await();
	    } catch (BrokenBarrierException e) {
	    }
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            try {
		cb.await();
	    } catch (BrokenBarrierException e) {
	    }
            printBar.run();
            fin = true;
        }
    }
}
```



#### 4 synchronized

> 思路

synchronized



> 代码

```java
class FooBar {
    private int n;
    private Object lock = new Object();
    private boolean flag;
    public FooBar(int n) {
        this.n = n;
    }

    public void foo(Runnable printFoo) throws InterruptedException {
        synchronized(lock){
            for (int i = 0; i < n; i++) {
            while(flag){
                lock.wait();
            }
        	// printFoo.run() outputs "foo". Do not change or remove this line.
        	printFoo.run();
            flag = true;
            lock.notifyAll();
        }

        }
        
    }

    public void bar(Runnable printBar) throws InterruptedException {
        synchronized(lock){
        for (int i = 0; i < n; i++) {
            while(!flag){
                lock.wait();
            }
            // printBar.run() outputs "bar". Do not change or remove this line.
        	printBar.run();
            flag = false;
            lock.notifyAll();
        }
        }
    }
}
```





