# Lock接口

`java.util.concurrent.locks` 包下的。

多线程编程：**线程** **操作** **资源类**

​	实现步骤：1. 创建资源类，2. 资源类里创建同步方法，同步代码块。

​	这样用来达到**高内聚低耦合**的目的。



## ReentrantLock可重入锁

Lock的实现类。

可重入锁：什么是 “可重入”，可重入就是说某个线程已经获得某个锁，可以再次获取锁而不会出现死锁。

[可重入锁介绍](https://blog.csdn.net/w8y56f/article/details/89554060)

```java
class X {
   private final Lock lock = new ReentrantLock();
   // ...
 
   public void m() {
     lock.lock();  // 
     try {
       // ... method body
     } finally {
       lock.unlock(); // 释放锁
     }
   }
 }

```

## synchronized和Lock区别

| synchronized                                                 | Lock                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| java内置关键字                                               | java类                                                       |
| 无法判断是否获取锁的状态                                     | 可以判断是否获取到锁（tryLock()方法）                        |
| synchronized会自动释放锁(a 线程执行完同步代码会释放锁 ；b 线程执行过程中发生异常会释放锁) | Lock需在finally中手工释放锁（unlock()方法释放锁），否则容易造成线程死锁 |
| 用synchronized关键字的两个线程1和线程2，如果当前线程1获得锁，线程2线程等待。如果线程1阻塞，线程2则会一直等待下去 | Lock锁就不一定会等待下去，如果尝试获取不到锁，线程可以不用一直等待就结束了 |
| synchronized的锁可重入、不可中断、非公平                     | Lock锁可重入、可判断、可公平                                 |
| synchronized锁适合代码少量的同步问题                         | Lock锁适合大量同步的代码的同步问题                           |



```java
Lock lock = ...;
if(lock.tryLock()) {
      try{
         //处理任务
　　 }catch(Exception ex){
     }finally{
          lock.unlock();   //释放锁
　　}
}else {
  //如果不能获取锁，则直接做其他事情
}
```



> 通过`public static native boolean holdsLock(Object obj);`方法可以判断当前线程是否获取到锁。
>
> ```java
> public class Demo02 {
> 
>  private static  Object lock = new Object();
> 
>  public static void main(String[] args) {
>      new Thread(() -> {
>          synchronized (lock) {
>              System.out.println("A:" + Thread.holdsLock(lock));
>          }
>      }, "A").start();
>      System.out.println("main:" + Thread.holdsLock(lock));
>  }
> }
> 
> 结果：
> main:false
> A:true
> 
> 结论：
> 判断的是当前线程是否持有锁
> ```



## SaleTicket



```java
class Ticket {

    private int number = 30;
    private Lock lock = new ReentrantLock();

    public void saleTicket() {
        lock.lock();
        try {
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "---" + number--);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

public class SaleTicket {
    public static void main(String[] args) {
        Ticket t = new Ticket();
        new Thread(() -> { for (int i = 0; i < 40; i++) { t.saleTicket(); } }, "AA").start();
        new Thread(() -> { for (int i = 0; i < 40; i++) { t.saleTicket(); } }, "BB").start();
        new Thread(() -> { for (int i = 0; i < 40; i++) { t.saleTicket(); } }, "CC").start();
    }
}

```



# 线程间通信

## 面试题

两个线程，一个线程打印1-52，另一个打印字母A-Z打印顺序为12A34B...5152Z,
要求用线程间通信

```java
class Print {
    private Integer count = 1;
    private Integer num = 1;
    private char letter = 'A';

    public synchronized void printNum() throws InterruptedException {
        while (count % 3 == 0) {
            this.wait();
        }
        count++;
        System.out.print(num++);
        this.notifyAll();
    }

    public synchronized void printLetter() throws InterruptedException {
        while (count % 3 != 0) {
            this.wait();
        }
        count++;
        System.out.print(letter++);
        this.notifyAll();
    }
}

public class Test {
    public static void main(String[] args) {
        Print print = new Print();

        new Thread(() -> {
            for (int i = 0; i < 52; i++) {
                try {
                    print.printNum();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 26; i++) {
                try {
                    print.printLetter();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

    }
}

```



多线程的编程模板：

1. 判断
2. 干活
3. 通知



## synchronized实现

需求：多线程操作资源类，一个+1，另外一个-1。

```java
class Item {
    private int number = 0;

    public synchronized void increment() throws InterruptedException {
        // 判断
        while (number != 0) {
            this.wait();
        }
        // 干活
        this.number++;
        System.out.println(Thread.currentThread().getName() + "-->" + number);
        // 通知
        this.notifyAll();
    }

    public synchronized void decrement() throws InterruptedException {
        // 判断
        while (number == 0) {
            this.wait();
        }
        // 干活
        this.number--;
        System.out.println(Thread.currentThread().getName() + "-->" + number);
        // 通知
        this.notifyAll();
    }
}
public class ThreadWaitNotify {
    public static void main(String[] args) {
        Item item = new Item();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(400);
                    item.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(400);
                    item.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(400);
                    item.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(400);
                    item.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}

```



注意：**判断的时候用while而不能使用if**。

原因：因为线程同步必需保证获得锁后从上到下完整的执行,如wait()期间,另外线程notify()后,不能保证获得锁后条件是否成立。

例子：考虑两个+1线程的同样的方法**先后获得了锁**，然后都调用了wait进入阻塞队列（wait会释放锁），然后-1线程消耗了一个资源后唤醒所有。如果使用if的话，那么两个线程就不会判断，直接再次+1。这时候需要再进行一次判断，即当wait()被唤醒之后，再次判断一遍条件是否成立，所以需要使用while。



Synchronized实现使用`wait`和`notify`来进行线程间的通信。

## Lock实现

`Condition` 将 `Object` 监视器方法（`wait`、`notify` 和 `notifyAll`）分解成截然不同的对象，以便通过将这些对象与任意 `Lock` 实现组合使用，为每个对象提供多个等待 set（wait-set）。其中，`Lock` 替代了 `synchronized` 方法和语句的使用，`Condition` 替代了 Object 监视器方法的使用。

```java
class Item2 {
    private int number = 0;//初始值为零的一个变量

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void increment() throws InterruptedException {

        lock.lock();
        try {
            while (number != 0) {
                condition.await();
            }
            ++number;
            System.out.println(Thread.currentThread().getName() + " \t " + number);
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }


    public void decrement() throws InterruptedException {
        lock.lock();
        try {
            while (number != 1) {
                condition.await();
            }
            --number;
            System.out.println(Thread.currentThread().getName() + " \t " + number);
            //通知
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    
}

```



## 精准通知精准唤醒

需求：

多个线程间按顺序调用，实现A-B-C

三个线程启动，要求如下：

AA打印5次，BB打印10次，CC打印15次

```java
public class ThreadOrderAccess {
    public static void main(String[] args) {
        ShareResource shareResource = new ShareResource();

        new Thread(() -> {
            shareResource.print5();
        }, "A").start();
        new Thread(() -> {
            shareResource.print10();
        }, "B").start();
        new Thread(() -> {
            shareResource.print15();
        }, "C").start();
    }
}
class ShareResource {
    private int number = 1; // 1:A,2:B,3:C
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();
	
    // 打印五次
    public void print5() {
        lock.lock();
        try {
            while (number != 1) {
                condition1.await();
            }
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "-->" + number);
            }
            number = 2;
            condition2.signal();// 这时候只唤醒number等于2的
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    // 打印10次
    public void print10() {
        lock.lock();
        try {
            while (number != 2) {
                condition2.await();
            }
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "-->" + number);
            }
            number = 3;
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    
    // 打印15次
    public void print15() {
        lock.lock();
        try {
            while (number != 3) {
                condition3.await();
            }
            for (int i = 0; i < 15; i++) {
                System.out.println(Thread.currentThread().getName() + "-->" + number);
            }
            number = 1;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}


程序运行结果：
A-->1
A-->1
A-->1
A-->1
A-->1
B-->2
B-->2
B-->2
B-->2
B-->2
B-->2
B-->2
B-->2
B-->2
B-->2
C-->3
C-->3
C-->3
C-->3
C-->3
C-->3
C-->3
C-->3
C-->3
C-->3
C-->3
C-->3
C-->3
C-->3
C-->3
```



