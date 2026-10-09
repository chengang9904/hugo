+++
date = '2026-10-09T21:30:00+08:00'
draft = false
title = 'JUC 常用类与面试手撕题'
description = 'Java · 并发、JUC'
categories = ['Java 学习']
tags = ['Java', '并发', 'JUC']
+++

JUC（`java.util.concurrent`）要解决的问题，归根结底是"多个线程同时读写共享状态"引出的三类麻烦：互斥、协调、限流。

> JUC 里的每个类，都是在"要不要阻塞、要不要排队、要不要重试"这几个选择上做了不同取舍。

先过一遍面试和实际项目里最常碰到的几个类，再用几道手撕题检验是不是真的理解。

## ReentrantLock

核心是它比 `synchronized` 多出来的几项能力，而不是"可重入"本身——`synchronized` 本来就可重入。

真正的优势：

- `tryLock()`：拿不到锁直接返回，不会一直阻塞。
- `lockInterruptibly()`：等锁的过程可以被中断。
- 可以配置公平锁，避免同一个线程反复抢到锁，饿死其他线程。
- 可以配合多个 `Condition` 做分组唤醒，而不是像 `notifyAll()` 那样广播给所有等待者。

```java
private final ReentrantLock lock = new ReentrantLock();

public void increment() {
    lock.lock();
    try {
        count++;
    } finally {
        lock.unlock();
    }
}
```

`unlock()` 必须放在 `finally` 里。`synchronized` 的释放是编译器保证的，`ReentrantLock` 的释放要自己负责，这也是它比 `synchronized` 更容易踩雷的地方。

## CountDownLatch 与 CyclicBarrier

两者都是"等所有线程到齐再继续"，区别在于用一次还是能反复用。

- `CountDownLatch`：计数器减到 0 才放行，等待方和计数方可以是不同线程，用完即废。
- `CyclicBarrier`：参与的线程互相等，所有人都调用 `await()` 后一起放行，可以重置重复使用，放行时还能跑一个回调。

```java
CountDownLatch latch = new CountDownLatch(3);

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        // 执行任务
        latch.countDown();
    }).start();
}

latch.await(); // 等三个任务都完成
```

```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> System.out.println("全部到达"));

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        // 执行任务
        barrier.await(); // 等其他线程也到达这里
    }).start();
}
```

一次性汇合用 `CountDownLatch`；需要分阶段反复汇合（比如多轮迭代，每轮都要等大家算完再进下一轮），用 `CyclicBarrier`。

## Semaphore

核心不是互斥，而是控制同时能有多少个线程进入某段代码，典型场景是限流、控制连接池并发数。

```java
Semaphore semaphore = new Semaphore(3); // 最多 3 个并发

public void access() throws InterruptedException {
    semaphore.acquire();
    try {
        // 受限资源的访问逻辑
    } finally {
        semaphore.release();
    }
}
```

许可数为 1 时退化成互斥锁，但和 `ReentrantLock` 不同：`Semaphore` 没有"拥有者"概念，`acquire()` 和 `release()` 可以在不同线程里调用。

## ConcurrentHashMap

核心是怎么在不整体加锁的前提下保证 Map 操作线程安全。

- JDK 1.7：分段锁（`Segment`），数组拆成多段，每段单独加锁，写操作只锁所在段。
- JDK 1.8：去掉分段锁，改成 CAS + `synchronized`：没有哈希冲突时用 CAS 直接写；冲突时只对链表/红黑树的头节点加锁，粒度缩小到单个桶。

容易踩的坑：

- `size()` 不是强一致的，遍历期间发生的修改可能反映也可能不反映，不能拿它做精确计数。
- `computeIfAbsent` 的函数参数里不能再调用同一个 map 的方法，否则容易在同一个桶的锁上发生重入死锁。

## BlockingQueue

核心是把"队列满了阻塞生产者、队列空了阻塞消费者"封装好，不用自己手写 `wait/notify`。

```java
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);

queue.put(1);      // 满了就阻塞
int value = queue.take(); // 空了就阻塞
```

- `ArrayBlockingQueue`：有界，数组实现，一把锁同时管生产和消费。
- `LinkedBlockingQueue`：可选有界，链表实现，生产和消费分别用两把锁，吞吐量更高。

这是后面"生产者消费者"手撕题的标准答案。面试通常会要求不借助它，自己用 `wait/notify` 或 `Condition` 实现一遍。

## ThreadPoolExecutor

核心是不要无脑用 `Executors` 的工厂方法，要清楚线程池内部几个参数的真实含义。

```java
new ThreadPoolExecutor(
        4,                                // 核心线程数
        8,                                // 最大线程数
        60, TimeUnit.SECONDS,             // 空闲线程存活时间
        new LinkedBlockingQueue<>(100),   // 任务队列
        new ThreadPoolExecutor.CallerRunsPolicy() // 拒绝策略
);
```

任务提交后的决策顺序：

- 线程数小于核心线程数，直接新建线程执行。
- 核心线程数已满，任务进队列排队。
- 队列也满了，且线程数小于最大线程数，新建非核心线程执行。
- 线程数也到了最大值，触发拒绝策略。

`Executors.newFixedThreadPool` 底层用的是无界队列，任务堆积时不会触发拒绝策略，而是把内存占满，最终 OOM。这也是不建议直接用它的原因。

拒绝策略四选一：

- `AbortPolicy`：直接抛异常，默认策略。
- `CallerRunsPolicy`：交回调用者线程执行，起到降速作用。
- `DiscardPolicy`：直接丢弃，不抛异常。
- `DiscardOldestPolicy`：丢弃队列里最老的任务，换新任务进去。

## 手撕一：交替打印 1-100

两个线程交替打印，顺序必须严格交替，不能靠 `sleep` 凑时间。

`wait/notify` 写法：

```java
public class AlternatePrint {
    private int count = 1;
    private final int max = 100;
    private final Object lock = new Object();

    public void printOdd() {
        synchronized (lock) {
            while (count <= max) {
                if (count % 2 == 1) {
                    System.out.println(Thread.currentThread().getName() + ": " + count++);
                    lock.notifyAll();
                } else {
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                }
            }
        }
    }

    public void printEven() {
        synchronized (lock) {
            while (count <= max) {
                if (count % 2 == 0) {
                    System.out.println(Thread.currentThread().getName() + ": " + count++);
                    lock.notifyAll();
                } else {
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                }
            }
        }
    }
}
```

> 判断条件必须用 `while` 而不是 `if`。`notifyAll()` 会把两个等待线程都叫醒，被叫醒后要重新确认是不是真的轮到自己，否则同一个数字会被打印两次。

`ReentrantLock` + `Condition` 写法：

```java
public class AlternatePrintWithLock {
    private int count = 1;
    private final int max = 100;
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition oddCondition = lock.newCondition();
    private final Condition evenCondition = lock.newCondition();

    public void printOdd() throws InterruptedException {
        lock.lock();
        try {
            while (count <= max) {
                if (count % 2 == 1) {
                    System.out.println("奇数：" + count++);
                    evenCondition.signal();
                } else {
                    oddCondition.await();
                }
            }
        } finally {
            lock.unlock();
        }
    }

    // printEven 对称：偶数打印时 signal(oddCondition)，否则 evenCondition.await()
}
```

两个 `Condition` 分别只唤醒对应的等待者，不像 `notifyAll()` 要广播再过滤。这正是前面 `ReentrantLock` 一节提到的"分组唤醒"能力的实际用处。

## 手撕二：生产者消费者模型（不借助 BlockingQueue）

自己实现一个有容量上限的队列，满了阻塞生产者，空了阻塞消费者。

```java
public class BoundedQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public BoundedQueue(int capacity) {
        this.capacity = capacity;
    }

    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await();
            }
            queue.offer(item);
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await();
            }
            T item = queue.poll();
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

两个 `Condition` 分别管"不满"和"不空"，生产者只唤醒消费者，消费者只唤醒生产者，不会有无关线程被叫醒之后又去空跑抢锁。这就是 `ArrayBlockingQueue` 内部的实现思路。

## 手撕三：CountDownLatch 控制多线程汇合

三个线程并发抓数据，全部完成后主线程再汇总。

```java
public class DataAggregator {
    public static void main(String[] args) throws InterruptedException {
        int taskCount = 3;
        CountDownLatch latch = new CountDownLatch(taskCount);
        List<String> results = new CopyOnWriteArrayList<>();

        for (int i = 0; i < taskCount; i++) {
            int taskId = i;
            new Thread(() -> {
                try {
                    results.add("task-" + taskId + " done");
                } finally {
                    latch.countDown();
                }
            }).start();
        }

        latch.await();
        System.out.println("全部完成: " + results);
    }
}
```

`countDown()` 要放在 `finally` 里，任务抛异常也要释放计数，否则 `await()` 会一直阻塞。

这里用 `CopyOnWriteArrayList` 收集结果，是因为多个线程并发写入同一个 List，读多写少、不要求写入立刻全局可见，正适合它；用普通 `ArrayList` 会有并发修改异常的风险。

## 手撕四：手写极简线程池

一个线程池最少需要三样东西：固定数量的工作线程、一个任务队列、提交任务的入口。工作线程循环从队列取任务执行，队列空了就阻塞等待。

```java
public class SimpleThreadPool {
    private final BlockingQueue<Runnable> taskQueue;
    private final List<WorkerThread> workers = new ArrayList<>();
    private volatile boolean isShutdown = false;

    public SimpleThreadPool(int poolSize, int queueCapacity) {
        taskQueue = new LinkedBlockingQueue<>(queueCapacity);
        for (int i = 0; i < poolSize; i++) {
            WorkerThread worker = new WorkerThread();
            worker.start();
            workers.add(worker);
        }
    }

    public void submit(Runnable task) {
        if (isShutdown) {
            throw new IllegalStateException("线程池已关闭");
        }
        try {
            taskQueue.put(task);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    public void shutdown() {
        isShutdown = true;
        workers.forEach(Thread::interrupt);
    }

    private class WorkerThread extends Thread {
        @Override
        public void run() {
            while (!isShutdown) {
                try {
                    Runnable task = taskQueue.take();
                    task.run();
                } catch (InterruptedException e) {
                    break;
                }
            }
        }
    }
}
```

任务队列这里直接借用了 `BlockingQueue`，因为手撕的重点是线程池的调度逻辑，不是队列实现——队列实现已经在手撕二里单独考过。

和真正的 `ThreadPoolExecutor` 相比，这版缺了核心线程数/最大线程数的区分、空闲线程回收、拒绝策略。面试时能说清楚"少了什么、为什么真正的线程池需要它们"，通常比写出一个功能完整的版本更重要。

## 小结

- 只是要保证一段代码互斥执行，`synchronized` 够用，不需要上 `ReentrantLock`。
- 需要 `tryLock`、响应中断、公平锁，或者多个等待队列分组唤醒，才用 `ReentrantLock`。
- "等所有人到齐再继续"且只用一次，用 `CountDownLatch`；需要反复汇合，用 `CyclicBarrier`。
- 限制并发访问数量用 `Semaphore`，它和互斥锁的区别是没有"拥有者"。
- 高并发读写的 Map 用 `ConcurrentHashMap`，但不要依赖它的 `size()` 做精确计数。
- 生产者消费者模型优先用 `BlockingQueue`，面试要求手写时才自己拿 `ReentrantLock` + `Condition` 实现。
- 线程池别用 `Executors` 的工厂方法，自己配置 `ThreadPoolExecutor` 的四个核心参数，想清楚队列满了之后系统应该降速还是报错。

面试被问到 JUC，比起背出每个类的方法名，更重要的是能说清楚"为什么需要这个类"——它解决的是互斥、协调还是限流问题，以及它和更简单的方案（`synchronized`、普通 `List`、手写循环）相比，换来了什么，又付出了什么代价。
