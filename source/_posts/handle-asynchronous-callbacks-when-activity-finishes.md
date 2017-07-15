title: 如何在Activity/Fragment结束时处理异步回调？
date: 2017-03-05 21:05:45
tags: [Android, Asynchronous, Callback, Activity, Fragment, Finish]
categories: Android
---

# 头疼的IllegalArgumentException

在Android开发的过程中，涉及到与UI相关的操作只能在主线程执行，否则就会抛出以下异常：

```
android.view.ViewRoot$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views.
```

当然这属于基本常识，也不是本文讨论的重点，但后续的所有讨论都围绕这一基本常识进行。在开发Android应用时，如果所有的代码都在主线程执行，很容易就会出现ANR，并且Android在4.0以后已经禁止在主线程中执行网络请求，因此或多或少地需要与多线程打交道。无论是使用当前热火朝天的`OkHttp`(`Retrofit`)，还是使用过时的`Volley`或者`Android-Async-Http`，它们都支持异步请求。这些异步请求的请求流程一般如下：

	主线程发起请求
	->网络框架开启工作线程进行网络请求
	->工作线程拿到请求结果
	->将请求结果通过Handler返回主线程
	->主线程更新UI，完成一次网络请求

这个流程看似正常，实则暗含危机。下面的崩溃就是其中一个例子。

```
java.lang.IllegalArgumentException: View=com.android.internal.policy.impl.PhoneWindow$DecorView{24e9c19a V.E..... R......D 0,0-1026,348} not attached to window manager
at android.view.WindowManagerGlobal.findViewLocked(WindowManagerGlobal.java:403)
at android.view.WindowManagerGlobal.removeView(WindowManagerGlobal.java:322)
at android.view.WindowManagerImpl.removeViewImmediate(WindowManagerImpl.java:84)
at android.app.Dialog.dismissDialog(Dialog.java:368)
at android.app.Dialog.dismiss(Dialog.java:351)
at com.kaola.spring.ui.kaola.AvatarNicknameSetActivity.e(Unknown Source)
at com.kaola.spring.ui.kaola.AvatarNicknameSetActivity.c(Unknown Source)
at com.kaola.spring.ui.kaola.d.a(Unknown Source)
at com.kaola.common.c.d$b.handleMessage(Unknown Source)
at android.os.Handler.dispatchMessage(Handler.java:102)
at android.os.Looper.loop(Looper.java:135)
at android.app.ActivityThread.main(ActivityThread.java:5539)
at java.lang.reflect.Method.invoke(Native Method)
at java.lang.reflect.Method.invoke(Method.java:372)
at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:960)
at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:755)
```

这个崩溃是怎么发生的呢？原因很简单，工作线程执行网络请求，如果这个请求执行的时间过久(由于网络延迟等原因)，Activity或者Fragment已经不存在了(被销毁了)，而线程并不知道这件事，这时候请求数据结果回来以后，将数据通过Handler抛给了主线程，在异步回调里一般都会执行数据的更新或者进度条的更新等操作，但页面已经不存在了，所以就gg了。。。

# 目前的解决方案

有人会问，框架设计者怎么会没有考虑到这个问题？实际上他们确实考虑了这个问题。目前来说有两种解决方案：

* 在Activity结束时取消请求
* 在异步回调的时候，通过`Activity.isFinishing()`方法判断Activity是否已经被销毁

这两种方案本质是一样的，就是判断Activity是否被销毁，如果销毁，则要么取消回调，要么不执行回调。

## 在Activity结束时取消请求

以`Volley`为例，在`RequestQueue`这个类中，提供了通过`tag`来取消与之相关的网络请求。

```
/**
 * Cancels all requests in this queue with the given tag. Tag must be non-null
 * and equality is by identity.
 */
public void cancelAll(final Object tag) {
    if (tag == null) {
        throw new IllegalArgumentException("Cannot cancelAll with a null tag");
    }
    cancelAll(new RequestFilter() {
        @Override
        public boolean apply(Request<?> request) {
            return request.getTag() == tag;
        }
    });
}
```

这个`tag`是在主线程调起网络请求时，通过`Request.setTag(Object)`传进去的。`tag`可以是任意类型，通常来说可以使用下面两种类型：

* Context
* PATH

### Context

将`Context`作为tag带入请求中，当持有`Context`的对象销毁时，可以通知该请求线程取消。一个典型的使用场景就是在Activity的`onDestroy()`方法调用`Request.cancelAll(this)`来取消与该`Context`相关的所有请求。就`Volley`来说，它会在请求发出之前以及数据回来之后这两个时间段判断请求是否被cancel。

但是作为Android开发者，应该知道，持有Context引用的线程是危险的，如果线程发生死锁，Context引用会被线程一直持有，导致该Context得不到释放，容易引起内存泄漏。如果该Context的实例是一个Activity，那么这个结果是灾难性的。

有没有解决办法？有。线程通过弱引用持有Context。当系统内存不足需要GC时，会优先回收持有弱引用的对象。然而这样做还是存在隐患，有内存泄漏的风险。

### PATH

既然持有Context的问题比较严重，那么我们可以根据请求路径来唯一识别一个请求。发起请求的对象需要维持一个列表，记录当前发出的请求路径，在请求回来时再从该列表通过路径来删除该请求。在Activity结束但请求未发出或者未返回时，再将于这个Activity绑定的列表中的所有请求取消。

看起来方案不错，但执行起来如何呢？每个Activity都需要维持一个当前页面发出的请求列表，在Activity结束时再取消列表中的请求。面对一个应用几十上百个Activity，这样的实现无疑是蛋疼的。

有没有解决办法？有。通过良好的设计，可以避免这个问题。可以使用一个单例的路径管理类来管理所有的请求。所有发出的请求需要在这个管理类里注册，请求可以与当前发出的页面的类名(Class)进行绑定，从而在页面销毁时，注销所有与该页面关联的请求。

## Activity.isFinishing()

在异步请求的流程中，我们注意到最后两步：

	->将请求结果通过Handler返回主线程
	->主线程更新UI，完成一次网络请求

在这最后两步执行的过程中，我们可以加入判断，如果页面被销毁了，那么直接返回，不通知主线程更新UI了，这样就可以完美解决问题了。

类似的一个例子如下：

```
mMessageManager.getBoxList(new BaseManager.UIDataCallBack<JSONObject>() {
    @Override
    public void onSuccess(JSONObject object) {
        if(isFinishing()){
            return;
        }
        do what you want...
    }

    @Override
    public void onFail(int code, String msg) {
        if(isFinishing()){
            return;
        }
        do what you want...
    }
});
```

尽管解决了问题，但是麻烦又来了，如果只有一个人开发还好，需要时刻记住每个网络回调执行的时候，都需要提前判断`Activity.isFinishing()`。然而一个App往往有多个开发者一起协作完成，如果有一个开发者没有按照规定判断，那么这个App就有可能存在上述隐患，并且，在原有的网络基础框架上修改这么多的网络回调是不太现实的。

有没有更好的解决方案？请看下文。

# 基于Lifeful接口的异步回调框架

## Lifeful接口设计

我们定义Lifeful，一个不依赖于Context、也不依赖于PATH的接口。

```
/**
 * Created by xingli on 9/21/16.
 *
 * 判断生命周期是否已经结束的一个接口。
 */
public interface Lifeful {
    /**
     * 判断某一个组件生命周期是否已经走到最后。一般用于异步回调时判断Activity或Fragment生命周期是否已经结束。
     *
     * @return
     */
    boolean isAlive();
}
```
实际上，我们只需要让具有生命周期的类(一般是Activity或Fragment)实现这个接口，然后再通过这个接口来判断这个实现类是否还存在，就可以与Context解耦了。

接下来定义一个接口生成器，通过弱引用包装Lifeful接口的实现类，并返回所需要的相关信息。

```
/**
 * Created by xingli on 9/22/16.
 *
 * 生命周期具体对象生成器。
 */
public interface LifefulGenerator<Callback> {

    /**
     * @return 返回回调接口。
     */
    Callback getCallback();

    /**
     * 获取与生命周期绑定的弱引用，一般为Context，使用一层WeakReference包装。
     *
     * @return 返回与生命周期绑定的弱引用。
     */
    WeakReference<Lifeful> getLifefulWeakReference();

    /**
     * 传入的引用是否为Null。
     *
     * @return true if {@link Lifeful} is null.
     */
    boolean isLifefulNull();
}
```

提供一个该接口的默认实现：

```
/**
 * Created by xingli on 9/22/16.
 *
 * 默认生命周期管理包装生成器。
 */

public class DefaultLifefulGenerator<Callback> implements LifefulGenerator<Callback> {

    private WeakReference<Lifeful> mLifefulWeakReference;
    private boolean mLifefulIsNull;
    private Callback mCallback;

    public DefaultLifefulGenerator(Callback callback, Lifeful lifeful) {
        mCallback = callback;
        mLifefulWeakReference = new WeakReference<>(lifeful);
        mLifefulIsNull = lifeful == null;
    }

    @Override
    public Callback getCallback() {
        return mCallback;
    }

    public WeakReference<Lifeful> getLifefulWeakReference() {
        return mLifefulWeakReference;
    }

    @Override
    public boolean isLifefulNull() {
        return mLifefulIsNull;
    }
}
```

接着通过一个静态方法判断是否对象的生命周期：

```
/**
 * Created by xingli on 9/22/16.
 *
 * 生命周期相关帮助类。
 */

public class LifefulUtils {
    private static final String TAG = LifefulUtils.class.getSimpleName();

    public static boolean shouldGoHome(WeakReference<Lifeful> lifefulWeakReference, boolean objectIsNull) {
        if (lifefulWeakReference == null) {
            Log.e(TAG, "Go home, lifefulWeakReference == null");
            return true;
        }
        Lifeful lifeful = lifefulWeakReference.get();
        /**
         * 如果传入的Lifeful不为null,但弱引用为null,则这个对象被回收了。
         */
        if (null == lifeful && !objectIsNull) {
            Log.e(TAG, "Go home, null == lifeful && !objectIsNull");
            return true;
        }
        /**
         * 对象的生命周期结束
         */
        if (null != lifeful && !lifeful.isAlive()) {
            Log.e(TAG, "Go home, null != lifeful && !lifeful.isAlive()");
            return true;
        }
        return false;
    }

    public static <T> boolean shouldGoHome(LifefulGenerator<T> lifefulGenerator) {
        if (null == lifefulGenerator) {
            Log.e(TAG, "Go home, null == lifefulGenerator");
            return true;
        } if (null == lifefulGenerator.getCallback()) {
            Log.e(TAG, "Go home, null == lifefulGenerator.getCallback()");
            return true;
        }
        return shouldGoHome(lifefulGenerator.getLifefulWeakReference(), lifefulGenerator.isLifefulNull());
    }
}
```

## 具有生命周期的Runnable

具体到跟线程打交道的异步类，只有`Runnable`(`Thread`也是其子类)，因此只需要处理`Runnable`就可以了。我们可以通过`Wrapper`包装器模式，在处理真正的Runnable类之前，先通过Lifeful接口判断对象是否还存在，如果不存在则直接返回。对于`Runnable`：

```
/**
 * Created by xingli on 9/21/16.
 *
 * 与周期相关的异步线程回调类。
 */
public class LifefulRunnable implements Runnable {

    private LifefulGenerator<Runnable> mLifefulGenerator;

    public LifefulRunnable(Runnable runnable, Lifeful lifeful) {
        mLifefulGenerator = new DefaultLifefulGenerator<>(runnable, lifeful);
    }

    @Override
    public void run() {
        if (LifefulUtils.shouldGoHome(mLifefulGenerator)) {
            return;
        }
        mLifefulGenerator.getCallback().run();
    }
}
```

## Lifeful的实现类

最后说一下Lifeful类的实现类，主要包括Activity和Fragment，

```
public class BaseActivity extends Activity implements Lifeful {
	
    @Override
    public boolean isAlive() {
        return activityIsAlive();
    }

	public boolean activityIsAlive() {
		if (currentActivity == null) return false;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
            return !(currentActivity.isDestroyed() || currentActivity.isFinishing());
        } else {
            return !currentActivity.isFinishing();
        }
	}
}
```

```
public class BaseFragment extends Fragment implements Lifeful {
	
    @Override
    public boolean isAlive() {
        return activityIsAlive();
    }
    
	public boolean activityIsAlive() {
		Activity currentActivity = getActivity();
		if (currentActivity == null) return false;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
            return !(currentActivity.isDestroyed() || currentActivity.isFinishing());
        } else {
            return !currentActivity.isFinishing();
        }
	}
}
```

除了这两个类以外，别的类如果有生命周期，或者包含生命周期的引用，也可以实现Lifeful接口(如View，可以通过`onAttachedToWindow()`和`onDetachedToWindow()`)。

## 包含生命周期的异步调用

对于需要用到异步的地方，调用也很方便。

```
// ThreadCore是一个用于线程调度的ThreadPoolExecutor封装类，也用于主线程和工作线程之间的切换
ThreadCore.getInstance().postOnMainLooper(new LifefulRunnable(new Runnable() {
    @Override
    public void run() {
        // 实现真正的逻辑。
    }
}, this));

```

# 总结

本文主要针对Android中具有生命周期的对象在已经被销毁时对应的异步线程的处理方式进行解耦的过程。通过定义Lifeful接口，实现了不依赖于Context或其他容易造成内存泄漏的对象，却又能与对象的生命周期进行绑定的方法。

参考链接：

1. <http://www.jianshu.com/p/9376ddde4108>
2. <http://note.tyz.ren/blog/post/zerozhiqin/%E5%A6%82%E4%BD%95%E5%9C%A8%E5%9B%9E%E8%B0%83%E6%97%B6%E5%88%A4%E6%96%ADActivity%EF%BC%8CFragment%EF%BC%8CImageView%E7%AD%89%E7%AD%89%E6%98%AF%E5%90%A6%E5%B7%B2%E7%BB%8F%E8%A2%AB%E5%85%B3%E9%97%AD>