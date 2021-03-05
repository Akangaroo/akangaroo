# ReadWriteLock读写锁

`ReadWriteLock` 维护了一对相关的`锁`，一个用于只读操作，另一个用于写入操作。只要没有 writer，`读取锁`可以由多个 reader 线程同时保持。`写入锁`是独占的。

所有 `ReadWriteLock` 实现都必须保证 `writeLock` 操作的内存同步效果也要保持与相关 `readLock` 的联系。也就是说，成功获取读锁的线程会看到写入锁之前版本所做的所有更新。

与互斥锁相比，读-写锁允许对共享数据进行更高级别的并发访问。虽然一次只有一个线程（*writer* 线程）可以修改共享数据，但在许多情况下，任何数量的线程可以同时读取共享数据（*reader* 线程），读-写锁利用了这一点。从理论上讲，与互斥锁相比，使用读-写锁所允许的并发性增强将带来更大的性能提高。在实践中，只有在多处理器上并且只在访问模式适用于共享数据时，才能完全实现并发性增强。

```java
class MyCache {
    private volatile Map<String, Object> map = new HashMap<>();
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public void put(String key, Object value) {
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "---写入数据");
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            map.put(key,value);
            System.out.println(Thread.currentThread().getName() + "---完成");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }

    }

    public void get(String key) {
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "读取");
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            map.get(key);
            System.out.println(Thread.currentThread().getName() + "读取完成");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();

        for (int i = 0; i < 5; i++) {
            final int temp = i;
            new Thread(()->{
                myCache.put(temp + " ", temp + " ");
            }, String.valueOf(i)).start();
        }

        for (int i = 0; i < 5; i++) {
            final int temp = i;
            new Thread(()->{
                myCache.get(temp + " ");
            }, String.valueOf(i)).start();
        }
    }
}
```



