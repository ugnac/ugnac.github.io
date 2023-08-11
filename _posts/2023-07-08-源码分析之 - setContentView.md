---
layout: post
title: 源码分析之 - setContentView
date: 2023-07-08 14:44:45 +0800
categories: [Android源码, setContentView]
tags: [Android源码, setContentView]
author: 
---

## 布局框架图

![setContentView.drawio](../.image/setContentView.drawio.svg)

![activity_layout_inspecto](../.image/activity_layout_inspector.png)

## 继承自 Activity

`setContentView(R.layout.main_activity)` 流程

### 0. 伪代码概览

1.  创建 DecorView
2.  解析系统内置的模版布局(R.layout.screen_simple), 将模版布局放入到 DecorView, 并解析返回 contentParent
3.  将我们设置的布局 渲染到 mContentParent 上

```java
// 伪代码流程
public void setContentView(int layoutResID) {
    installDecor() {
        // 1. 创建 DecorView
        mDecor = generateDecor(-1) -> new DecorView(); 	
        
        // 2. 解析系统内置的模版布局(R.layout.screen_simple), 将模版布局放入到 DecorView, 并解析返回 contentParent
        mContentParent = generateLayout(mDecor) {
            layoutResource = R.layout.screen_simple // @android:id/content -> mContentParent
            mDecor.onResourcesLoaded(mLayoutInflater, layoutResource) {
                final View root = inflater.inflate(layoutResource, null); // 加载解析布局资源
                
                // 将 layoutResource 添加到 DecorView 上
                addView(root, 0, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
            }
            ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
            return contentParent; // @android:id/content -> mContentParent
        }
    };

    // 3. 将我们设置的布局 渲染到 mContentParent 上
    mLayoutInflater.inflate(layoutResID, mContentParent);
}
```

### 1. Activity.setContentView

```java
Activity.java
public void setContentView(View view) {
    getWindow().setContentView(view);
    initWindowDecorActionBar();
}
```

### 2. PhoneWindow.getWindow() Window 是什么时候初始化的?

Window 的初始化其实是在 ActivityThread `performLaunchActivity` 时, 通过 Activity 的 `attach` 方法里面初始化的

```java
ActivityThread.performLaunchActivity
    -> activity = mInstrumentation.newActivity() // 反射创建activity
    -> activity.attach
    	-> new PhoneWindow() // 创建PhoneWindow
    -> mInstrumentation.callActivityOnCreate()
    
public final class ActivityThread extends ClientTransactionHandler
        implements ActivityThreadInternal {
    // 1. performLaunchActivity
    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        省略若干代码...
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = appContext.getClassLoader();
            activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent); // 通过放射创建 activity
        }
        省略若干代码...
        if (activity != null) {
            // 2. activity.attach
            activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.activityConfigCallback,
                        r.assistToken, r.shareableActivityToken);
        }
    }
}

@UiContext
public class Activity extends ContextThemeWrapper
        implements LayoutInflater.Factory2,
        Window.Callback, KeyEvent.Callback,
        OnCreateContextMenuListener, ComponentCallbacks2,
        Window.OnWindowDismissedCallback,
        ContentCaptureManager.ContentCaptureClient {
            
    @UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.R, trackingBug = 170729553)
    final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken,
            IBinder shareableActivityToken) {
        attachBaseContext(context);

        mFragments.attachHost(null /*parent*/);

        // 3. new PhoneWindow()
        mWindow = new PhoneWindow(this, window, activityConfigCallback); // 创建 PhoneWindow   
        
    	省略若干代码...
            
        mUiThread = Thread.currentThread(); // mUiThread

        mMainThread = aThread; // ActivityThread 对象
        mInstrumentation = instr;
        mToken = token;
        mAssistToken = assistToken;
        mShareableActivityToken = shareableActivityToken;
        mIdent = ident;
        mApplication = application;
        mIntent = intent;
        mReferrer = referrer;
        mComponent = intent.getComponent();
        mActivityInfo = info;
        mTitle = title;
        mParent = parent;
        mEmbeddedID = id;
        
        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0); // setWindowManager
        if (mParent != null) {
            mWindow.setContainer(mParent.getWindow());
        }
        mWindowManager = mWindow.getWindowManager(); // getWindowManager
        mCurrentConfig = config;
        
        省略若干代码...
    }
}
```

### 3. PhoneWindow.setContentView

```java
public class PhoneWindow extends Window implements MenuBuilder.Callback {
    public static final int ID_ANDROID_CONTENT = com.android.internal.R.id.content;
    
    @Override
    public void setContentView(int layoutResID) {
        if (mContentParent == null) {
            installDecor(); //1. 创建 DecorView, 初始化 mContentParent
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            /**
            * com.android.internal.R.id.content -> mContentParent
            * public View inflate(@LayoutRes int resource, @Nullable ViewGroup root)
            * 也就是说我们的布局是放在 R.id.content -> mContentParent 里面
            */
            mLayoutInflater.inflate(layoutResID, mContentParent); // R.layout.activity_main 渲染到 mContentParent
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
    
    private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
            mDecor = generateDecor(-1); // 2. 创建 DecorView
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
                mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
            }
        } else {
            mDecor.setWindow(this);
        }
        
        if (mContentParent == null) {
            mContentParent = generateLayout(mDecor); // 3. 创建 mContentParent
        }
    }
    
    protected DecorView generateDecor(int featureId) {
        ...
        return new DecorView(context, featureId, this, getAttributes()); // new DecorView() 这是一个 FrameLayout
    }
    
    @Override
    public final @NonNull View getDecorView() {
        if (mDecor == null || mForceDecorInstall) {
            installDecor();
        }
        return mDecor;
    }
    
    protected ViewGroup generateLayout(DecorView decor) {
        // Apply data from current theme.
        略...
            
        // Inflate the window decor.
        int layoutResource;
        略...
        layoutResource = R.layout.screen_simple; // 最简单的模版布局
        
        mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
        
        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT); // com.android.internal.R.id.content
         略...
        return contentParent; // com.android.internal.R.id.content
    }
}

public class DecorView extends FrameLayout implements RootViewSurfaceTaker, WindowCallbacks {
    void onResourcesLoaded(LayoutInflater inflater, int layoutResource) {
         略...
        final View root = inflater.inflate(layoutResource, null); // 加载解析布局资源
        if (mDecorCaptionView != null) {
            略...
        } else {
            // 将 R.layout.screen_simple 布局放在第0个, 也就是最下层
            // Put it below the color views.
            addView(root, 0, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
        }
        mContentRoot = (ViewGroup) root; // 
        initializeElevation();
    }
}
```

### 4. 总结:

1.  创建 DecorView
2.  解析系统内置的模版布局(R.layout.screen_simple), 将模版布局放入到 DecorView, 并解析返回 contentParent
3.  将我们设置的布局 渲染到 mContentParent 上

```java
继承Activity的流程
// 伪代码流程
PhoneWindow.setContentView --- 主要目的 创建 DecorView 拿到 Content
    -> installDecor() // 创建DecorView拿到Content
    	-> mDecor = generateDecor(-1) -> new DecorView()	
    	-> mContentParent = generateLayout(mDecor) // @android:id/content -> mContentParent
    		-> layoutResource = R.layout.screen_simple // @android:id/content -> mContentParent
            -> mDecor.onResourcesLoaded(mLayoutInflater, layoutResource); // 将 layoutResource 添加到 DecorView 上
			-> ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
    -> mLayoutInflater.inflate(layoutResID, mContentParent); // R.layout.activity_main 渲染到 mContentParent
    
PhoneWindow --> 分为3类, 哪些地方会创建
    1. Activity
    2. Dialog
    3. PopupWindow
    4. Toast
```



## 继承自 AppCompactActivity

`setContentView(R.layout.main_activity)` 流程

### 0. 伪代码概览

1.  创建 mSubDecor R.layout.abc_screen_simple
2.  找到 contentParent 布局
3.  将我们设置的布局 渲染到 mContentParent 上

```java
// 伪代码流程
public void setContentView(int layoutResID) {
    ensureSubDecor() {
        // 1. 创建 mSubDecor R.layout.abc_screen_simple
        mSubDecor = createSubDecor() {
            ensureWindow(); // 从 Activity 拿 PhoneWindow
            mWindow.getDecorView() { // PhoneWindow.getDecorView
                installDecor() {
                    // 接下来的流程和继承自 Activity 的一样
                    
                    // 1.1 创建 DecorView
                    mDecor = generateDecor(-1) -> new DecorView(); 	

                    // 1.2. 解析系统内置的模版布局(R.layout.screen_simple), 将模版布局放入到 DecorView, 并解析返回 contentParent
                    mContentParent = generateLayout(mDecor) {
                        layoutResource = R.layout.screen_simple // @android:id/content -> mContentParent
                        mDecor.onResourcesLoaded(mLayoutInflater, layoutResource) {
                            final View root = inflater.inflate(layoutResource, null); // 加载解析布局资源

                            // 将 layoutResource 添加到 DecorView 上
                            addView(root, 0, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
                        }
                        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
                        return contentParent;
                    }
                }
            }
            
            // 1.3 AppCompatActivity 实际的模版布局 R.layout.abc_screen_simple
            ViewGroup subDecor = (ViewGroup) inflater.inflate(R.layout.abc_screen_simple, null);
            
            final ContentFrameLayout contentView = subDecor.findViewById(R.id.action_bar_activity_content);
            
            // 1.4 替换View ID
            final ViewGroup windowContentView = (ViewGroup) mWindow.findViewById(android.R.id.content); //  
            if (windowContentView != null) {
                // 如果原来的 content 里面有 child 节点, 则迁移到新的 contentView 上
                while (windowContentView.getChildCount() > 0) {
                    final View child = windowContentView.getChildAt(0);
                    windowContentView.removeViewAt(0);
                    contentView.addView(child);
           		}
                
                // Change our content FrameLayout to use the android.R.id.content id.
                // Useful for fragments.
                windowContentView.setId(View.NO_ID); // 将原始的content id 置为 NO_ID
			    contentView.setId(android.R.id.content); // R.id.action_bar_activity_content -> android.R.id.content
            }
            
            /**
            * 1.5 很重要的一步, 将 subDecor 添加到 mContentParent 里面
            * 相当于subDecor 是添加到 R.layout.screen_simple 的 content 里面的
            */
            // Now set the Window's content view with the decor
            mWindow.setContentView(subDecor) {
                mContentParent.addView(subDecor, params);
            }
            
			return subDecor; // 1.6 返回 subDecor -> R.layout.abc_screen_simple
        }
        
        // 2. 找到 contentParent 布局
        ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
        
        // 3. 将我们设置的布局 渲染到 mContentParent 上
        LayoutInflater.from(mContext).inflate(resId, contentParent);
    }
}
```

### 1. AppCompatActivity.setContentView

```java
public class AppCompatActivity extends FragmentActivity implements AppCompatCallback,
        TaskStackBuilder.SupportParentable, ActionBarDrawerToggle.DelegateProvider {
            
    @Override
    public void setContentView(@LayoutRes int layoutResID) {
        initViewTreeOwners();
        getDelegate().setContentView(layoutResID);
    }
            
    @NonNull
    public AppCompatDelegate getDelegate() {
        if (mDelegate == null) {
            mDelegate = AppCompatDelegate.create(this, this);
        }
        return mDelegate;
    }
            
}
```

### 2. AppCompatDelegateImpl.setContentView

1.  创建 subDecor R.layout.abc_screen_simple
2.  找到 contentParent 布局
3.  将我们的布局渲染到 contentParent 里面

```java
@RestrictTo(LIBRARY)
class AppCompatDelegateImpl extends AppCompatDelegate
        implements MenuBuilder.Callback, LayoutInflater.Factory2 {
    
    @Override
    public void setContentView(int resId) {
        ensureSubDecor(); // 1. 创建 subDecor R.layout.abc_screen_simple
        ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content); // 2. 找到content布局
        contentParent.removeAllViews();
        LayoutInflater.from(mContext).inflate(resId, contentParent); // 3. 将我们的布局渲染到 content 里面
        mAppCompatWindowCallback.bypassOnContentChanged(mWindow.getCallback());
    }
    
    private void ensureSubDecor() {
        if (!mSubDecorInstalled) {
            mSubDecor = createSubDecor();
            ...
            mSubDecorInstalled = true;
        }
    }
    
    private ViewGroup createSubDecor() {
        ...
        // Now let's make sure that the Window has installed its decor by retrieving it
        ensureWindow(); // 初始化 AppCompatDelegateImpl 的 mWindow, 确保 mWindow != null
        mWindow.getDecorView(); // 如果 mDecorView == null, 就初始化 mDecorView

        final LayoutInflater inflater = LayoutInflater.from(mContext);
        ViewGroup subDecor = null;
        
        ...
        // 加载内置布局文件到 subDecor
        subDecor = (ViewGroup) inflater.inflate(R.layout.abc_screen_simple, null);
        
        ...
            
        // 拿到 AppCompat 的 contentView
        final ContentFrameLayout contentView = (ContentFrameLayout) subDecor.findViewById(
                R.id.action_bar_activity_content);
        
        final ViewGroup windowContentView = (ViewGroup) mWindow.findViewById(android.R.id.content);
        if (windowContentView != null) {
            // There might be Views already added to the Window's content view so we need to
            // migrate them to our content view
            while (windowContentView.getChildCount() > 0) {
                final View child = windowContentView.getChildAt(0);
                windowContentView.removeViewAt(0);
                contentView.addView(child);
            }

            // Change our content FrameLayout to use the android.R.id.content id.
            // Useful for fragments.
            windowContentView.setId(View.NO_ID);
            contentView.setId(android.R.id.content); // 
        }
        
        ...
        // Now set the Window's content view with the decor
        mWindow.setContentView(subDecor);
        
        ...
        return subDecor; // R.layout.abc_screen_simple
    }
}

public class PhoneWindow extends Window implements MenuBuilder.Callback {
    @Override
    public final @NonNull View getDecorView() {
        if (mDecor == null || mForceDecorInstall) {
            installDecor();
        }
        return mDecor;
    }
    
    private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
            mDecor = generateDecor(-1); // 2. 创建 DecorView
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
                mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
            }
        } else {
            mDecor.setWindow(this);
        }
        
        if (mContentParent == null) {
            mContentParent = generateLayout(mDecor); // 3. 创建 mContentParent
        }
    }
}
```

### 3. 总结:

1.  创建 subDecor R.layout.abc_screen_simple
2.  找到 contentParent 布局
3.  将我们的布局渲染到 contentParent 里面

```java
AppCompatDelegateImpl.java
AppCompatDelegate.setContentView(int resId) 
    -> ensureSubDecor()
    	-> mSubDecor = createSubDecor();
			-> ensureWindow(); // 从 Activity 拿 PhoneWindow
			-> mWindow.getDecorView(); // PhoneWindow.getDecorView
				-> installDecor() // mContentParent 接下来的流程和继承自Activity的一样
                    ...// 看Activity的流程
            -> final ContentFrameLayout contentView = subDecor.findViewById(R.id.action_bar_activity_content);
                    
            // R.layout.screen_simple 里面的 content -> 把 content 的 View 复制到 R.id.action_bar_activity_content
            -> final ViewGroup windowContentView = (ViewGroup) mWindow.findViewById(android.R.id.content); //  
			-> windowContentView.setId(View.NO_ID); // 将原始的content id 置为 NO_ID
			-> contentView.setId(android.R.id.content); // R.id.action_bar_activity_content -> android.R.id.content
    -> ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
	-> LayoutInflater.from(mContext).inflate(resId, contentParent);
```

