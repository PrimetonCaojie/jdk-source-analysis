# Semaphore

### 1、介绍

Semaphore是一个计数信号量，可以控同时访问的线程个数，它的本质是一个"共享锁"。

信号量维护了一个信号量许可集。线程可以通过调用acquire()来获取信号量的许可；当信号量中有可用的许可时，线程能获取该许可；否则线程必须等待，直到有可用的许可为止。 线程可以通过release()来释放它所持有的信号量许可。

**结构示意图**

![semaphore](https://github.com/muyutingfeng/jdk-source-analysis/raw/master/note/doc/java/util/concurrent/semaphore.jpg?raw=true)

#### 1.1、核心接口/方法定义

```java
public void acquire() throws InterruptedException {  }     //获取一个许可
public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
public void release() { }          //释放一个许可
public void release(int permits) { }    //释放permits个许可
```

acquire()用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。

release()用来释放许可。注意，在释放许可之前，必须先获获得许可。

#### 1.2、构造方法

```java
/**
     * Creates a {@code Semaphore} with the given number of
     * permits and nonfair fairness setting.
     *
     * @param permits the initial number of permits available.
     *        This value may be negative, in which case releases
     *        must occur before any acquires will be granted.
     */
    public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }

    /**
     * permits:表示许可数目，即同时可以允许多少线程进行访问
     * fair:表示是否是公平的，即等待时间越久的越先获取许可
     */
    public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }
```

#### 1.3、demo示例

假如一个游乐场有3台滑梯，但是有5个小朋友，一台滑梯同时只能被一个小朋友使用，只有使用完了，其他小朋友才能继续使用。那么我们就可以通过Semaphore来实现：

```java
package com.github.mf.common;

import java.util.concurrent.Semaphore;

public class SemaphoreTest {
    public static void main(String[] args) {
        //小朋友的数量
        int kidsNum = 5;
        //共有三台滑梯
        Semaphore semaphore = new Semaphore(3);
        for(int i=0;i<kidsNum;i++)
            new SemaphoreTest.Worker(i,semaphore).start();
    }

    static class Worker extends Thread{
        private int num;
        private Semaphore semaphore;
        public Worker(int num,Semaphore semaphore){
            this.num = num;
            this.semaphore = semaphore;
        }

        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println(this.num+"号小朋友，占用这台滑梯");
                Thread.sleep(2000);
                System.out.println(this.num+"号小朋友，离开这台滑梯");
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

输出：
0号小朋友，占用这台滑梯
1号小朋友，占用这台滑梯
2号小朋友，占用这台滑梯
0号小朋友，离开这台滑梯
2号小朋友，离开这台滑梯
1号小朋友，离开这台滑梯
4号小朋友，占用这台滑梯
3号小朋友，占用这台滑梯
4号小朋友，离开这台滑梯
3号小朋友，离开这台滑梯
```

### 2、原理

#### 2.1、tryAcquire方法

以上4个方法都会被阻塞，如果想立即得到执行结果，可以使用下面几个方法：

```java
//尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire() { };    
//尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { };  
//尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(int permits) { }; 
//尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { }; 

```

另外还可以通过availablePermits()方法得到可用的许可数目。

### 3、总结

对三个辅助类做一下总结：

　　1）CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：

　　　　CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；

　　　　而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；

　　　　另外，CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的。

　　2）Semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限。