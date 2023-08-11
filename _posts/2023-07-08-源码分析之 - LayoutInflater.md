---
layout: post
title: 源码分析之 - LayoutInflater
date: 2023-07-08 14:44:45 +0800
categories: [Android源码, LayoutInflater]
tags: [Android源码, LayoutInflater]
author: 
---

> 参考:
>
> [Android | 带你探究 LayoutInflater 布局解析原理-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/1061544)

## 0. 前言

`LayoutInflater`是Android开发中的一个重要类，它用于将XML布局文件转换为相应的View对象。负责解析XML布局并在内存中构建对应的视图层次结构。

注意：此类不是线程 安全的，给定的实例只能由单个线程访问。

## 1. 获取 LayoutInflater 对象

```java
@SystemService(Context.LAYOUT_INFLATER_SERVICE)
public abstract class LayoutInflater {
}
```

首先，你要获得 LayoutInflater  的实例，由于 LayoutInflater 是抽象类，不能直接创建对象，因此这里总结一下获取 LayoutInflater 对象的方法。具体如下：

1.  **`View.inflate(...)`**

   ```java
   public static View inflate(Context context, @LayoutRes int resource, ViewGroup root) {
       LayoutInflater factory = LayoutInflater.from(context);
       return factory.inflate(resource, root);
   }
   ```

   

2.  **`Activity#getLayoutInflater()`**

   ```java
   @NonNull
   public LayoutInflater getLayoutInflater() {
       return getWindow().getLayoutInflater();
   }
   ```

   

3.  **`PhoneWindow#getLayoutInflater()`**

   ```java
   @UnsupportedAppUsage
   public PhoneWindow(@UiContext Context context) {
       super(context);
       mLayoutInflater = LayoutInflater.from(context);
   }
   
   @Override
   public LayoutInflater getLayoutInflater() {
       return mLayoutInflater;
   }
   ```

   

4.  **`LayoutInflater#from(Context)`**

   ```java
   public static LayoutInflater from(@UiContext Context context) {
       LayoutInflater LayoutInflater =
           (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
       if (LayoutInflater == null) {
           throw new AssertionError("LayoutInflater not found.");
       }
       return LayoutInflater;
   }
   ```

可以看到，前面 3 种方法最后走到`LayoutInflater#from(Context)`，这其实也是平时用的最多的方式。

现在，我们看`getSystemService(...)`内的逻辑：

如果仅关注 inflate 流程, 这块可以跳过

## 2. ContextImpl#getSystemService流程

这部分我们以Activity#getLayoutInflater来讲解LAYOUT_INFLATER_SERVICE的获取流程

```java
Activity.java
@NonNull
public LayoutInflater getLayoutInflater() {
    return getWindow().getLayoutInflater();
}

public Window getWindow() {
    return mWindow;
}
```

我们知道这里返回的是 PhoneWindow, 那么 PhoneWindow 是在什么时候创建的?  PhoneWindow 是在 `Activity#attch()` 方法里面创建的

```java
Activity.java
final void attach(...) {
    mWindow = new PhoneWindow(this, window, activityConfigCallback);
}
```

那么 `Activity#attch()` 方法什么时候被调用? 答案是在 `ActivityThread#performLaunchActivity`

```java
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    // 通过反射实例化 activity 对象
    java.lang.ClassLoader cl = appContext.getClassLoader();
    activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
    
    ...
    // attach
    activity.attach(appContext, this, ...);
    ...
}
```



```java
ContextImpl.java

@Override
public Object getSystemService(String name) {
    if (vmIncorrectContextUseEnabled()) {
        // Check incorrect Context usage.
        ...
    }
    return SystemServiceRegistry.getSystemService(this, name);
}
```



```java
@SystemApi
public final class SystemServiceRegistry {
    // 1. 
    private static final Map<String, ServiceFetcher<?>> SYSTEM_SERVICE_FETCHERS =
        new ArrayMap<String, ServiceFetcher<?>>();
    
    static {
        //CHECKSTYLE:OFF IndentationCheck
        
        // 2. 静态初始化代码块, 注册LAYOUT_INFLATER_SERVICE服务
        // 关注点：CachedServiceFetcher
    	// 关注点：PhoneLayoutInflater
        registerService(Context.LAYOUT_INFLATER_SERVICE, LayoutInflater.class,
                new CachedServiceFetcher<LayoutInflater>() {
            @Override
            public LayoutInflater createService(ContextImpl ctx) {
                // 注意：getOuterContext()，参数使用的是 ContextImpl 的代理对象，一般是 Activity
                return new PhoneLayoutInflater(ctx.getOuterContext());
            }});
    }
    
    // 注册服务
    private static <T> void registerService(@NonNull String serviceName,
            @NonNull Class<T> serviceClass, @NonNull ServiceFetcher<T> serviceFetcher) {
        SYSTEM_SERVICE_NAMES.put(serviceClass, serviceName);
        SYSTEM_SERVICE_FETCHERS.put(serviceName, serviceFetcher); // 
        SYSTEM_SERVICE_CLASS_NAMES.put(serviceName, serviceClass.getSimpleName());
    }
    
     /**
     * 根据 name 获取服务对象
     * @hide
     */
    public static Object getSystemService(ContextImpl ctx, String name) {
        if (name == null) {
            return null;
        }
        
        // 4. 
        final ServiceFetcher<?> fetcher = SYSTEM_SERVICE_FETCHERS.get(name);
        if (fetcher == null) {
            if (sEnableServiceNotFoundWtf) {
                Slog.wtf(TAG, "Unknown manager requested: " + name);
            }
            return null;
        }

        // 5. 
        final Object ret = fetcher.getService(ctx);
        if (sEnableServiceNotFoundWtf && ret == null) {
            // Some services do return null in certain situations, so don't do WTF for them.
            switch (name) {
                case Context.CONTENT_CAPTURE_MANAGER_SERVICE:
                case Context.APP_PREDICTION_SERVICE:
                case Context.INCREMENTAL_SERVICE:
                case Context.ETHERNET_SERVICE:
                    return null;
            }
            Slog.wtf(TAG, "Manager wrapper not available: " + name);
            return null;
        }
        return ret;
    }
}
    
```

```java
// 服务捕获器通用接口
static abstract interface ServiceFetcher<T> {
    T getService(ContextImpl ctx);
}
```

```java
/**
* CachedServiceFetcher : 优先取缓存
*/
static abstract class CachedServiceFetcher<T> implements ServiceFetcher<T> {
    private final int mCacheIndex;

    CachedServiceFetcher() {
        // Note this class must be instantiated only by the static initializer of the
        // outer class (SystemServiceRegistry), which already does the synchronization,
        // so bare access to sServiceCacheSize is okay here.
        mCacheIndex = sServiceCacheSize++;
    }

    @Override
    @SuppressWarnings("unchecked")
    public final T getService(ContextImpl ctx) { // 5.1 ServiceFetcher.getService 实现
        final Object[] cache = ctx.mServiceCache;
        final int[] gates = ctx.mServiceInitializationStateArray;
        boolean interrupted = false;

        T ret = null;

        for (;;) {
            boolean doInitialize = false;
            synchronized (cache) {
                // 优先去取缓存
                T service = (T) cache[mCacheIndex];
                if (service != null) {
                    ret = service;
                    break; // exit the for (;;)
                }

                // 没有缓存
                // 如果 gate 是 STATE_READY 状态, 意味着我们已经初始化过一次了, 但是被其他人清除了
                // Similarly, if the previous attempt returned null, we'll retry again.
                if (gates[mCacheIndex] == ContextImpl.STATE_READY
                        || gates[mCacheIndex] == ContextImpl.STATE_NOT_FOUND) {
                    gates[mCacheIndex] = ContextImpl.STATE_UNINITIALIZED;
                }

                // It's possible for multiple threads to get here at the same time, so
                // use the "gate" to make sure only the first thread will call createService().

                // At this point, the gate must be either UNINITIALIZED or INITIALIZING.
                if (gates[mCacheIndex] == ContextImpl.STATE_UNINITIALIZED) {
                    doInitialize = true;
                    gates[mCacheIndex] = ContextImpl.STATE_INITIALIZING;
                }
            }

            if (doInitialize) {
                // Only the first thread gets here.

                T service = null;
                // 初始化时, 默认为 STATE_NOT_FOUND 状态
                @ServiceInitializationState int newState = ContextImpl.STATE_NOT_FOUND;
                try {
                    // This thread is the first one to get here. Instantiate the service
                    // *without* the cache lock held.
                    service = createService(ctx);
                    newState = ContextImpl.STATE_READY;

                } catch (ServiceNotFoundException e) {
                    onServiceNotFound(e);

                } finally {
                    synchronized (cache) {
                        cache[mCacheIndex] = service;
                        gates[mCacheIndex] = newState;
                        cache.notifyAll();
                    }
                }
                ret = service;
                break; // exit the for (;;)
            }
            // The other threads will wait for the first thread to call notifyAll(),
            // and go back to the top and retry.
            synchronized (cache) {
                // Repeat until the state becomes STATE_READY or STATE_NOT_FOUND.
                // We can't respond to interrupts here; just like we can't in the "doInitialize"
                // path, so we remember the interrupt state here and re-interrupt later.
                while (gates[mCacheIndex] < ContextImpl.STATE_READY) {
                    try {
                        // Clear the interrupt state.
                        interrupted |= Thread.interrupted();
                        cache.wait();
                    } catch (InterruptedException e) {
                        // This shouldn't normally happen, but if someone interrupts the
                        // thread, it will.
                        Slog.w(TAG, "getService() interrupted");
                        interrupted = true;
                    }
                }
            }
        }
        if (interrupted) {
            Thread.currentThread().interrupt();
        }
        return ret;
    }

    public abstract T createService(ContextImpl ctx) throws ServiceNotFoundException;
}
```

可以看到，ContextImpl 内部通过 **SystemServiceRegistry** 来获取服务对象，逻辑并不复杂：

1、静态代码块注册了 **name - ServiceFetcher** 的映射

2、根据 name 获得 ServiceFetcher

3、ServiceFetcher 创建对象

ServiceFetcher 的子类有三种类型，它们的`getSystemService()`都是线程安全的，主要差别体现在 **单例范围**，具体如下：

| ServiceFetcher子类                     | 单例范围      | 描述                             | 举例                                      |
| -------------------------------------- | ------------- | -------------------------------- | ----------------------------------------- |
| CachedServiceFetcher                   | ContextImpl域 | /                                | LayoutInflater、LocationManager等（最多） |
| StaticServiceFetcher                   | 进程域        | /                                | InputManager、JobScheduler等              |
| StaticApplicationContextServiceFetcher | 进程域        | 使用 ApplicationContext 创建服务 | ConnectivityManager                       |

对于 LayoutInflater 来说，服务获取器是 **CachedServiceFetcher** 的子类，最终获得的服务对象为 **PhoneLayoutInflater**。









