# [并发编程——volatile关键字和atomic包](https://www.cnblogs.com/java-chen-hao/p/9968544.html)阅读笔记

学习到的知识点：JMM内存管理模型，Volatile关键字作用，atomic包的实现原理。



### JVM 内存管理模型

JVM 是 Java  Memory  Model 的缩写，是java虚拟机的内存模型。理解CPU操作的数据来自哪几个地方：CPU寄存器（访问最快，应该算也算物理内存吧）、CPU高速缓存(仅次于寄存器)、物理内存、磁盘。

计算机执行二进制程序，本质就是指令+数据，而数据可能出现在上述几个地方，通常来说程序执行前就会分配一块内存地址，位于物理内存(主存)上，分为data段、堆、栈等。程序中的数据读写本质一般就是先从物理内存读取到寄存器(或CPU高速缓存)，操作完后再写回物理内存。ps:CPU执行指令的速度比内存读写快相当多，貌似记得是100倍。

![](https://img2018.cnblogs.com/blog/1168971/201811/1168971-20181116104336722-1804192704.png)

[文中例子](https://www.cnblogs.com/java-chen-hao/p/9968544.html)说明：

当程序在运行过程中，会将运算需要的数据从主存复制一份到CPU的高速缓存当中，那么CPU进行计算时就可以直接从它的高速缓存读取数据和向其中写入数据，当运算结束之后，再将高速缓存中的数据刷新到主存当中。举个简单的例子，比如下面的这段代码：

```c
i = i + 1;
```

当线程执行这个语句时，会先从主存当中读取i的值，然后复制一份到高速缓存当中，然后CPU执行指令对i进行加1操作，然后将数据写入高速缓存，最后将高速缓存中i最新的值刷新到主存当中。

　　这个代码在单线程中运行是没有任何问题的，但是在多线程中运行就会有问题了。在多核CPU中，每条线程可能运行于不同的CPU中，因此每个线程运行时有自己的高速缓存（对单核CPU来说，其实也会出现这种问题，只不过是以线程调度的形式来分别执行的）。本文我们以多核CPU为例。

　　比如同时有2个线程执行这段代码，假如初始时i的值为0，那么我们希望两个线程执行完之后i的值变为2。但是事实会是这样吗？

　　可能存在下面一种情况：初始时，两个线程分别读取i的值存入各自所在的CPU的高速缓存当中，然后线程1进行加1操作，然后把i的最新值1写入到内存。此时线程2的高速缓存当中i的值还是0，进行加1操作之后，i的值为1，然后线程2把i的值写入内存。

　　最终结果i的值是1，而不是2。这就是著名的**缓存一致性问题**。通常称这种被多个线程访问的变量为共享变量。



### 原子性

> 即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

数据库中在码农翻身一书中提到使用了日志记录方式保证原子性，将事务提交在操作前写入到日志中，出错可回滚。



### 可见性

可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。这里的"看到"按照下文意思是指读操作的时候，发现此时本地缓存值失效，那么需要重新从物理内存读取。

```c
//线程1执行的代码
int i = 0;
i = 10;
 
//线程2执行的代码
j = i;
```

假若执行线程1的是CPU1，执行线程2的是CPU2。由上面的分析可知，当线程1执行 i =10这句时，会先把i的初始值加载到CPU1的高速缓存中，然后赋值为10，那么在CPU1的高速缓存当中i的值变为10了，却没有立即写入到主存当中。

此时线程2执行 j = i，它会先去主存读取i的值并加载到CPU2的缓存当中，注意此时内存当中i的值还是0，那么就会使得j的值为0，而不是10。

这就是可见性问题，线程1对变量i修改了之后，线程2没有立即看到线程1修改的值。

### 有序性

有序性：即程序执行的顺序按照代码的先后顺序执行。主要考虑到编译器优化可能导致指令重排问题，这里涉及了 volatile 关键字。指令重排也要遵循两个规则：

1. **重排序操作不会对存在数据依赖关系的操作进行重排序**。比如：a=1;b=a; 这个指令序列，由于第二个操作依赖于第一个操作，所以在编译时和处理器运行时这两个操作不会被重排序。

2. **重排序是为了优化性能，但是不管怎么重排序，单线程下程序的执行结果不能被改变**。 比如：a=1;b=2;c=a+b这三个操作，第一步（a=1)和第二步(b=2)由于不存在数据依赖关系，所以可能会发生重排序，但是c=a+b这个操作是不会被重排序的，因为需要保证最终的结果一定是c=a+b=3。

### volatile 关键字

1. 保证可见性；
2. 不能保证原子性；
3. 禁止指令重排

**1.共享变量的可见性**

这里很神奇，我自己没有试验过，按照博主的测试来讲，结合自己的理解：

```java
public class TestVolatile {
    
    public static void main(String[] args) {
        ThreadDemo td = new ThreadDemo();
        new Thread(td).start();<============= 这里会开启一个新线程，但是内部延迟了200ms，保证下面的while(true) 代码先执行到
        while(true){
            if(td.isFlag()){
                System.out.println("------------------");
                break;
            }
        }
    }

}

class ThreadDemo implements Runnable {
    private  boolean flag = false;
    @Override
    public void run() {
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
        }
        flag = true;
        System.out.println("flag=" + isFlag());
    }

    public boolean isFlag() {
        return flag;
    }
}
```

**答案是NO**。

注释中说到线程由于延迟了200ms，所以此时主存(物理内存)中的flag值还是false，所以执行到 `while(true)`中的`td.isFlag()` 时会将主存中的flag=false读取到高速缓存中，按照代码逻辑，一直会跑在这个while循环中；

200 ms 后，子线程中的flag修改了true，然后写回了主存！但是作者意思主线程中的 `while(true)`  是底层的指令来实现，速度非常之快，一直循环都没有时间去主存中更新td的值——即一直在使用高速缓存中的值，所以这里会造成死循环！

解决方法是加volatile关键字，一旦子线程中修改了，那么立即会修改主存值，这不是关键，关键是这个更新操作会让所以其他的高速缓存的flag值都失效！！！！所以当主线程中 `td.isFlag()` 读高速缓存值时发现已经失效，只能从物理内存读取了！

**2. 禁止指令重排**

```java
class Singleton{
    private volatile static Singleton instance = null;

    private Singleton() {
    }
     
    public static Singleton getInstance() {
        if(instance==null) {
            synchronized (Singleton.class) {//<===== 多线程进到这里 由于加了锁 所以只会有一个进入下面代码进行实例化操作
                if(instance==null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```

乍看之下没有问题，单例实例化不就这么玩吗，问题就出在`instance = new Singleton();` 这是个复合操作！！！

instance = new Singleton(); 这段代码可以分为三个步骤：

1. memory = allocate() 分配对象的内存空间
2. ctorInstance() 初始化对象
3. instance = memory 设置instance指向刚分配的内存

假设指令重排顺序改为

1. memory = allocate() 分配对象的内存空间
2. instance = memory 设置instance指向刚分配的内存
3. ctorInstance() 初始化对象

在单线程的情况下，1->3->2这种顺序执行是没有问题的，但是如果是多线程的情况则有可能出现问题，线程A执行到11行代码，执行了指令1和3，此时instance已经有值了，值为第一步分配的内存空间地址，但是还没有进行对象的初始化；

此时线程B执行到了第8行代码处，此时instance已经有值了则return instance，线程B 使用instance的时候，就会出现异常。

**volatile 无法保证原子性**，例子如下：

```java
package com.mmall.concurrency.example.count;
import java.util.concurrent.CountDownLatch;

/**
 * @author: ChenHao
 * @Description:
 * @Date: Created in 15:05 2018/11/16
 * @Modified by:
 */
public class CountTest {
    // 请求总数
    public static int clientTotal = 5000;
    public static volatile int count = 0;

    public static void main(String[] args) throws Exception {
        //使用CountDownLatch来等待计算线程执行完
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        //开启clientTotal个线程进行累加操作
        for(int i=0;i<clientTotal;i++){
            new Thread(){
                public void run(){
                    count++;//自加操作
                    countDownLatch.countDown();
                }
            }.start();
        }
        //等待计算线程执行完
        countDownLatch.await();
        System.out.println(count);
    }
}
```

问题就出在count++这个操作上，**因为count++不是个原子性的操作，而是个复合操作**。我们可以简单讲这个操作理解为由这三步组成:

1. 读取count

2. count 加 1

3. 将count 写到主存

即使加了volatile关键字，如果线程A和线程B分别在执行count++，线程A，B都先从物理内存读取了count值，保存在了各自的本地内存中，假设线程A执行很快，剩下的都执行完了，累加了count值写入了物理内存，由于volatile关键，此时所有的本地缓存都会失效——但这个应该只针对读操作的时候才能发现，由于我们先于失效前就读入了，所以操作的还是旧值，线程B依然对过期的本地缓存count进行自加，重新写到主存中，最终导致了count的结果不合预期，而是小于5000。

### Atomic包实现原理

**基于Compare And Swap(CAS)，比较值并交换。**

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。” Java并发包(java.util.concurrent)中大量使用了CAS操作,涉及到并发的地方都调用了sun.misc.Unsafe类方法进行CAS操作。

```java
public final int incrementAndGet() {
     return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2); <=== 先从物理内存读值
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));<=== 这里还会再读一次，然后和上一次读到的var5比较

    return var5;
}
```

其中valueOffset就是可理解为从对象obj所在内存偏移valueoffset所在内存的值。



