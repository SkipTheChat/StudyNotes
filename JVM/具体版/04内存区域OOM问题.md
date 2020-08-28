### 1 参数设置

这些参数对实验的结果有直接影响，调试代码的时候千万不要忽略。

1. 如果使用控制台命令来执行程序，那直接跟在Java命令之后书写就可以。

2. 如果使用Eclipse IDE，则可以参考下图进行设置：

   -verbose:gc -Xms20M -Xmx20m -Xmn10m -XX:+PrintGCDetails -XX:SurvivorRatio=8

![TIM截图20191214101205](./assets/TIM%E6%88%AA%E5%9B%BE20191214101205.png)



参数说明：

 * -verbose:gc  ：报告每个垃圾收集事件,稳定版本,用于垃圾收集的信息收集
 * -Xms20m  ：  设置堆最小为20M，是指设定程序启动时占用内存大小。一般来讲，大点，程序会启动的快一点，但是也可能会导致机器暂时间变慢。
 * -Xmx20m  ：   设置堆最大为20M
 * -XX:+PrintGCDetails： 控制台打印GC日志详情，包含-XX:+PrintGC
 * -XX:SurvivorRatio=8 ：设置两个Survivor区和eden的比，8表示两个Survivor:eden=2:8即一个Survivor占年轻代的1/10
 *  -Xss :是指设定每个线程的堆栈大小。默认JDK1.4中是256K，JDK1.5+中是1M



### 2. Java堆内存溢出异常测试

##### 2.1 测试用例

Java堆用于存储对象实例，只要不断地创建对象，并且保证GC Roots到对象之间有可达路径来避免垃圾回收机制清除这些对象，那么在对象数量到达最大堆的容量限制后就会产生内存溢出异常。

测试代码如下：

```
/**
 * VM Args：-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
 * @author zzm
 */
public class HeapOOM {

    static class OOMObject {
    }

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<OOMObject>();

        while (true) {
            list.add(new OOMObject());
        }
    }
}
```



运行结果：

```
java.lang.OutOfMemoryError :Java heap space
Dumping heap to java_pid3404.hprof.
Heap dump file created[22045981 bytes in 0.663 secs]
```



##### 2.2 解决手段

 要解决这个区域的异常，一般的手段是先通过内存映像分析工具（如Eclipse Memory Analyzer）对Dump出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分清楚到底是出现了内存泄漏（Memory Leak）还是内存溢出（Memory Overflow）。

1. 如果是内存泄露，可进一步通过工具查看泄露对象到GC Roots的引用链。于是就能找到泄露对象是通过怎样的路径与GC Roots相关联并导致垃圾收集器无法自动回收它们的。掌握了泄露对象的类型信息及GC Roots引用链的信息，就可以比较准确地定位出泄露代码的位置。
2.  如果不存在泄露，换句话说，就是内存中的对象确实都还必须存活着，那就应当检查虚拟机的堆参数**（-Xmx与-Xms）**，与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。 



##### 2.3 内存泄漏和内存溢出的区别

**内存泄漏：**内存泄漏是指程序中己动态分配的堆内存由于某种原因程序**未释放或无法释放**，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃。 一次内存泄露危害可以忽略，但内存泄露堆积会导致OOM。

**内存溢出(Out Of Memory，简称OOM) ：**是指应用系统中存在无法回收的内存或使用的内存过多，最终使得程序运行要用到的内存大于能提供的最大内存。





### 3 虚拟机栈和本地方法栈溢出 

HotSpot虚拟机并不区分虚拟机栈和本地方法栈，栈容量只由**-Xss参数**设定。关于虚拟机栈和本地方法栈，Java虚拟机规范描述了两种异常：

- StackOverflowError异常：线程请求的栈深度大于虚拟机所允许的最大深度。 
- OutOfMemoryError异常：虚拟机在扩展栈时无法申请到足够的内存空间。

以上两种异常有所重叠。



##### 3.1 测试用例（限制单线程）

以下代码尝试两种方法均无法让虚拟机产生OutOfMemoryError异常。

* 使用-Xss参数减少栈内存容量。

  结果：抛出StackOverflowError异常，异常出现时输出的堆栈深度相应缩小。

* 定义了大量的本地变量，增大此方法帧中本地变量表的长度。

  结果：抛出StackOverflowError异常时输出的堆栈深度相应缩小。

```
package com.jvm.OutOfMemoryError;
/**
 * 虚拟机栈和本地方法栈溢出 StackOverFlow
 * VM Args：-Xss128k
 */

public class JavaVMStackSOF {

    private int stackLength = 1;

    // 递归调用方法，定义大量的本地变量，增大此方法帧中本地变量表的长度
    public void stackLeak() {
        stackLength++;
        stackLeak();
    }

    public static void main(String[] args) {
        JavaVMStackSOF oom = new JavaVMStackSOF();
        try {
            oom.stackLeak();
        } catch (Throwable e) {
            System.out.println("stack length: " + oom.stackLength);
            throw e;
        }
    }

}
```



运行结果： 

```
stack length: 999
Exception in thread "main" java.lang.StackOverflowError
    at com.jvm.OutOfMemoryError.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:26)
    at com.jvm.OutOfMemoryError.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:26)
    at com.jvm.OutOfMemoryError.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:26)
    ......
```

实验结果表明：在单个线程下，无论是由于栈帧太大还是虚拟机栈容量太小，当内存无法分配的时候，虚拟机抛出的都是StackOverflowError异常。 



##### 3.2 测试用例（多线程）

如果测试时不限于单线程，通过不断地建立线程的方式可以产生内存溢出异常。在这种情况下，为每个线程的栈分配的内存越大，反而越容易产生内存溢出异常。

注意：特别提示一下，如果要尝试运行上面这段代码，记得要先保存当前的工作。上述代码执行时有较大的风险，可能会导致操作系统假死。  

```
package com.jvm.OutOfMemoryError;
/**
 * 虚拟机栈和本地方法栈溢出
 * 
 * !!!!!注意!!!!!
 * 运行该代码可能造成系统假死！！！！
 * 
 * VM Args：-Xss2M（可以适当大一点）
 */

public class JavaVMStackOOM {

    private void dontStop() {
        while(true) {

        }
    }

    // 多线程方式造成栈内存溢出 OutOfMemoryError
    public void stackLeakByThread() {
        while(true) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    dontStop();
                }
            });
            thread.start();
        }
    }

    public static void main(String[] args) {
        JavaVMStackOOM oom = new JavaVMStackOOM();
        oom.stackLeakByThread();
    }
}
```



运行结果：

```
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
```

 

原因：线程越多，就越容易耗尽分配给进程的虚拟机栈和本地方法栈内存。

如果是建立过多线程导致的内存溢出，在不能减少线程数或者更换64位虚拟机的情况下，就只能通过减少最大堆和减少栈容量来换取更多的线程。



### 4 方法区和运行常量池溢出

由于运行时常量池是方法区的一部分，因此这两个区域的溢出测试就放在一起进行。 JDK1.7开始逐步“去永久代”（方法区常被称为永久代），下面的讨论可以测试下实际影响。 

##### 4.1 测试运行时常量池导致的内存溢出异常

String.intern（）是一个Native方法，它的作用是：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回代表池中这个字符串的String对象；否则，将此String对象包含的字符串添加到常量池中，并且返回此String对象的引用。

在JDK 1.6及之前的版本中，由于常量池分配在永久代内，我们可以通过XX：PermSize和-XX：MaxPermSize限制方法区大小，从而间接限制其中常量池的容量，如下代码所示

```
package com.jvm.OutOfMemoryError;

import java.util.ArrayList;
import java.util.List;

/**
 * 运行时常量池溢出
 * 
 * jdk1.7 及其以后的版本已经“去永久代”，jdk1.6之前的环境下运行才会出现溢出
 * -XX:PermSize=10M -XX:MaxPermSize=10M （间接指定常量池容量大小）
 */

public class RuntimeConstantPoolOOM {
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        int i = 0;
        while(true) {
            // list保留引用，避免Full GC 回收 
            list.add(String.valueOf(i++).intern());
        }
    }
}
```

  

运行结果：

```
jdk1.6：（PermGen space，说明运行时常量池属于方法区（HotSpot虚拟机中的永久代）的一部分。  ）
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space
    at java.lang.String.intern(Native Method)
    ......
    
jdk1.7:while循环将一直进行下去

```





##### 4.2 验证jdk1.7去永久代

```
public class RuntimeConstantPoolOOM {
    public static void main(String[] args) {
            public static void main(String[] args) {
            String str1 = new StringBuilder("计算机").append("软件").toString();
            System.out.println(str1.intern() == str1);
            String str2 = new StringBuilder("ja").append("va").toString();
            System.out.println(str2.intern() == str2);
    }      }
}

```



运行结果：

```
jdk1.6：
false
false

jdk1.7：
true
false
```



产生差异的原因是：

* 在JDK 1.6中，intern（）方法会把首次遇到的字符串实例复制到永久代中，返回的也是永久代中这个字符串实例的引用，而由StringBuilder创建的字符串实例在Java堆上，所以必然不是同一个引用，将返回false。

* JDK 1.7（以及部分其他虚拟机，例如JRockit）的intern（）实现不会再复制实例，**只是在常量池中记录首次出现的实例引用**，因此intern（）返回的引用和由StringBuilder创建的那个字符串实例是同一个。

  对str2比较返回false是因为“java”这个字符串在执行StringBuilder.toString（）之前已经出现过，字符串常量池中已经有它的引用了，**不符合“首次出现”的原则**，而“计算机软件”这个字符串则是首次出现的，因此返回true。

  

由此也可验证 JDK1.7开始逐步“去永久代”。



##### 4.3 测试方法区溢出

方法区用于存放Class的相关信息，如类名、访问修饰符、常量池、字段描述、方法描述等。对于这些区域的测试，基本的思路是运行时产生大量的类去填满方法区，直到溢出。

本次借助CGLib直接操作字节码运行时生成了大量的动态类。 在这个例子中模拟的场景并非纯粹是一个实验，这样的应用经常会出现在实际应用中：当前的很多主流框架，如Spring、Hibernate，在对类进行增强时，都会使用到CGLib这类字节码技术，增强的类越多，就需要越大的方法区来保证动态生成的Class可以加载入内存。另外，JVM上的动态语言（例如Groovy等）通常都会持续创建类来实现语言的动态性，随着这类语言的流行越来越容易遇到与代码清单2-8相似的溢出场景。  

CGLib动态生成类导致的方法区溢出 ：

```
package com.jvm.OutOfMemoryError;

import java.lang.reflect.Method;
import com.jvm.OutOfMemoryError.HeapOOM.OOMObject;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

/**
 * 方法区溢出
 * -XX:PermSize=10M -XX:MaxPermSize=10M
 */

public class JavaMethodAreaOOM {
    public static void main(String[] args) {
        while(true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(OOMObject.class);
            enhancer.setUseCache(false);
            enhancer.setCallback(new MethodInterceptor() {

                @Override
                public Object intercept(Object obj, Method m, Object[] objs, MethodProxy proxy) throws Throwable {
                    // TODO Auto-generated method stub
                    return proxy.invokeSuper(obj, objs);
                }
            });
            enhancer.create();
        }
    }
}
```



运行结果：

```
Caused by: java.lang.OutOfMemoryError: PermGen space
    at java.lang.ClassLoader.defineClass1(Native Method)
    ......
```



方法区溢出是一种常见的内存溢出异常，一个类要被垃圾收集器回收掉，判定条件是比较苛刻的。在经常动态生成大量Class的应用中，需要特别注意类的回收状况。

这类场景除了上面提到的程序使用了CGLib字节码增强和动态语言之外，常见的还有：大量JSP或动态产生JSP文件的应用（JSP第一次运行时需要编译为Java类）、基于OSGi的应用（即使是同一个类文件，被不同的加载器加载也会视为不同的类）等。 



### 5 本机直接内存溢出

DirectMemory容量可通过-XX：MaxDirectMemorySize指定，如果不指定，则默认与Java堆最大值（-Xmx指定）一样。

下面的代码越过了DirectByteBuffer类，直接通过反射获取Unsafe实例进行内存分配。

原因：虽然使用DirectByteBuffer分配内存也会抛出内存溢出异常，但它抛出异常时并没有真正向操作系统申请分配内存，而是通过计算得知内存无法分配，于是手动抛出异常，真正申请分配内存的方法是unsafe.allocateMemory()。 

代码测试：使用unsafe分配本机内存

```
import java.lang.reflect.Field;

public class DirectMemoryOOM {

    private static final int _1MB = 1024 * 1024 * 1024;

    public static void main(String[] args) throws Exception {
        Field unsafeField = Unsafe . class .getDeclaredFields()[0];     
        unsafeField.setAccessible( true );
        Unsafe unsafe = ( Unsafe ) unsafeField.get( null );

        while ( true ) {
            // unsafe 直接想操作系统申请内存
            unsafe.allocateMemory( _1MB );
        }
    }
}
```



运行结果：

```
Exception in thread "main" java.lang.OutOfMemoryError
    at sun.misc.Unsafe.allocateMemory(Native Method)
    ......
```



一个明显的特征是在Heap Dump文件中不会看见明显的异常，如果发现OOM之后Dump文件很小，而程序中又直接或间接使用了NIO，那就可以考虑检查一下是不是这方面的原因。 





参考博客：https://blog.csdn.net/weixin_34384915/article/details/94216694