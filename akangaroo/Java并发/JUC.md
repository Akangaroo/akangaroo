















## 分支合并之ForkJoin

### 简介

[可以参考的博客](https://blog.csdn.net/tyrroo/article/details/81390202)

Fork：把一个复杂任务进行分拆，大事化小

Join：把分拆任务的结果进行合并



### 相关类

ForkJoinPool：分支合并池    类比=>   线程池

![image-20210117164918552](JUC.assets/image-20210117164918552.png)

ForkJoinTask：ForkJoinTask    类比=>   FutureTask

![image-20210117164946270](JUC.assets/image-20210117164946270.png)

RecursiveTask：递归任务：继承后可以实现递归(自己调自己)调用的任务

![image-20210117164958423](JUC.assets/image-20210117164958423.png)

### 案例

```java
class Fibonacci extends RecursiveTask<Integer> {
    final int n;
    Fibonacci(int n) { this.n = n; }

    @Override
    protected Integer compute() {
        if (n <= 1)
            return n;
        Fibonacci f1 = new Fibonacci(n - 1);
        f1.fork();
        Fibonacci f2 = new Fibonacci(n - 2);
        f2.fork();
        return f2.join() + f1.join();
    }
}
```





```java
class MyTask extends RecursiveTask<Integer> {

    private static final Integer ADJUST_VALUE = 10;
    private int begin;
    private int end;
    private int result;

    public MyTask(int begin, int end) {
        this.begin = begin;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        if (end - begin <= ADJUST_VALUE) {
            for (int i = begin; i < end; i++) {
                result = result + i;
            }
        } else {
            int middle = (end + begin) / 2;
            MyTask task01 = new MyTask(begin, middle);
            MyTask task02 = new MyTask(middle + 1, end);
            task01.fork();
            task02.fork();
            result = task01.join() + task02.join();
        }
        return null;
    }
}

public class ForkJoinPoolDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyTask myTask = new MyTask(0, 100);
        ForkJoinPool threadPool = new ForkJoinPool();
        ForkJoinTask<Integer> forkJoinTask = threadPool.submit(myTask);
        System.out.println(forkJoinTask.get());
        threadPool.shutdown();
    }
}
```





## 异步回调

[学习博客](https://www.cnblogs.com/cjsblog/p/9267163.html)

所谓**异步调用**其实就是**实现一个可无需等待被调用函数的返回值而让操作继续运行**的方法。在 Java 语言中，简单的讲就是另启一个线程来完成调用中的部分计算，使调用继续运行或返回，而不需要等待计算结果。但调用者仍需要取线程的计算结果。



案例：

```java
public class CompletableFutureDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 没有返回值
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "--> 没有返回值");
        });
        completableFuture.get();

        // 异步回调, 需要返回值
        CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "--> CompletableFuture2");
            int t = 10 / 0;
            return 1024;
        });
        completableFuture2.whenComplete((t, u) -> {
            System.out.println("****t : " + t);
            System.out.println("****u : " + u); //代表异常返回
        }).exceptionally(f -> {
            System.out.println("**** exception : " + f.getMessage());
            return 404;
        });

    }
}

运行结果：
ForkJoinPool.commonPool-worker-1--> 没有返回值
ForkJoinPool.commonPool-worker-1--> CompletableFuture2
****t : null
****u : java.util.concurrent.CompletionException: java.lang.ArithmeticException: / by zero
**** exception : java.lang.ArithmeticException: / by zero
```



