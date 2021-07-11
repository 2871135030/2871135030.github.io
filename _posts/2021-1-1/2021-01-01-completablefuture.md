---
layout: post
title: 关于使用CompletableFuture过程中线程等待的问题
date:   2020-11-13 03:12:09
categories: [java]
---

##### 关于使用CompletableFuture过程中线程等待的问题

> 在电商的应用场景中，通过异步多线程获取服务端信息比较常见，如用户打开个人中心查看个人综合信息，可能会展示用户的账户余额、优惠券、积分、消费红包等等信息，这时服务端就会通过异步线程将所需信息汇总后一并返回给用户。如果按单线程逐一返回个人信息，用户等待的时间显然是不能接受的，通过异步多线程的方式大大减少请求的响应时间。

> `jdk8`简化了异步任务的写法，提供了很多异步任务的计算方式。

###### 1、示例（先从一个简单测试示例代码，查看运行的结果）

```
import lombok.extern.slf4j.Slf4j;

import java.time.Duration;
import java.time.Instant;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.CompletableFuture;

@Slf4j
public class CompletableFutureApplicationTests {

    public static void main(String[] args) {
        log.info("当前cpu核数:{}", Runtime.getRuntime().availableProcessors());
        Instant start = Instant.now();
        List<CompletableFuture<Integer>> list = new ArrayList<>();
        //模拟任务需要10个调用
        for (int i = 0; i < 10; i++) {
            list.add(CompletableFuture.supplyAsync(() -> info()));
        }

        list.forEach(e -> {
            try {
                log.info("{}", e.get());
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        });

        Instant end = Instant.now();
        log.info("执行任务完成,耗时共:{}ms", Duration.between(start, end).toMillis());

    }

    public static Integer info() {
        //模拟任务耗时
        try {
            Thread.sleep(1000);
            log.info("{}",Thread.currentThread().getName());
        } finally {
            return new Random().nextInt(1000);
        }
    }
}
```

> 执行此程序，运行结果如下

```
09:20:49.603 [main] INFO * - 当前cpu核数:4
09:20:50.672 [ForkJoinPool.commonPool-worker-1] INFO * - ForkJoinPool.commonPool-worker-1
09:20:50.672 [main] INFO * - 750
09:20:50.673 [ForkJoinPool.commonPool-worker-2] INFO * - ForkJoinPool.commonPool-worker-2
09:20:50.673 [ForkJoinPool.commonPool-worker-3] INFO * - ForkJoinPool.commonPool-worker-3
09:20:50.673 [main] INFO * - 941
09:20:50.673 [main] INFO * - 480
09:20:51.672 [ForkJoinPool.commonPool-worker-1] INFO * - ForkJoinPool.commonPool-worker-1
09:20:51.672 [main] INFO * - 939
09:20:51.673 [ForkJoinPool.commonPool-worker-2] INFO * - ForkJoinPool.commonPool-worker-2
09:20:51.673 [ForkJoinPool.commonPool-worker-3] INFO * - ForkJoinPool.commonPool-worker-3
09:20:51.673 [main] INFO * - 722
09:20:51.673 [main] INFO * - 781
09:20:52.673 [ForkJoinPool.commonPool-worker-1] INFO * - ForkJoinPool.commonPool-worker-1
09:20:52.673 [main] INFO * - 868
09:20:52.674 [ForkJoinPool.commonPool-worker-2] INFO * - ForkJoinPool.commonPool-worker-2
09:20:52.674 [main] INFO * - 632
09:20:52.674 [ForkJoinPool.commonPool-worker-3] INFO * - ForkJoinPool.commonPool-worker-3
09:20:52.678 [main] INFO * - 471
09:20:53.673 [ForkJoinPool.commonPool-worker-1] INFO * - ForkJoinPool.commonPool-worker-1
09:20:53.673 [main] INFO * - 498
09:20:53.687 [main] INFO * - 执行任务完成,耗时共:4066ms
```

###### 2、结果分析（dump线程等待）

> 可以看到在cpu核心数为4个的机器上运行，其运行结果超过4秒，如果按异步线程运行的话，耗时应该是在1秒多一点才符合预期。

> 从打印的日志信息中可以看到`main`主线程的时间在`50 51 52 53`秒返回，子线程`ForkJoinPool.commonPool-worker`的最大编号为3，而且是只3个子线程循环执行任务，说明10个任务同时调用时发生了线程等待，导致结果不符合预期。

* 跟踪源码方法分析

> 查看`supplyAsync`方法源码

```
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
        return asyncSupplyStage(asyncPool, supplier);
    }
```

> 在此主要查看`asyncPool`参数，看起来像是异步线程池

```
    private static final Executor asyncPool = useCommonPool ?
        ForkJoinPool.commonPool() : new ThreadPerTaskExecutor();
```

> 看到是一个变量，根据`useCommonPool`判断创建的方式，继续看`useCommonPool`参数

```
private static final boolean useCommonPool =
        (ForkJoinPool.getCommonPoolParallelism() > 1);
```

> 继续查看方法`getCommonPoolParallelism`中的`commonParallelism`取值，从静态代码段中获取（只贴出关键部分）

```
        common = java.security.AccessController.doPrivileged
            (new java.security.PrivilegedAction<ForkJoinPool>() {
                public ForkJoinPool run() { return makeCommonPool(); }});
        int par = common.config & SMASK; // report 1 even if threads disabled
        commonParallelism = par > 0 ? par : 1;
```

> 通过位操作，获取`par`的值，继续跟踪参数`common.config`，再重点看`makeCommonPool`方法（只贴出关键部分）

```
        int parallelism = -1;
        try {  // ignore exceptions in accessing/parsing properties
            String pp = System.getProperty
                ("java.util.concurrent.ForkJoinPool.common.parallelism");
        ...
        if (parallelism < 0 && // default 1 less than #cores
            (parallelism = Runtime.getRuntime().availableProcessors() - 1) <= 0)
            parallelism = 1;
        if (parallelism > MAX_CAP)
            parallelism = MAX_CAP;
```

> 参数`parallelism`初始赋值为-1，可通过系统参数`java.util.concurrent.ForkJoinPool.common.parallelism`变更并发数，同时增加了并发数范围的控制，<font color=red>需要特别注意`parallelism`在`if`条件中赋值，核数大于1赋值后为false</font>。举例：如cpu只有1核的话，通过`Runtime.getRuntime().availableProcessors()`获取到cpu核心数，计算并发数只为1，按上面的程序如果在1核环境下需要10秒（可以vmware上创建1核心环境安装系统运行查看结果），如cpu核数大于1的话则并发数为cpu核数减1。

###### 3、配置系统参数，设置并发数为10

```
增加启动参数
-Djava.util.concurrent.ForkJoinPool.common.parallelism=10
```

> 运行结果即可观察到可以并行启动10个线程执行任务，总耗时在1秒范围；如果改为9个并发则在2秒的范围。

###### 4、使用线程模式

> 查看方法`CompletableFuture.supplyAsync`，可传入参数`Executor`，如果传入一个线程池最大线程只有一个的线程，则预期结果将在10秒范围，代码如下，可验证结果

```
public static void main(String[] args) {
        log.info("当前cpu核数:{}", Runtime.getRuntime().availableProcessors());
        Instant start = Instant.now();
        List<CompletableFuture<Integer>> list = new ArrayList<>();
        ExecutorService executor = Executors.newFixedThreadPool(1);
        //模拟任务需要10个调用
        for (int i = 0; i < 10; i++) {
            list.add(CompletableFuture.supplyAsync(() -> info(), executor));
        }

        list.forEach(e -> {
            try {
                log.info("{}", e.get());
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        });

        Instant end = Instant.now();
        log.info("执行任务完成,耗时共:{}ms", Duration.between(start, end).toMillis());
        executor.shutdown();
    }
```

> 调整线程数为10，即可以1秒范围内执行完成

###### 5、`CompletableFuture`常用方法基本使用

|方法名|说明|
|-|-|
|CompletableFuture.runAsync|无返回值运行任务|
|CompletableFuture.suppliAsync|有返回值运行任务|
|CompletableFuture.allOf(...).get()|多个任务全部执行完成后才继续后面处理|
|CompletableFuture.anyOf(...).get()|多个任务任何一个执行完成后即继续后面处理|
|CompletableFuture.thenApply|接收执行结果，有返回值，可将返回值继续往下传递|
|CompletableFuture.thenAccept|接收执行结果，无返回值|


###### 6、总结

> 针对不同的场景需要使用不同的线程方式

* cpu密集型的任务，建议直接使用cpu的核心数创建线程，以避免频繁的线程切换造成线程等待；
* io密集型的任务，建议使用线程方式，一般io等待时间远远超过线程等待时间；


