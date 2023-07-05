---
layout: post
title: 源码分析之 - AsyncTask
date: 2023-07-06 14:44:45 +0800
categories: [Android源码, AsyncTask]
tags: [Android源码, AsyncTask]
author: 
---

# 源码分析之 - AsyncTask

> `AsyncTask` 已在  `API 30` 中弃用。
>
> 官方推荐使用标准的 `java.util.concurrent`  或  [Kotlin concurrency utilities](https://developer.android.com/topic/libraries/architecture/coroutines)  代替。

## AsyncTask 简介

`AsyncTask` 是 Android 实现异步方式之一，可以在子线程进行数据操作，主线程更新 `UI` 操作。

`AsyncTask` 被设计为一个围绕 Thread 和 Handler 的帮助类，并不构成一个通用的线程框架。

## AsyncTask 基本使用

一个 `AsyncTask` 异步任务由三种通用类型定义 `Params`,  `Progress`,  `Result`, 

以及四个步骤组成  `onPreExecute` ，`doInBackground` ，`onProgressUpdate` ，`onPostExecute` 

必须将 `AsyncTask` 子类化才能使用。子类将至少复写一个 `doInBackground` 方法。

示例：

```java
private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
    
    protected Long doInBackground(URL... urls) {
        int count = urls.length;
        long totalSize = 0;
        for (int i = 0; i < count; i++) {
            totalSize += Downloader.downloadFile(urls[i]);
            publishProgress((int) ((i / (float) count) * 100));
            
            // 如果 cancel() 方法被调用，直接退出
            if (isCancelled()) break;
        }
        return totalSize;
    }

    protected void onProgressUpdate(Integer... progress) {
        setProgressPercent(progress[0]);
    }

    protected void onPostExecute(Long result) {
        showDialog("已下载 " + result + " 字节");
    }
}

// 一旦创建了AsyncTask子类，一个异步任务的执行将非常简单：
new DownloadFilesTask().execute(url1, url2, url3);
```

### 创建说明

首先，在创建 AsyncTask 的时候，需要传入三个泛型数据 `AsyncTask<Params, Progress, Result>` ，其分别对应着

```java
private class DownloadFilesTask extends AsyncTask<Params: URL, Progress: Integer, Result: Long> {}
    
protected abstract Result doInBackground(Params... params);

@MainThread
protected void onProgressUpdate(Progress... values) {
}

@MainThread
protected void onPostExecute(Result result) {
}
```

并非所有的类型都总会被用到, 如果要标记一个参数是未使用到的，可以使用Void类型代替。

### 执行流程

+ `AsyncTask` 调用 `execute(Params... params)` 方法

+ `onPreExecute() ` 在task被执行前调用于`UI`线程，此步骤通常用于准备工作，例如在用户界面显示一个进度条。

+ `doInBackground(Params... params) ` 调用于后台线程，用于执行耗时任务，如数据库操作等

+ 在 `doInBackground` 进行操作的过程中，可以通过 `publishProgress(Progress... values)` 进行进度更新，从而自动调用 `onProgressUpdate(Progress... values) `

+ 当`doInBackground`执行完毕后，返回数据，将会在 `UI` 线程上调用`onPostExecute(Result result)`

### AsyncTask 使用的原则：

1. 必须在 UI 线程上加载 AsyncTask 类，Android 4.1  之后会自动完成此步骤；
2. AsyncTask 实例对象必须在 UI 线程创建；
3. execute() 方法的调用必须在 UI 线程；
4. 不要手动调用四个步骤方法；
5. 该任务只能执行一次。（第二次执行会抛出异常）

### AsyncTask 执行顺序：

+ Android 1.6 之前单线程顺序执行；
+ Android 1.6 之后使用线程池允许同时多个任务并行执行；
+ Android 3.0 开始在单线程执行，为了避免并行执行引起的应用程序错误；
+ 如果真的需要并行执行，可以通过 THREAD_POOL_EXECUTOR 来调用 executeOnExecutor(Executor exec, Params... params)

## AsyncTask 源码分析

### 1. 类定义

public 的抽象类，有三个泛型参数，必须被继承使用

```java
// public 的抽象类，有三个泛型参数，必须被继承使用
public abstract class AsyncTask<Params, Progress, Result> {
}
```

### 2. 构造函数

```java
public AsyncTask() {
    this((Looper) null);
}

/**
* Creates a new asynchronous task. This constructor must be invoked on the UI thread.
*
* @hide
*/
public AsyncTask(@Nullable Handler handler) {
    this(handler != null ? handler.getLooper() : null);
}

/**
 * @hide
 */
public AsyncTask(@Nullable Looper callbackLooper) {
    // 1. 创建Handler, 默认使用主线程的Looper
    mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
        ? getMainHandler() : new Handler(callbackLooper);

    // 2. 创建一个 WorkerRunnable 对象(Callable 接口)
    mWorker = new WorkerRunnable<Params, Result>() {
        @Override
        public Result call() throws Exception {
            mTaskInvoked.set(true); // 表示当前任务已经被调用过了
            Result result = null;
            try {
                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                
                result = doInBackground(mParams); // 执行 doInBackground 方法
                Binder.flushPendingCommands();
            } catch (Throwable tr) {
                mCancelled.set(true);
                throw tr;
            } finally {
                postResult(result); // 将返回值传递给 postResult 方法
            }
            return result;
        }
    };

    // 3. 创建一个 FutureTask 对象, 并接收第2步创建的 Calllable
    mFuture = new FutureTask<Result>(mWorker) {
        @Override
        protected void done() {
            ...
            postResultIfNotInvoked(get());
        }
    };
    
    // call() 和 done() 方法里面的具体内容可以先不用理会
    // 等 mFuture 被线程调用的时候, 就会调用 call()
}

private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
    Params[] mParams;
}

public class FutureTask<V> implements RunnableFuture<V> {
}
```

这里Callable, Runnable, Feature 傻傻分不清, 我们来简单做个对比

+ Runnable:  接口, 表达一种执行耗时任务的能力, 无参, 无返回值
+ Callable:  接口, 无参, 有返回值, 返回值是传入的泛型类型, 并且抛出一个可检查的异常Exception
+ Future:  接口, 表示异步计算的结果, 本身不具备程序执行耗时任务的能力。提供了检查计算是否完成、等待计算完成以及检索计算结果的方法
+ RunnableFeature:  接口, Feature + Runnable 
+ FutureTask:  类, RunnableFeature 的具体实现, 表达了一种可取消的异步计算能力, 本质是一个 Runnable 对象, run() 方法内部会调用 callable 对象的 call() 方法返回结果

### 3. execute 方法

为了分析 AsyncTask 的工作原理，我们从 execute 方法开始分析，execute 方法又会调用 executeOnExecutor 方法：

```java
private volatile Status mStatus = Status.PENDING; // 初始状态为 PENDING

public enum Status {
    PENDING,
    RUNNING,
    FINISHED,
}

@MainThread
public final AsyncTask<Params, Progress, Result> execute(Params... params) {
    // 提交在默认的线程池 sDefaultExecutor 上
    return executeOnExecutor(sDefaultExecutor, params);
}

@MainThread
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
        Params... params) {
    if (mStatus != Status.PENDING) {
        switch (mStatus) {
            case RUNNING:
                // 一个 AsyncTask 只能被执行一次 
                throw new IllegalStateException("Cannot execute task:"
                        + " the task is already running.");
            case FINISHED:
                // 一个 AsyncTask 只能被执行一次 
                throw new IllegalStateException("Cannot execute task:"
                        + " the task has already been executed "
                        + "(a task can be executed only once)");
        }
    }

    mStatus = Status.RUNNING; // 标记为 RUNNING 状态

    onPreExecute(); // 这个方法最先执行

    mWorker.mParams = params; // 参数赋值给 WorkerRunnable 对象的 mParams
    exec.execute(mFuture); // 然后线程池开始执行, mFuture 本质为 Runnable 对象

    return this;
}
```

根据上面代码可知，execute 操作将提交任在默认的线程池 `sDefaultExecutor` 上。

###  4. sDefaultExecutor 线程池

`sDefaultExecutor` 实际上是一个串行的线程池，一个进程中所有的 `AsyncTask` 全部再这个串行的线程池中排队运行。

```java
// 这是一个静态的对象，所以所有的 AsyncTask 任务都在同一个线程池中运行。
public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;

private static class SerialExecutor implements Executor {
    // 数组双端队列, 无容量限制, 自动扩容, 线程不安全, 元素不允许 null
    final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>(); 
    Runnable mActive; // mActive 就是从 mTasks 中取出的 Runnable 对象

    // 这里面的 Runnable 就是 mFuture 对象, 提交时有：exec.execute(mFuture); 
    public synchronized void execute(final Runnable r) {
        // 插入一个元素到队列尾部
        mTasks.offer(new Runnable() {
            public void run() {
                try {
                    r.run();
                } finally {
                    // 当前任务执行完之后, 再去取下一个任务
                    scheduleNext();
                }
            }
        });
        if (mActive == null) {
            scheduleNext(); // 提交之后，取出队列的头节点任务
        }
    }
    
    
    protected synchronized void scheduleNext() {
        // mActive 就是从 mTasks 中取出的 Runnable 对象
        if ((mActive = mTasks.poll()) != null) {
            THREAD_POOL_EXECUTOR.execute(mActive);
        }
    }
}
```

可以看到`THREAD_POOL_EXECUTOR` 才是用于真正的执行任务线程池

### 5. THREAD_POOL_EXECUTOR

`THREAD_POOL_EXECUTOR` 真正的用于执行任务线程池

```java
private static final int CORE_POOL_SIZE = 1; // 核心线程
private static final int MAXIMUM_POOL_SIZE = 20; // 最大线程
private static final int BACKUP_POOL_SIZE = 5; // 
private static final int KEEP_ALIVE_SECONDS = 3; // 非核心线程最大存活时间

private static final ThreadFactory sThreadFactory = new ThreadFactory() {
    private final AtomicInteger mCount = new AtomicInteger(1);

    // 线程工厂方法
    public Thread newThread(Runnable r) {
        return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
    }
};

public static final Executor THREAD_POOL_EXECUTOR;

static {
    ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
        CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
        new SynchronousQueue<Runnable>(), sThreadFactory);
    
    // 拒绝策略, 当线程数达到最大线程数, 并且队列已满时, 新添加的任务根据该策略判断怎么处理
    // 默认有四种策略 (CallerRunsPolicy: 直接在调用线程执行 AbortPolicy: 抛出异常 DiscardPolicy: 直接丢弃 DiscardOldestPolicy: 丢弃最早的未被处理的任务
    // 这里使用的是自定义策略, 新开启一个备用的sBackupExecutor线程池执行新任务
    threadPoolExecutor.setRejectedExecutionHandler(sRunOnSerialPolicy);
    THREAD_POOL_EXECUTOR = threadPoolExecutor;
}
```

具体的任务执行流程就在 `threadPoolExecutor` 的 `execute(Runnable command)` 方法内，关于线程池的源码可以参考:[线程池 Executor]()

当线程池中调用

