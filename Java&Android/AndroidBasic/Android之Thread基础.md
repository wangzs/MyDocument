# Android之Efficient Thread
## 1. Multithreading in Java
* 一个简单的java多线程例子：
```java
private static class MyTask implements Runnable {
       @Override
       public void run() {
           System.out.println("Run my task: " + Thread.currentThread().getId());
       }
   }

   public static void main(String[] args) {
       Thread myThread = new Thread(new MyTask());
       myThread.start();
       System.out.println("Main thread: " + Thread.currentThread().getId());
   }
```
* Thread类中的run和start函数的区别：
```java
// Thread的start接口
public synchronized void start() {
  // 其他代码省略
  start0(); // 调用native方法，启动新线程
}
// Thread实现了Runnable的接口
@Override
public void run() {
    if (target != null) {
        target.run(); // 直接运行，所以直接调用run函数，是仍然执行在原线程
    }
}
```
* Thread的优先级:
当不explicit设置priority时，thread会使用parent的priority，一般默认为5。
```java
/**
 * The minimum priority that a thread can have.
 */
public final static int MIN_PRIORITY = 1;
/**
 * The default priority that is assigned to a thread.
 */
public final static int NORM_PRIORITY = 5;
/**
 * The maximum priority that a thread can have.
 */
public final static int MAX_PRIORITY = 10;
```
* 一个简单的data racing问题
```java
private static class DataRacing {
    int sharedResource = 0;
    public void startRacingThread() {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                sharedResource++;
                System.out.println("====>0 " + sharedResource);
            }
        });
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                sharedResource--;
                System.out.println("====>1 " + sharedResource);
            }
        });

        t1.start();
        t2.start();
    }
}
// 为sharedResource添加synchronized解决data race的问题
synchronized (this) {
   sharedResource--;
}
```

* 使用intrinsic lock，通过关键字synchronized实现对数据的lock
**Method level：**
```java
synchronized void changeState() {
  sharedResource++;
}
```
**Block-level:**
```java
void changeState() {
    synchronized (this) {
      changeState++;
    }
}
```
**Block-level with other object:**
```java
private final Object otherLock = new Object();
void changeState() {
    synchronized (otherLock) {
      changeState++;
    }
}
```
**Method lock (class instance):**
```java
synchronized static void changeState() {
    staticSharedResource++;
}
```
**Block-level (class instanc):**
```java
static void changeState() {
    synchronized(this.getClass()) {
      staticSharedResource++;
    }
}
```
* 使用explicit locking机制
**ReentrantLoc的使用：**
```java
private int shareData = 1;
private ReentrantLock mLock = new ReentrantLock();
public void changeData() {
    mLock.lock();
    try {
      shareData++;
    } finally {
        mLock.unlock();
    }
}
```
**ReentrantReadWriteLock的使用:**
```java
private int shareData = 1;
private ReentrantReadWriteLock mLock = new ReentrantReadWriteLock();
public void changeData() {
    mLock.writeLock().lock();   // 写锁
    try {
        shareData++;
    } finally {
        mLock.writeLock().unlock();
    }
}
public int readData() {
    mLock.readLock().lock();  // 读锁
    try {
        return shareData;
    } finally {
        mLock.readLock().unlock();
    }
}
```
**一个生产者与消费者的简单例子**
```java
public class ConsumerProducer {
    private LinkedList<Integer> mList = new LinkedList<>();
    private final int LIMIT = 10;
    private Object mLock = new Object();
    public void produce() throws InterruptedException {
        int value = 0;
        while (true) {
            synchronized (mLock) {
                while (mList.size() > LIMIT) {
                    mLock.wait();
                }
                System.out.println("p ====> " + value);
                mList.add(value++);
                mLock.notify();
            }
        }
    }
    public void consume() throws InterruptedException {
        while (true) {
            synchronized (mLock) {
                while (mList.isEmpty()) {
                    mLock.wait();
                }
                int val = mList.removeFirst();
                System.out.println("c ====> " + val);
                mLock.notify();
            }
        }
    }
    public static void main(String[] args) {
        final ConsumerProducer consumerProducer = new ConsumerProducer();
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    consumerProducer.produce();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    consumerProducer.consume();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        t1.start();
        t2.start();
    }
}
```

## 2. Thread on Android
Android默认的线程为UI线程，所有ui组件的更新都必须在该线程，否则会抛`CalledFromWrongthreadException`异常。

## 3. Thread Communication
#### 3.1 Pipes
Java中的pipe不同于linux中的，linux中属于不同process之间的，一个命令的输出重定向为另一个命令的输入；但是java中的pipes工作于同一个process的不同thread之间。
一个简单例子：
```java
private PipedWriter w = new PipedWriter();
private PipedReader r = new PipedReader();
// 绑定write和reader
try {
    w.connect(r);
} catch (IOException e) {
    e.printStackTrace();
}
// 一个线程中write产生随机ascii字符
while (true) {
    try {
        Thread.currentThread().sleep(500);
        w.write(RandomStringUtils.randomAscii(5));  // apache.commons.lang3工具类
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
// 另一个线程消费产生的ascii字符
try {
    int i;
    while ((i = r.read()) != -1) {
        char c = (char) i;
        System.out.print(c);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```




























[]
