---
title: AQS-CountDownLatch
---

## 概念

CountDownLatch是一个同步工具类，用来协调多个线程之间的同步，或者说起到线程之间的通信（而不是用作互斥的作用）。

底层基于AQS实现，CountDownLatch 构造函数中指定的count直接赋给AQS的state；每次countDown（）都是release(1) 减1，最后减到0是unpack 阻塞线程；这一步是由最后一个执行 countDown 方法的线程执行的。

调用 await 方法时，当前线程就会判断 state 属性是否为0，如果为0，则继续往下执行，如果不为0，则使当前线程进入等待状态，知道某个线程 state 属性置为0，其就会唤醒在 await 中的线程

CountDownLatch能够使一个线程在等待另外一些线程完成各自工作之后，再继续执行。使用一个计数器进行实现。计数器初始值为线程的数量。当每一个线程完成自己任务后，计数器的值就会减一。当计数器的值为0时，表示所有的线程都已经完成一些任务，然后在CountDownLatch上等待的线程就可以恢复执行接下来的任务。



## 用法

1. 创建

```java
// 计数器的值设置为 线程数量
CountDownLatch countDownLatch = new CountDownLatch(10);
```

2. countDown

线程执行结束后调用此方法，计数器的值-1

```java
countDownLatch.countDown();
```

3. await

当计数器的数值为0的时候，调用此方法可以在 CountDownLatch 上的线程

```java
countDownLatch.await();
// 可以设置超时时间 超过次时间也会唤醒
countDownLatch.await(3000, TimeUnit.MILLISECONDS);
```

4. getCount

获取当前计数器的值

```java
countDownLatch.getCount();
```



## 实战

1. 确保子线程执行完毕 执行主线程

反例：

```java
public class CountDownLatchTest {

    private static int sum = 0;

    private static Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        // 创建十个线程
        for (int i = 0; i < 10; i++) {
            Thread t = new Thread(() -> {
                lock.lock();
                try {
                    for (int j = 0; j < 50000; j++) {
                        sum++;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    // 一定要在finally中释放锁
                    lock.unlock();
                }
            });
            t.start();
        }
        // 确保任务执行完成
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 输出结果
        System.out.println(sum);
    }
}
```

正例：

```java
public class CountDownLatchTest {

    private static int sum = 0;

    private static Lock lock = new ReentrantLock();

    private final static int THREAD_COUNT = 10;

    public static void main(String[] args) {
        // 创建十个线程
        CountDownLatch countDownLatch = new CountDownLatch(THREAD_COUNT);
        for (int i = 0; i < THREAD_COUNT; i++) {
            Thread t = new Thread(() -> {
                lock.lock();
                try {
                    for (int j = 0; j < 50000; j++) {
                        sum++;
                    }
                    System.out.println(countDownLatch.getCount());
                    countDownLatch.countDown();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    // 一定要在finally中释放锁
                    lock.unlock();
                }
            });
            t.start();
        }
        try {
            countDownLatch.await(3000, TimeUnit.MILLISECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 输出结果
        System.out.println(sum);
    }
}
```



