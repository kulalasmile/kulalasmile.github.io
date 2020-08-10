[TOC]
# Java锁

## 公平锁和非公平锁

> 概念：

1.公平锁： 是指多个线程按照**申请**锁的顺序来**获取**锁。获取不到锁的时候，会自动加入队列，等待线程释放后，队列的**第一个线程获取锁** ，类似排队打饭，先来后到。 

2.非公平锁： 是指多个线程获取锁的顺序并不是按照申请锁的顺序。 获取不到锁的时候，会自动加入队列，等待线程释放锁后**所有等待的线程同时去竞争** 

> 区别：

1.公平锁： 就是很公平，在**并发坏境**中．每个线程在获取锁时会先查看此锁维护的**等待队列**，如果为空，或者当前线程是等待队列的第一个，就占有锁．否则就会加入到等待队列中．以后会按照FIFO的规则从队列中取到自己。 

2.非公平锁： 非公平锁比较粗鲁，上来就**直接尝试占有锁**，如果尝试失败，就再采用类似公平锁那种方式。 

> 运用：

ReentrantLock和synchronized是非公平锁

> 举例：

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo11183270-ba0934f9109b276b.jpg)



## 可重入锁和递归锁

> 概念：

 可重入锁指的是同一个线程进入外层函数获得锁，内层同步函数仍然可以使用该锁，并且不会发生死锁。 ReentrantLock和synchronized默认都是可重入锁。 

> 举例：

 可重入锁就是，在一个method1方法中加入一把锁，method2也加锁了，那么他们拥有的是同一把锁 

```java
public synchronized void method1() {
	method2();
}

public synchronized void method2() {

}
```

 也就是说我们只需要进入method1后，那么它也能直接进入method2方法，因为他们所拥有的锁，是同一把。 

> 作用：

 可重入锁的最大作用就是避免死锁 

> 证明：

```java
/**
 * 资源类
 */
class Phone {

    /**
     * 发送短信
     * @throws Exception
     */
    public synchronized void sendSMS() throws Exception{
        System.out.println(Thread.currentThread().getName() + "\t invoked  sendSMS()");

        // 在同步方法中，调用另外一个同步方法
        sendEmail();
    }

    /**
     * 发邮件
     * @throws Exception
     */
    public synchronized void sendEmail() throws Exception{
        System.out.println(Thread.currentThread().getId() + "\t invoked sendEmail()");
    }
}
public class ReenterLockDemo {

    public static void main(String[] args) {
        Phone phone = new Phone();

        // 两个线程操作资源列
        new Thread(() -> {
            try {
                phone.sendSMS();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "t1").start();

        new Thread(() -> {
            try {
                phone.sendSMS();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "t2").start();
    }
}
```

 在这里，我们编写了一个资源类phone，拥有两个加了synchronized的同步方法，分别是sendSMS 和 sendEmail，我们在sendSMS方法中，调用sendEmail。最后在主线程同时开启了两个线程进行测试，最后得到的结果为： 

```java
t1	 invoked sendSMS()
t1	 invoked sendEmail()
t2	 invoked sendSMS()
t2	 invoked sendEmail()
```

 这就说明当 t1 线程进入sendSMS的时候，拥有了一把锁，同时t2线程无法进入，直到t1线程拿着锁，执行了sendEmail 方法后，才释放锁，这样t2才能够进入 

```java
t1	 invoked sendSMS()      t1线程在外层方法获取锁的时候
t1	 invoked sendEmail()    t1在进入内层方法会自动获取锁

t2	 invoked sendSMS()      t2线程在外层方法获取锁的时候
t2	 invoked sendEmail()    t2在进入内层方法会自动获取锁
```



## 自旋锁和互斥锁

> 概念：

 自旋锁：（spin lock）是一种**非阻塞锁**，也就是说，如果某线程需要获取锁，但该锁已经被其他线程占用时，该线程**不会被挂起**，而是在不断的**消耗CPU的时间**，不停的**试图获取**锁。 

 互斥量：（mutex）是**阻塞锁**，当某线程无法获取锁时，该线程会被直接**挂起**，该线程**不再消耗**CPU时间，当其他线程释放锁后，操作系统会激活那个被挂起的线程，让其投入运行。 

> 优点：

 自旋锁避免了进程上下文的调度开销，因此对于线程只会阻塞很短时间的场合是有效的 。

互斥锁有一个缺点，他的执行流程是这样的 托管代码  - 用户态代码 - 内核态代码、上下文切换开销与损耗，假如获取到资源锁的线程A立马处理完逻辑释放掉资源锁，如果是采取互斥的方式，那么线程B从没有获取锁到获取锁这个过程中，就要用户态和内核态调度、上下文切换的开销和损耗。所以就有了自旋锁的模式，让线程B就在用户态循环等着，减少消耗。

 自旋锁不会使线程状态发生切换，一直处于用户态，即线程一直都是active的；不会使线程进入阻塞状态，减少了不必要的上下文切换，执行速度快 

 非自旋锁在获取不到锁的时候会进入阻塞状态，从而进入内核态，当获取到锁的时候需要从内核态恢复，需要线程上下文切换。 （线程被阻塞后便进入内核（Linux）调度状态，这个会导致系统在用户态与内核态之间来回切换，严重影响锁的性能） 

> 问题：

 过多占用CPU的资源，如果锁持有者线程A一直长时间的持有锁处理自己的逻辑，那么这个线程B就会一直循环等待过度占用cpu资源

 

## 共享锁和排它锁

> 概念：





## 读写锁

>  概念：  

在同一时刻允许多个读线程访问，但是当写线程访问，所有的写线程和读线程均被阻塞。读写锁维护了一个读锁加一个写锁，通过读写锁分离的模式来保证线程安全，性能高于一般的排他锁。

>  锁降级：

```java
public class Test1 {

    public static void main(String[] args) {
        ReentrantReadWriteLock rtLock = new ReentrantReadWriteLock();
        rtLock.readLock().lock();
        System.out.println("get readLock.");
        rtLock.writeLock().lock();
        System.out.println("blocking");
    }
}
```

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200519184200.png)

 结论：上面的测试代码会产生死锁，因为同一个线程中，在没有释放读锁的情况下，就去申请写锁，这属于**锁升级，ReentrantReadWriteLock是不支持的**。 

```java

public class Test2 {

    public static void main(String[] args) {
        ReentrantReadWriteLock rtLock = new ReentrantReadWriteLock();  
        rtLock.writeLock().lock();  
        System.out.println("writeLock");  
          
        rtLock.readLock().lock();  
        System.out.println("get read lock");  
    }
}
```

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200519184250.png)

## 阻塞队列

> 概念：

支持阻塞操作的队列。具体来讲，支持阻塞添加和阻塞移除。

阻塞添加：当队列满的时候，队列会阻塞插入插入的元素的线程，直到队列不满；

阻塞移除：在队列为空时，队里会阻塞插入元素的线程，直到队列不满。

阻塞队列常用于生产者和消费者场景，生产者是向队列添加元素的线程；消费者是从队列取元素的线程。


![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200519190046.png)

> 特点：

1）当阻塞队列为空时，从队列中获取元素的操作将会被阻塞，就好比餐馆休息区没人了，此时不能接纳新的顾客了。换句话，肚子为空的时候也没东西吃。

（2）当阻塞队列满了，往队列添加元素的操作将会被阻塞，好比餐馆的休息区也挤满了，后来的顾客只能走了。

从上面的概念我们类比到线程中去，我们会发现，在某些时候线程可能不能不阻塞，因为CPU内核就那么几个，阻塞现状更加说明了资源的利用率高，换句话来说，阻塞其实是一个好事。

 阻塞队列常用于生产者和消费者的场景，生产者是向队列里添加元素的线程，消费者是从队列里取元素的线程。阻塞队列就是生产者用来存放元素、消费者用来获取元素的容器。 

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200519191534.png)

![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200519191743.png)

抛出异常：是指当阻塞队列满时候，再往队列里插入元素，会抛出 IllegalStateException("Queue full") 异常。当队列为空时，从队列里获取元素时会抛出 NoSuchElementException 异常 。

返回特殊值：插入方法会返回是否成功，成功则返回 true。移除方法，则是从队列里拿出一个元素，如果没有则返回 null。

一直阻塞：当阻塞队列满时，如果生产者线程往队列里 put 元素，队列会一直阻塞生产者线程，直到拿到数据，或者响应中断退出。当队列空时，消费者线程试图从队列里 take 元素，队列也会阻塞消费者线程，直到队列可用。

超时退出：当阻塞队列满时，队列会阻塞生产者线程一段时间，如果超过一定的时间，生产者线程就会退出。

