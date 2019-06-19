<!-- 多线程之ThreadPoolExecutor和ForkJoinPool的用法 -->


在平时的工作中，当遇到数据量比较大、程序运行较慢，需要提升程序性能时，一般会涉及到多线程。有些小伙伴对多线程的用法不是很清楚，本文主要说明一下 `ThreadPoolExecutor` 和 `ForkJoinPool` 的用法。

# 场景

首先我们假设这样一个场景，有一个接口，用来计算数组的和。接口定义如下：

```java
package mutilthread;

/**
 * 求和的接口
 * @Author: Rebecca
 * @Description:
 * @Date: Created in 2019/6/18 15:28
 * @Modified By:
 */
public interface Calculator {
    long sumUp(int[] numbers) throws Exception;
}

```

# 单线程实现

最开始我们的代码肯定是使用普通的单线程实现，这样的好处是代码比较简单，坏处就是当数据了比较大时，程序运行较慢，无法利用多核CPU。

```java
package mutilthread;

import java.util.ArrayList;
import java.util.List;

/**
 * 单线程的类
 * @Author: Rebecca
 * @Description:
 * @Date: Created in 2019/6/18 10:24
 * @Modified By:
 */
public class SingleThread implements Calculator {
    /**
     * 用单线程计算数组的和
     * @param calcData 需要求和的数组
     * @return
     * @author Rebecca 10:51 2019/6/18
     * @version 1.0
     */
    @Override
    public long sumUp(int[] calcData) {
        // 此句代码只是为了延长程序运行时间，和程序逻辑无关
        List<SingleThread> tasks = new ArrayList<SingleThread>();

        int calcDataLength = calcData.length;
        long sum = 0l;
        for (int i = 0; i < calcDataLength; i++) {
            sum += calcData[i];

            // 此句代码只是为了延长程序运行时间，和程序逻辑无关
            tasks.add(new SingleThread());
        }
        return sum;
    }
}
```

# 多线程实现-`ExecutorService`

因为单线程的劣势严重影响程序处理速度，我们把代码优化为多线程的`ExecutorService`来实现。

```java
package mutilthread;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;
import java.util.concurrent.ThreadPoolExecutor.CallerRunsPolicy;

/**
 * 用 ThreadPoolExecutor 线程池计算数组的和
 * @Author: Rebecca
 * @Description:
 * @Date: Created in 2019/6/18 10:50
 * @Modified By:
 */
public class MutilThreadOfThreadPoolExecutor implements Calculator {

    /**
     * 用 ThreadPoolExecutor 线程池计算数组的和
     * @param calcData 需要求和的数组
     * @return
     * @author Rebecca 10:51 2019/6/18
     * @version 1.0
     */
    @Override
    public long sumUp(int[] calcData) throws Exception {
        // 创建线程池
        ExecutorService executorService = new ThreadPoolExecutor(5, 10, // 线城数
                60l, TimeUnit.SECONDS,  // 超时时间
                new ArrayBlockingQueue<Runnable>(100, true),  // 线程处理数据的方式
                Executors.defaultThreadFactory(),  // 创建线程的工厂
                new CallerRunsPolicy());  // 超出处理范围的处理方式


        int calcDataLength = calcData.length;
        long sum = 0l;
        int threadSize = 5;

        for (int i = 0; i < threadSize; i++) {
            int arrStart = calcDataLength / threadSize * i;
            int arrEnd = calcDataLength / threadSize * (i+1);

            SumTask task = new SumTask(calcData, arrStart, arrEnd);
            // 线程池处理数据
            Future<Long> future = executorService.submit(task);

            sum += future.get().longValue();
        }
        // 关闭线程池
        executorService.shutdown();

        return sum;
    }


    public static class SumTask implements Callable<Long> {
        private int[] arr;
        private int start, end;

        public SumTask() {}

        public SumTask(int[] arr, int start, int end)
        {
            this.arr = arr;
            this.start = start;
            this.end = end;
        }

        @Override
        public Long call()
        {
            // 此句代码只是为了延长程序运行时间，和程序逻辑无关
            List<SumTask> tasks = new ArrayList<SumTask>();

            long sum = 0l;
            for (int i = start; i < end; i++)
            {
                sum += arr[i];
                // 此句代码只是为了延长程序运行时间，和程序逻辑无关
                tasks.add(new SumTask());
            }

            return sum;
        }
    }
}
```

`Executors`也提供了一些方法，可以直接创建`ExecutorService`线程池，如`newSingleThreadExecutor()`、`newCachedThreadPool()`、`newFixedThreadPool()`、`newScheduledThreadPool()`，相比于`ThreadPoolExecutor`提供的构造函数，`Executors`提供的方法只用传2个参数甚至更少，但`new ThreadPoolExecutor()`则要传一堆参数。那么我们为什么还要用`new ThreadPoolExecutor()`这种方式呢？

答案很简单，为了不让程序出现OOM。如果你看过`Executors`构造线程池相关方法的源码就会发现，它内部也是用`new ThreadPoolExecutor()`方式创建的线程池。但有一个参数它传的是`Integer.MAX_VALUE`。这个参数是什么意思呢？即线程池中允许出现的线程最大数量。如果线程池中真的创建了`Integer.MAX_VALUE`的线程数，程序肯定会OOM的。

```java
// Executors的newCachedThreadPool方法源码
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>(),
                                  threadFactory);
}
```

为了避免这种情况，我们一般用`new ThreadPoolExecutor()`这种方式创建线程池。那么这么多参数分别是什么意思呢？

别急，其实我们可以分组记忆：

第1组（线程数量相关的）：
1. corePoolSize: 核心线程数。即使线程池中没有任务，这些线程也不会被销毁，因为创建和销毁线程是需要消耗CPU资源的
2. maximumPoolSize: 线程池中允许创建的最大线程数

第2组（非核心线程销毁时间相关的）：
1. keepAliveTime: 非核心线程的销毁时间。非核心线程不可能一直在线程池中占用资源，所以需要销毁
2. unit: 销毁的时间单位。可以为`TimeUnit`中的枚举类型

第3组（线程池处理数据相关的）：
1. workQueue: 线程处理数据的方式。一般用JDK提供的`ArrayBlockingQueue`(数组)和`LinkedBlockingDeque`(链表)
2. handler: 超出处理范围的处理方式。<br> `AbortPolicy` : 如果超出处理范围，则抛`RejectedExecutionException`异常； <br> `CallerRunsPolicy` : 如果超出处理范围，则用调用该线程池的线程处理； <br> `DiscardOldestPolicy`: 如果超出处理范围，则把最旧的元素删除，保留新的元素 <br>`DiscardPolicy`: 如果超出处理范围，则不处理，丢弃掉

第4组（创建线程的工厂）：
1. threadFactory: 创建线程的工厂，一般我们用 `Executors.defaultThreadFactory()` 即可

```java
// ThreadPoolExecutor的构造方法源码
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize,
                          long keepAliveTime, TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

假设我们有一串任务，被分为3组，每组任务数量为3，线程池中只有3个线程来处理，那么处理顺序则如下所示：

第1步：

任务组1 被 线程1 处理，线程1 处理任务组1中的第一个任务；
任务组2 被 线程2 处理，线程2 处理任务组2中的第一个任务；
任务组3 被 线程3 处理，线程3 处理任务组3中的第一个任务；

第2步：

线程2处理的较快，任务组2中的所有任务都处理完了，因为没有任务组是等待处理的状态，所以线程2此时是空闲状态。此时 线程1 处理的任务组1只处理了第1个任务，那么有没有办法让线程2把任务组1里的第二个任务偷过来处理一下，减少等待时间呢？

在JDK7之后，提供了`ForkJoinPool`线程池就可以实现啦~ 接着往下看吧

# 多线程实现-`ForkJoinPool`

我们用求和的例子来模拟偷任务。

```java
package mutilthread;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * 用 ForkJoinPool 线程池计算数组的和
 * @Author: Rebecca
 * @Description:
 * @Date: Created in 2019/6/18 10:50
 * @Modified By:
 */
public class MutilThreadOfForkJoinPool implements Calculator {

    private ForkJoinPool pool;

    public MutilThreadOfForkJoinPool() {
        // jdk8之后可以用公用的 ForkJoinPool: pool = ForkJoinPool.commonPool()
        pool = new ForkJoinPool();
    }

    /**
     * 用 ForkJoinPool 线程池计算数组的和
     * @param calcData 需要求和的数组
     * @return
     * @author Rebecca 10:51 2019/6/18
     * @version 1.0
     */
    @Override
    public long sumUp(int[] calcData) {
        SumTask task = new SumTask(calcData, 0, calcData.length - 1);
        return pool.invoke(task);
    }


    public static class SumTask extends RecursiveTask<Long> {
        private int[] numbers;
        private int start;
        private int end;

        private SumTask(){}

        public SumTask(int[] numbers, int start, int end) {
            this.numbers = numbers;
            this.start = start;
            this.end = end;
        }

        @Override
        protected Long compute() {
            // 当需要计算的数字小于 10万 时，直接计算结果
            if (end - start < 1000000) {
                long total = 0;

                // 此句代码只是为了延长程序运行时间，和程序逻辑无关
                List<SumTask> tasks = new ArrayList<SumTask>();
                for (int i = start; i <= end; i++) {
                    total += numbers[i];
                    // 此句代码只是为了延长程序运行时间，和程序逻辑无关
                    tasks.add(new SumTask());
                }
                return total;
            } else {  // 否则，把任务一分为二，递归计算
                int middle = (start + end) / 2;
                SumTask taskLeft = new SumTask(numbers, start, middle);
                SumTask taskRight = new SumTask(numbers, middle + 1, end);
                taskLeft.fork();
                taskRight.fork();
                return taskLeft.join() + taskRight.join();
            }
        }
    }
}
```

`RecursiveTask`的`fork`方法和`Thread`的`start`方法是类似的。这种“偷任务”的专业名词叫[工作窃取(work-stealing)算法](https://blog.csdn.net/pange1991/article/details/80944797)，利用JDK7提供的`ForkJoinPool`就可以实现啦。

# 测试

下面是测试类代码

```java
package mutilThread;

import mutilthread.CalcData;
import mutilthread.MutilThreadOfForkJoinPool;
import mutilthread.MutilThreadOfThreadPoolExecutor;
import mutilthread.SingleThread;
import org.junit.Test;

/**
 * 线程测试类
 * @Author: Rebecca
 * @Description:
 * @Date: Created in 2019/6/18 10:40
 * @Modified By:
 */
public class ThreadTest {

    @Test
    public void testThread() throws Exception {
        int[] data = CalcData.getCalcData();
        // 单线程测试
        SingleThread singleThread = new SingleThread();
        long startTime = System.currentTimeMillis();
        System.out.println("数组的和： " + singleThread.sumUp(data));
        System.out.println("单线程耗时： " + (System.currentTimeMillis() - startTime) + " ms");

        // 多线程(ThreadPoolExecutor)测试
        MutilThreadOfThreadPoolExecutor threadPool = new MutilThreadOfThreadPoolExecutor();
        startTime = System.currentTimeMillis();
        System.out.println("数组的和： " + threadPool.sumUp(data));
        System.out.println("多线程(ThreadPoolExecutor)耗时： " + (System.currentTimeMillis() - startTime) + " ms");

        // 多线程(ForkJoinPool)测试
        MutilThreadOfForkJoinPool forkJoinPool = new MutilThreadOfForkJoinPool();
        startTime = System.currentTimeMillis();
        System.out.println("数组的和： " + forkJoinPool.sumUp(data));
        System.out.println("多线程(ForkJoinPool)耗时： " + (System.currentTimeMillis() - startTime) + " ms");
    }
}
```

程序运行结果：
```
数组的和： 499913683383
单线程耗时： 3307 ms
数组的和： 499913683383
多线程(ThreadPoolExecutor)耗时： 197 ms
数组的和： 499913683383
多线程(ForkJoinPool)耗时： 169 ms
```

整理成表格如下：

|线程类型|耗时(ms)|
|---|---|
|单线程|3307|
|多线程(ThreadPoolExecutor)|197|
|多线程(ForkJoinPool)|169|

# 总结

1. 一般我们使用多线程时会用`ExecuterService`，构造用`new ThreadPoolExecutor()`，一般不使用`Executors`提供了构造线线程池方法，避免出现OOM；
2. 线程池相对于线程组(本文没提到)更好管理；
3. 在JDK7之后可以用`ForkJoinPool`，相对于`ExecuterService`执行效率更快。
4. 线程之间通信是需要成本的。<br> 如果你细心的话，会发现上面的示例代码中都有这么两行多余的代码：

```java
// 此句代码只是为了延长程序运行时间，和程序逻辑无关
List<SumTask> tasks = new ArrayList<SumTask>();

// 此句代码只是为了延长程序运行时间，和程序逻辑无关
tasks.add(new SumTask());
```
如果不加创建对象的多余代码，只是单纯累加数组的和，你会发现单线程执行效率更高。所以在实际使用中还是要根据实际业务逻辑对比，选取适合的方式。如果业务逻辑很简单，程序处理跟快，就完全没有必要使用多线程了。<br> 在`ForkJoinPool`中，设置的数组大小是10万，之所以设置这个数字，是为了跟 `ExecutorService` 方式做对比，如果在`ForkJoinPool`中设置的数组长度过小，就会出现性能不如 `ExecutorService` 的情况。


---

程序中用到的生成计算数据的类

```java
package mutilthread;

import java.util.Random;

/**
 * 生成计算数据的类
 * @Author: Rebecca
 * @Description:
 * @Date: Created in 2019/6/18 10:25
 * @Modified By:
 */
public class CalcData {
    // 长度为1000万
    private static int calcDataLength = 10000000;

    public static int[] getCalcData() {
        Random random = new Random();
        int[] calcData = new int[calcDataLength];
        for (int i = 0; i < calcDataLength; i++) {
            // 0~10的随机数  生成[m,n]范围内指定的随机数： rand.nextInt(n -m + 1) +m;
            calcData[i] = random.nextInt(100001);
        }
        return calcData;
    }
}
```

# 参考链接

[Java 并发编程笔记：如何使用 ForkJoinPool 以及原理](http://blog.dyngr.com/blog/2016/09/15/java-forkjoinpool-internals/)

[Java并发 之 线程池系列 (2) 使用ThreadPoolExecutor构造线程池](https://juejin.im/post/5ca21380e51d450ecd7b7f9f#heading-9)
