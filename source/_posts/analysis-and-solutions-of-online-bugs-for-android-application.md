title: Android应用线上崩溃Bug分析及解决方案
date: 2016-05-19 09:23:29
tags: [Android, Bug, Online, Exception]
categories: Android
---

# Android v2.5.2

## java.lang.OutOfMemoryError: Failed to allocate a 2560012 byte allocation with 2300136 free bytes and 2MB until OOM

```
java.lang.OutOfMemoryError: Failed to allowcate a 2560012 byte allocation with 2300136 free bytes and 2MB until OOM
	at dalvik.system.VMRuntime.newNonMovableArray(Native Method)
	at android.graphics.Bitmap.nativeCreate(Native Method)
	at android.graphics.Bitmap.createBitmap(Bitmap.java:847)
	at android.graphics.Bitmap.createBitmap(Bitmap.java:817)
	at android.graphics.Bitmap.createBitmap(Bitmap.java:784)
	at com.facebook.imagepipeline.memory.BitmapPool.i(Unknown Source)
	at com.facebook.imagepipeline.memory.BitmapPool.b(Unknown Source)
	at com.facebook.imagepipeline.memory.BasePool.a(Unknown Source)
	at com.facebook.imagepipeline.platform.ArtDecoder.a(Unknown Source)
	at com.facebook.imagepipeline.platform.ArtDecoder.a(Unknown Source)
	at com.facebook.imagepipeline.decoder.ImageDecoder.a(Unknown Source)
	at com.facebook.imagepipeline.decoder.ImageDecoder.a(Unknown Source)
	at com.facebook.imagepipeline.producers.DecodeProducer$ProgressiveDecoder.c(Unknown Source)
	at com.facebook.imagepipeline.producers.DecodeProducer$ProgressiveDecoder.a(Unknown Source)
	at com.facebook.imagepipeline.producers.DecodeProducer$ProgressiveDecoder$1.a(Unknown Source)
	at com.facebook.imagepipeline.producers.JobScheduler.e(Unknown Source)
	at com.facebook.imagepipeline.producers.JobScheduler.a(Unknown Source)
	at com.facebook.imagepipeline.producers.JobScheduler$1.run(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1112)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:587)
	at com.facebook.imagepipeline.core.PriorityThreadFactory$1.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:831)
```

系统版本特点：5.0以上  
机型分布特点：华为、小米居多  
原因分析：图片加载占用内存过多，短时间内无法释放，剩余的内存资源无法再加载更多的图片，导致OOM异常。  
可能的解决办法：

```
<application
	android:name=".ParaseApplication"
	android:allowBackup="true"
	android:icon="@mipmap/ic_launcher"
	android:label="@string/app_name"
	android:theme="@style/AppTheme"
	android:largeHeap="true"
>
```

## java.lang.RuntimeException: Unable to start activity ComponentInfo{com.kaola/com.kaola.spring.ui.kaola.InterestedCategoryActivity}: java.lang.NullPointerException: Attempt to read from field 'com.android.server.am.ActivityStackSupervisor com.android.server.am.ActivityRecord.mStackSupervisor' on a null object reference

```
java.lang.RuntimeException: Unable to start activity ComponentInfo{com.kaola/com.kaola.spring.ui.kaola.InterestedCategoryActivity}: java.lang.NullPointerException: Attempt to read from field 'com.android.server.am.ActivityStackSupervisor com.android.server.am.ActivityRecord.mStackSupervisor' on a null object reference
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2379)
	at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2441)
	at android.app.ActivityThread.access$800(ActivityThread.java:154)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1323)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:135)
	at android.app.ActivityThread.main(ActivityThread.java:5336)
	at java.lang.reflect.Method.invoke(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:372)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:904)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:699)
Caused by: java.lang.NullPointerException: Attempt to read from field 'com.android.server.am.ActivityStackSupervisor com.android.server.am.ActivityRecord.mStackSupervisor' on a null object reference
	at android.os.Parcel.readException(Parcel.java:1546)
	at android.os.Parcel.readException(Parcel.java:1493)
	at android.app.ActivityManagerProxy.convertToTranslucent(ActivityManagerNative.java:4417)
	at android.app.Activity.convertToTranslucent(Activity.java:5469)
	at android.app.EnterTransitionCoordinator.prepareEnter(EnterTransitionCoordinator.java:275)
	at android.app.EnterTransitionCoordinator.(EnterTransitionCoordinator.java:69)
	at android.app.ActivityTransitionState.enterReady(ActivityTransitionState.java:181)
	at android.app.Activity.performStart(Activity.java:6011)
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2342)
	... 10 more
java.lang.NullPointerException: Attempt to read from field 'com.android.server.am.ActivityStackSupervisor com.android.server.am.ActivityRecord.mStackSupervisor' on a null object reference
	at android.os.Parcel.readException(Parcel.java:1546)
	at android.os.Parcel.readException(Parcel.java:1493)
	at android.app.ActivityManagerProxy.convertToTranslucent(ActivityManagerNative.java:4417)
	at android.app.Activity.convertToTranslucent(Activity.java:5469)
	at android.app.EnterTransitionCoordinator.prepareEnter(EnterTransitionCoordinator.java:275)
	at android.app.EnterTransitionCoordinator.(EnterTransitionCoordinator.java:69)
	at android.app.ActivityTransitionState.enterReady(ActivityTransitionState.java:181)
	at android.app.Activity.performStart(Activity.java:6011)
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2342)
	at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2441)
	at android.app.ActivityThread.access$800(ActivityThread.java:154)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1323)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:135)
	at android.app.ActivityThread.main(ActivityThread.java:5336)
	at java.lang.reflect.Method.invoke(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:372)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:904)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:699)
```

系统版本特点：5.0以上  
机型分布特点：全部是nubia  
原因分析：特定机型修改了系统启动Activity的源码，导致的NPE。在`android.app.ActivityThread.performLaunchActivity#performLaunchActivity()`方法签名中并没有找到对应的Exception log。  
解决办法：暂时规避这些特定的机型。


## java.lang.OutOfMemoryError: Failed to allocate a 2560012 byte allocation with 2300136 free bytes and 2MB until OOM

```
java.lang.OutOfMemoryError: Failed to allocate a 2560012 byte allocation with 2300136 free bytes and 2MB until OOM
	at dalvik.system.VMRuntime.newNonMovableArray(Native Method)
	at android.graphics.Bitmap.nativeCreate(Native Method)
	at android.graphics.Bitmap.createBitmap(Bitmap.java:847)
	at android.graphics.Bitmap.createBitmap(Bitmap.java:817)
	at android.graphics.Bitmap.createBitmap(Bitmap.java:784)
	at com.facebook.imagepipeline.memory.BitmapPool.i(Unknown Source)
	at com.facebook.imagepipeline.memory.BitmapPool.b(Unknown Source)
	at com.facebook.imagepipeline.memory.BasePool.a(Unknown Source)
	at com.facebook.imagepipeline.platform.ArtDecoder.a(Unknown Source)
	at com.facebook.imagepipeline.platform.ArtDecoder.a(Unknown Source)
	at com.facebook.imagepipeline.decoder.ImageDecoder.a(Unknown Source)
	at com.facebook.imagepipeline.decoder.ImageDecoder.a(Unknown Source)
	at com.facebook.imagepipeline.producers.DecodeProducer$ProgressiveDecoder.c(Unknown Source)
	at com.facebook.imagepipeline.producers.DecodeProducer$ProgressiveDecoder.a(Unknown Source)
	at com.facebook.imagepipeline.producers.DecodeProducer$ProgressiveDecoder$1.a(Unknown Source)
	at com.facebook.imagepipeline.producers.JobScheduler.e(Unknown Source)
	at com.facebook.imagepipeline.producers.JobScheduler.a(Unknown Source)
	at com.facebook.imagepipeline.producers.JobScheduler$1.run(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1112)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:587)
	at com.facebook.imagepipeline.core.PriorityThreadFactory$1.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:831)
```

系统版本特点：4.3以上，5系统占大头  
机型分布特点：华为、小米  
原因分析：同1.

## java.lang.NoClassDefFoundError: com/facebook/imagepipeline/memory/NativeMemoryChunk

```
java.lang.NoClassDefFoundError: com/facebook/imagepipeline/memory/NativeMemoryChunk
	at com.facebook.imagepipeline.memory.NativeMemoryChunkPool.i(Unknown Source)
	at com.facebook.imagepipeline.memory.NativeMemoryChunkPool.b(Unknown Source)
	at com.facebook.imagepipeline.memory.BasePool.a(Unknown Source)
	at com.facebook.imagepipeline.memory.NativePooledByteBufferOutputStream.(Unknown Source)
	at com.facebook.imagepipeline.memory.NativePooledByteBufferFactory.a(Unknown Source)
	at com.facebook.imagepipeline.memory.NativePooledByteBufferFactory.b(Unknown Source)
	at com.facebook.imagepipeline.producers.LocalFetchProducer.a(Unknown Source)
	at com.facebook.imagepipeline.producers.LocalFetchProducer.b(Unknown Source)
	at com.facebook.imagepipeline.producers.LocalResourceFetchProducer.a(Unknown Source)
	at com.facebook.imagepipeline.producers.LocalFetchProducer$1.d(Unknown Source)
	at com.facebook.imagepipeline.producers.LocalFetchProducer$1.c(Unknown Source)
	at com.facebook.common.executors.StatefulRunnable.run(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1112)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:587)
	at java.lang.Thread.run(Thread.java:841)
```

系统版本特点：4.2.1-4.4.4  
机型分布特点：华为、三星N9008、小米3  
原因分析：`libmemchunk.so`加载失败，与`NELoginJni`有些类似  
解决办法：

1. 把`libmemchunk.so`放到armeabi-v7a目录下
2. <https://github.com/facebook/fresco/blob/master/fbcore/src/main/java/com/facebook/common/internal/proguard_annotations.pro>
3. 更新fresco到最新版本

Ref：  
<https://github.com/facebook/fresco/issues/410>

```
# Keep our interfaces so they can be used by other ProGuard rules.
# See http://sourceforge.net/p/proguard/bugs/466/
-keep,allowobfuscation @interface com.facebook.common.internal.DoNotStrip

# Do not strip any method/class that is annotated with @DoNotStrip
-keep @com.facebook.common.internal.DoNotStrip class *
-keepclassmembers class * {
    @com.facebook.common.internal.DoNotStrip *;
}
```

## java.lang.RuntimeException: Unable to start activity ComponentInfo{com.kaola/com.kaola.spring.ui.goodsdetail.SkuPopWindowActivity}: java.lang.NumberFormatException: Invalid double: "5,712"

```
java.lang.RuntimeException: Unable to start activity ComponentInfo{com.kaola/com.kaola.spring.ui.goodsdetail.SkuPopWindowActivity}: java.lang.NumberFormatException: Invalid double: "5,712"
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2444)
	at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2504)
	at android.app.ActivityThread.access$900(ActivityThread.java:165)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1368)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:150)
	at android.app.ActivityThread.main(ActivityThread.java:5546)
	at java.lang.reflect.Method.invoke(Native Method)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:792)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:682)
Caused by: java.lang.NumberFormatException: Invalid double: "5,712"
	at java.lang.StringToReal.invalidReal(StringToReal.java:63)
	at java.lang.StringToReal.initialParse(StringToReal.java:164)
	at java.lang.StringToReal.parseDouble(StringToReal.java:282)
	at java.lang.Double.parseDouble(Double.java:301)
	at com.kaola.common.utils.w.c(Unknown Source)
	at com.kaola.spring.ui.goodsdetail.SkuPopWindowActivity.f(Unknown Source)
	at com.kaola.spring.ui.goodsdetail.SkuPopWindowActivity.b(Unknown Source)
	at com.kaola.spring.ui.goodsdetail.SkuPopWindowActivity.d(Unknown Source)
	at com.kaola.spring.ui.goodsdetail.SkuPopWindowActivity.onCreate(Unknown Source)
	at android.app.Activity.performCreate(Activity.java:6367)
	at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1110)
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2397)
	... 9 more
java.lang.NumberFormatException: Invalid double: "5,712"
	at java.lang.StringToReal.invalidReal(StringToReal.java:63)
	at java.lang.StringToReal.initialParse(StringToReal.java:164)
	at java.lang.StringToReal.parseDouble(StringToReal.java:282)
	at java.lang.Double.parseDouble(Double.java:301)
	at com.kaola.common.utils.w.c(Unknown Source)
	at com.kaola.spring.ui.goodsdetail.SkuPopWindowActivity.f(Unknown Source)
	at com.kaola.spring.ui.goodsdetail.SkuPopWindowActivity.b(Unknown Source)
	at com.kaola.spring.ui.goodsdetail.SkuPopWindowActivity.d(Unknown Source)
	at com.kaola.spring.ui.goodsdetail.SkuPopWindowActivity.onCreate(Unknown Source)
	at android.app.Activity.performCreate(Activity.java:6367)
	at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1110)
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2397)
	at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2504)
	at android.app.ActivityThread.access$900(ActivityThread.java:165)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1368)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:150)
	at android.app.ActivityThread.main(ActivityThread.java:5546)
	at java.lang.reflect.Method.invoke(Native Method)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:792)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:682)
```

系统版本特点：4.3以上  
机型分布特点：华为、三星N9008、小米3  
原因分析：将String转换成Double的时候，不符合转换预设的格式，导致转换出错。这个转换跟区域有关  
解决办法：指定区域为`Locale.CHINA`。

## java.lang.IllegalArgumentException: pointerIndex out of range

```
java.lang.IllegalArgumentException: pointerIndex out of range
	at android.view.MotionEvent.nativeGetAxisValue(Native Method)
	at android.view.MotionEvent.getX(MotionEvent.java:1979)
	at android.support.v4.view.x.c(Unknown Source)
	at android.support.v4.view.w$b.c(Unknown Source)
	at android.support.v4.view.w.c(Unknown Source)
	at android.support.v4.view.ViewPager.onInterceptTouchEvent(Unknown Source)
	at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:1913)
	at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2348)
	at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2042)
	at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2348)
	at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2042)
	at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2348)
	at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2042)
	at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2348)
	at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2042)
	at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2348)
	at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2042)
	at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2348)
	at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2042)
	at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2348)
	at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2042)
	at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2348)
	at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2042)
	at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:2348)
	at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2042)
	at com.android.internal.policy.impl.PhoneWindow$DecorView.superDispatchTouchEvent(PhoneWindow.java:2164)
	at com.android.internal.policy.impl.PhoneWindow.superDispatchTouchEvent(PhoneWindow.java:1596)
	at android.app.Activity.dispatchTouchEvent(Activity.java:2527)
	at com.android.internal.policy.impl.PhoneWindow$DecorView.dispatchTouchEvent(PhoneWindow.java:2112)
	at android.view.View.dispatchPointerEvent(View.java:8015)
	at android.view.ViewRootImpl$ViewPostImeInputStage.processPointerEvent(ViewRootImpl.java:4632)
	at android.view.ViewRootImpl$ViewPostImeInputStage.onProcess(ViewRootImpl.java:4503)
	at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:4023)
	at android.view.ViewRootImpl$InputStage.onDeliverToNext(ViewRootImpl.java:4073)
	at android.view.ViewRootImpl$InputStage.forward(ViewRootImpl.java:4042)
	at android.view.ViewRootImpl$AsyncInputStage.forward(ViewRootImpl.java:4149)
	at android.view.ViewRootImpl$InputStage.apply(ViewRootImpl.java:4050)
	at android.view.ViewRootImpl$AsyncInputStage.apply(ViewRootImpl.java:4206)
	at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:4023)
	at android.view.ViewRootImpl$InputStage.onDeliverToNext(ViewRootImpl.java:4073)
	at android.view.ViewRootImpl$InputStage.forward(ViewRootImpl.java:4042)
	at android.view.ViewRootImpl$InputStage.apply(ViewRootImpl.java:4050)
	at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:4023)
	at android.view.ViewRootImpl.deliverInputEvent(ViewRootImpl.java:6331)
	at android.view.ViewRootImpl.doProcessInputEvents(ViewRootImpl.java:6293)
	at android.view.ViewRootImpl.enqueueInputEvent(ViewRootImpl.java:6247)
	at android.view.ViewRootImpl$WindowInputEventReceiver.onInputEvent(ViewRootImpl.java:6479)
	at android.view.InputEventReceiver.dispatchInputEvent(InputEventReceiver.java:185)
	at android.view.InputEventReceiver.nativeConsumeBatchedInputEvents(Native Method)
	at android.view.InputEventReceiver.consumeBatchedInputEvents(InputEventReceiver.java:176)
	at android.view.ViewRootImpl.doConsumeBatchedInput(ViewRootImpl.java:6441)
	at android.view.ViewRootImpl$ConsumeBatchedInputRunnable.run(ViewRootImpl.java:6507)
	at android.view.Choreographer$CallbackRecord.run(Choreographer.java:788)
	at android.view.Choreographer.doCallbacks(Choreographer.java:591)
	at android.view.Choreographer.doFrame(Choreographer.java:558)
	at android.view.Choreographer$FrameDisplayEventReceiver.run(Choreographer.java:774)
	at android.os.Handler.handleCallback(Handler.java:808)
	at android.os.Handler.dispatchMessage(Handler.java:103)
	at android.os.Looper.loop(Looper.java:193)
	at android.app.ActivityThread.main(ActivityThread.java:5351)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:515)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:829)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:645)
	at dalvik.system.NativeStart.main(Native Method)
```

系统版本特点：4.4.2以上  
机型分布特点：华为、三星、小米  
原因分析：调用`MotionEvent#getX();`或`MotionEvent#getX(int);`方法的时候系统找不到对应的pointerIndex，导致越界。  
解决办法：

* https://github.com/chrisbanes/PhotoView/issues/31

```
/** Custom your own ViewPager to extends support ViewPager. java source: */
/** Created by azi on 2013-6-21.  */

package com.chaokuadi.android.support.view;

import android.content.Context;
import android.util.AttributeSet;
import android.view.MotionEvent;

public class ViewPagerFixed extends android.support.v4.view.ViewPager {

    public ViewPagerFixed(Context context) {
        super(context);
    }

    public ViewPagerFixed(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        try {
            return super.onTouchEvent(ev);
        } catch (IllegalArgumentException ex) {
            ex.printStackTrace();
        }
        return false;
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        try {
            return super.onInterceptTouchEvent(ev);
        } catch (IllegalArgumentException ex) {
            ex.printStackTrace();
        }
        return false;
    }
}
```

* http://stackoverflow.com/questions/6919292/pointerindex-out-of-range-android-multitouch

有两个Pointer时判断`event.getPointerCount() >= 2`

* http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0501/2822.html

在我们的工程中，很有可能也是出现在这个`com.kaola.framework.ui.photoview.PhotoView`里。

在使用的地方也找到了下面类似的方法签名`MotionEventCompat#getX(MotionEvent, int)`

```
/**
 * Call {@link MotionEvent#getX(int)}.
 * If running on a pre-{@link android.os.Build.VERSION_CODES#ECLAIR} device,
 * {@link IndexOutOfBoundsException} is thrown.
 */
public static float getX(MotionEvent event, int pointerIndex) {
    return IMPL.getX(event, pointerIndex);
}
```

## java.lang.UnsatisfiedLinkError: dlopen failed: "/data/data/com.kaola/app_lib/libnelogin.so" is too small to be an ELF executable

```
java.lang.UnsatisfiedLinkError: dlopen failed: "/data/data/com.kaola/app_lib/libnelogin.so" is too small to be an ELF executable
	at java.lang.Runtime.load(Runtime.java:333)
	at java.lang.System.load(System.java:541)
	at com.netease.loginapi.util.e.a(Unknown Source)
	at com.netease.loginapi.util.NELoginJni.(Unknown Source)
	at com.netease.loginapi.NEConfig.(Unknown Source)
	at com.netease.loginapi.NELoginAPIFactory.createAPI(Unknown Source)
	at com.kaola.app.HTApplication.onCreate(Unknown Source)
	at android.app.Instrumentation.callApplicationOnCreate(Instrumentation.java:1009)
	at android.app.ActivityThread.handleBindApplication(ActivityThread.java:4613)
	at android.app.ActivityThread.access$1800(ActivityThread.java:139)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1316)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:136)
	at android.app.ActivityThread.main(ActivityThread.java:5314)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:515)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:864)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:680)
	at dalvik.system.NativeStart.main(Native Method)
```

系统版本特点：4.3.1以上  
机型分布特点：魅族>小米、华为、oppo等  
原因分析：在调用网易登录SDK的时候，找不到适配相关系统架构（x86、x86_64、armv7等）的so包，目前仅提供x86架构的so包，遇到其他CPU架构机子的时候可能会报这个问题。  
解决办法：提供全系列的so包，但不现实。


## java.lang.ExceptionInInitializerError

```
java.lang.ExceptionInInitializerError
	at com.tencent.tauth.Tencent.createInstance(Unknown Source)
	at com.netease.oauth.a.(Unknown Source)
	at com.netease.oauth.a.a(Unknown Source)
	at com.kaola.app.HTApplication.onCreate(Unknown Source)
	at android.app.Instrumentation.callApplicationOnCreate(Instrumentation.java:1007)
	at android.app.ActivityThread.handleBindApplication(ActivityThread.java:4564)
	at android.app.ActivityThread.access$1500(ActivityThread.java:151)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1402)
	at android.os.Handler.dispatchMessage(Handler.java:110)
	at android.os.Looper.loop(Looper.java:193)
	at android.app.ActivityThread.main(ActivityThread.java:5322)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:515)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:829)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:645)
	at dalvik.system.NativeStart.main(Native Method)
Caused by: java.lang.IllegalArgumentException: Invalid path: /storage/sdcard0
	at android.os.StatFs.doStat(StatFs.java:46)
	at android.os.StatFs.(StatFs.java:39)
	at com.tencent.open.a.d$c.b(Unknown Source)
	at com.tencent.open.a.d$b.b(Unknown Source)
	at com.tencent.open.a.f.c(Unknown Source)
	at com.tencent.open.a.f.(Unknown Source)
	... 16 more
Caused by: libcore.io.ErrnoException: statvfs failed: ENOTCONN (Transport endpoint is not connected)
	at libcore.io.Posix.statvfs(Native Method)
	at libcore.io.ForwardingOs.statvfs(ForwardingOs.java:132)
	at android.os.StatFs.doStat(StatFs.java:44)
	... 21 more
java.lang.IllegalArgumentException: Invalid path: /storage/sdcard0
	at android.os.StatFs.doStat(StatFs.java:46)
	at android.os.StatFs.(StatFs.java:39)
	at com.tencent.open.a.d$c.b(Unknown Source)
	at com.tencent.open.a.d$b.b(Unknown Source)
	at com.tencent.open.a.f.c(Unknown Source)
	at com.tencent.open.a.f.(Unknown Source)
	at com.tencent.tauth.Tencent.createInstance(Unknown Source)
	at com.netease.oauth.a.(Unknown Source)
	at com.netease.oauth.a.a(Unknown Source)
	at com.kaola.app.HTApplication.onCreate(Unknown Source)
	at android.app.Instrumentation.callApplicationOnCreate(Instrumentation.java:1007)
	at android.app.ActivityThread.handleBindApplication(ActivityThread.java:4564)
	at android.app.ActivityThread.access$1500(ActivityThread.java:151)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1402)
	at android.os.Handler.dispatchMessage(Handler.java:110)
	at android.os.Looper.loop(Looper.java:193)
	at android.app.ActivityThread.main(ActivityThread.java:5322)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:515)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:829)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:645)
	at dalvik.system.NativeStart.main(Native Method)
Caused by: libcore.io.ErrnoException: statvfs failed: ENOTCONN (Transport endpoint is not connected)
	at libcore.io.Posix.statvfs(Native Method)
	at libcore.io.ForwardingOs.statvfs(ForwardingOs.java:132)
	at android.os.StatFs.doStat(StatFs.java:44)
	... 21 more
libcore.io.ErrnoException: statvfs failed: ENOTCONN (Transport endpoint is not connected)
	at libcore.io.Posix.statvfs(Native Method)
	at libcore.io.ForwardingOs.statvfs(ForwardingOs.java:132)
	at android.os.StatFs.doStat(StatFs.java:44)
	at android.os.StatFs.(StatFs.java:39)
	at com.tencent.open.a.d$c.b(Unknown Source)
	at com.tencent.open.a.d$b.b(Unknown Source)
	at com.tencent.open.a.f.c(Unknown Source)
	at com.tencent.open.a.f.(Unknown Source)
	at com.tencent.tauth.Tencent.createInstance(Unknown Source)
	at com.netease.oauth.a.(Unknown Source)
	at com.netease.oauth.a.a(Unknown Source)
	at com.kaola.app.HTApplication.onCreate(Unknown Source)
	at android.app.Instrumentation.callApplicationOnCreate(Instrumentation.java:1007)
	at android.app.ActivityThread.handleBindApplication(ActivityThread.java:4564)
	at android.app.ActivityThread.access$1500(ActivityThread.java:151)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1402)
	at android.os.Handler.dispatchMessage(Handler.java:110)
	at android.os.Looper.loop(Looper.java:193)
	at android.app.ActivityThread.main(ActivityThread.java:5322)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:515)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:829)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:645)
	at dalvik.system.NativeStart.main(Native Method)
```

系统版本特点：4.4.2和4.4.4  
机型分布特点：华为、htc、dakele、酷派  
原因分析：腾讯登录sdk问题。猜测是他们用`StatFs`获取sd卡文件系统信息的时候出问题。  

```
private static StructStatVfs doStat(String path) {
    try {
        return Os.statvfs(path);
    } catch (ErrnoException e) {
        throw new IllegalArgumentException("Invalid path: " + path, e);
    }
}
```

解决办法：更新sdk

Ref：<https://github.com/paypal/PayPal-Android-SDK/issues/77>

## java.lang.RuntimeException: Unable to start receiver com.kaola.app.TimeChangeListener: java.lang.SecurityException: Permission Denial: get/set setting for user asks to run as user 0 but is calling from user 100; this requires android.permission.INTERACT\_ACROSS\_USERS\_FULL

```
java.lang.RuntimeException: Unable to start receiver com.kaola.app.TimeChangeListener: java.lang.SecurityException: Permission Denial: get/set setting for user asks to run as user 0 but is calling from user 100; this requires android.permission.INTERACT_ACROSS_USERS_FULL
	at android.app.ActivityThread.handleReceiver(ActivityThread.java:3641)
	at android.app.ActivityThread.access$2000(ActivityThread.java:221)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1876)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:158)
	at android.app.ActivityThread.main(ActivityThread.java:7224)
	at java.lang.reflect.Method.invoke(Native Method)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:1230)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1120)
Caused by: java.lang.SecurityException: Permission Denial: get/set setting for user asks to run as user 0 but is calling from user 100; this requires android.permission.INTERACT_ACROSS_USERS_FULL
	at android.os.Parcel.readException(Parcel.java:1620)
	at android.database.DatabaseUtils.readExceptionFromParcel(DatabaseUtils.java:183)
	at android.database.DatabaseUtils.readExceptionFromParcel(DatabaseUtils.java:135)
	at android.content.ContentProviderProxy.call(ContentProviderNative.java:646)
	at android.provider.Settings$NameValueCache.getStringForUser(Settings.java:1505)
	at android.provider.Settings$Secure.getStringForUser(Settings.java:8489)
	at com.netease.mobsecurity.factory.JNIFactory.w6e685a9adf5af10b(Native Method)
	at com.netease.mobsecurity.factory.a.a(Unknown Source)
	at com.netease.mobsecurity.impl.getInfo.a.a(Unknown Source)
	at com.netease.mobsecurity.interfacejni.SecruityInfo.getSecInfo(Unknown Source)
	at com.kaola.common.utils.i.b(Unknown Source)
	at com.kaola.spring.common.net.j.a(Unknown Source)
	at com.kaola.spring.common.net.a.a(Unknown Source)
	at com.kaola.spring.common.net.a.a(Unknown Source)
	at com.kaola.spring.common.net.a.a(Unknown Source)
	at com.kaola.app.TimeChangeListener.onReceive(Unknown Source)
	at android.app.ActivityThread.handleReceiver(ActivityThread.java:3634)
	... 8 more
java.lang.SecurityException: Permission Denial: get/set setting for user asks to run as user 0 but is calling from user 100; this requires android.permission.INTERACT_ACROSS_USERS_FULL
	at android.os.Parcel.readException(Parcel.java:1620)
	at android.database.DatabaseUtils.readExceptionFromParcel(DatabaseUtils.java:183)
	at android.database.DatabaseUtils.readExceptionFromParcel(DatabaseUtils.java:135)
	at android.content.ContentProviderProxy.call(ContentProviderNative.java:646)
	at android.provider.Settings$NameValueCache.getStringForUser(Settings.java:1505)
	at android.provider.Settings$Secure.getStringForUser(Settings.java:8489)
	at com.netease.mobsecurity.factory.JNIFactory.w6e685a9adf5af10b(Native Method)
	at com.netease.mobsecurity.factory.a.a(Unknown Source)
	at com.netease.mobsecurity.impl.getInfo.a.a(Unknown Source)
	at com.netease.mobsecurity.interfacejni.SecruityInfo.getSecInfo(Unknown Source)
	at com.kaola.common.utils.i.b(Unknown Source)
	at com.kaola.spring.common.net.j.a(Unknown Source)
	at com.kaola.spring.common.net.a.a(Unknown Source)
	at com.kaola.spring.common.net.a.a(Unknown Source)
	at com.kaola.spring.common.net.a.a(Unknown Source)
	at com.kaola.app.TimeChangeListener.onReceive(Unknown Source)
	at android.app.ActivityThread.handleReceiver(ActivityThread.java:3634)
	at android.app.ActivityThread.access$2000(ActivityThread.java:221)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1876)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:158)
	at android.app.ActivityThread.main(ActivityThread.java:7224)
	at java.lang.reflect.Method.invoke(Native Method)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:1230)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1120)
```

系统版本特点：5.0以上  
机型分布特点：三星、魅族、索尼  
原因分析：根据提示，`android.permission.INTERACT_ACROSS_USERS_FULL`权限未添加，与安全部门sdk有关。  
解决办法：更新安全部门sdk

## java.lang.UnsatisfiedLinkError: Couldn't load memchunk from loader dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.kaola-2.apk"],nativeLibraryDirectories=[/vendor/lib, /system/lib, /data/datalib]]]: findLibrary returned null

```
java.lang.UnsatisfiedLinkError: Couldn't load memchunk from loader dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.kaola-2.apk"],nativeLibraryDirectories=[/vendor/lib, /system/lib, /data/datalib]]]: findLibrary returned null
	at java.lang.Runtime.loadLibrary(Runtime.java:358)
	at java.lang.System.loadLibrary(System.java:555)
	at com.facebook.common.soloader.SoLoaderShim$DefaultHandler.a(Unknown Source)
	at com.facebook.common.soloader.SoLoaderShim.a(Unknown Source)
	at com.facebook.imagepipeline.memory.NativeMemoryChunk.(Unknown Source)
	at com.facebook.imagepipeline.memory.NativeMemoryChunkPool.i(Unknown Source)
	at com.facebook.imagepipeline.memory.NativeMemoryChunkPool.b(Unknown Source)
	at com.facebook.imagepipeline.memory.BasePool.a(Unknown Source)
	at com.facebook.imagepipeline.memory.NativePooledByteBufferOutputStream.(Unknown Source)
	at com.facebook.imagepipeline.memory.NativePooledByteBufferFactory.a(Unknown Source)
	at com.facebook.imagepipeline.memory.NativePooledByteBufferFactory.b(Unknown Source)
	at com.facebook.imagepipeline.cache.BufferedDiskCache.b(Unknown Source)
	at com.facebook.imagepipeline.cache.BufferedDiskCache.a(Unknown Source)
	at com.facebook.imagepipeline.cache.BufferedDiskCache$2.a(Unknown Source)
	at com.facebook.imagepipeline.cache.BufferedDiskCache$2.call(Unknown Source)
	at bolts.f.run(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1112)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:587)
	at java.lang.Thread.run(Thread.java:841)
```

系统版本特点：4.4.2  
机型分布特点：华为  
原因分析：找不到native包`libmemchunk.so`。  
Ref:<https://github.com/facebook/fresco/issues/410>

## java.lang.RuntimeException: startPreview failed

```
java.lang.RuntimeException: startPreview failed
	at android.hardware.Camera.startPreview(Native Method)
	at com.kaola.spring.ui.qrcode.a.c.d(Unknown Source)
	at com.kaola.spring.ui.qrcode.b.a.b(Unknown Source)
	at com.kaola.spring.ui.qrcode.b.a.(Unknown Source)
	at com.kaola.spring.ui.qrcode.QrCodeActivity.a(Unknown Source)
	at com.kaola.spring.ui.qrcode.QrCodeActivity.surfaceCreated(Unknown Source)
	at android.view.SurfaceView.updateWindow(SurfaceView.java:683)
	at android.view.SurfaceView$3.onPreDraw(SurfaceView.java:200)
	at android.view.ViewTreeObserver.dispatchOnPreDraw(ViewTreeObserver.java:1018)
	at android.view.ViewRootImpl.performTraversals(ViewRootImpl.java:2260)
	at android.view.ViewRootImpl.doTraversal(ViewRootImpl.java:1265)
	at android.view.ViewRootImpl$TraversalRunnable.run(ViewRootImpl.java:6944)
	at android.view.Choreographer$CallbackRecord.run(Choreographer.java:777)
	at android.view.Choreographer.doCallbacks(Choreographer.java:590)
	at android.view.Choreographer.doFrame(Choreographer.java:560)
	at android.view.Choreographer$FrameDisplayEventReceiver.run(Choreographer.java:763)
	at android.os.Handler.handleCallback(Handler.java:739)
	at android.os.Handler.dispatchMessage(Handler.java:95)
	at android.os.Looper.loop(Looper.java:145)
	at android.app.ActivityThread.main(ActivityThread.java:6862)
	at java.lang.reflect.Method.invoke(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:372)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:1404)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1199)
```

系统版本特点：4.2.1以上  
机型分布特点：魅族、华为、三星  
原因分析：相机预览失败，可能是部分厂商修改了这部分源码（相机源码通常是被修改得最频繁的一部分代码）。  	
解决办法：方法签名`public native final void startPreview()`，已经涉及native层代码，目前处理方案只能在preview的时候try-catch并提醒用户预览失败。

----

# Android其他版本较高崩溃率Bugs

## java.lang.RuntimeException: An error occured while executing doInBackground() (v2.1.2)
	
```
java.lang.RuntimeException: An error occured while executing doInBackground()
	at android.os.AsyncTask$3.done(AsyncTask.java:304)
	at java.util.concurrent.FutureTask.finishCompletion(FutureTask.java:355)
	at java.util.concurrent.FutureTask.setException(FutureTask.java:222)
	at java.util.concurrent.FutureTask.run(FutureTask.java:242)
	at android.os.AsyncTask$SerialExecutor$1.run(AsyncTask.java:231)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1112)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:587)
	at java.lang.Thread.run(Thread.java:818)
Caused by: java.lang.NullPointerException: Attempt to invoke virtual method 'com.netease.mobsecurity.impl.getInfo.IGenInfo com.netease.mobsecurity.impl.SecurityImplManager.getGenInfoComp()' on a null object reference
	at com.netease.mobsecurity.interfacejni.SecruityInfo.(Unknown Source)
	at com.kaola.common.utils.g.b(Unknown Source)
	at com.kaola.app.d.a(Unknown Source)
	at com.kaola.app.d.doInBackground(Unknown Source)
	at android.os.AsyncTask$2.call(AsyncTask.java:292)
	at java.util.concurrent.FutureTask.run(FutureTask.java:237)
	... 4 more
java.lang.NullPointerException: Attempt to invoke virtual method 'com.netease.mobsecurity.impl.getInfo.IGenInfo com.netease.mobsecurity.impl.SecurityImplManager.getGenInfoComp()' on a null object reference
	at com.netease.mobsecurity.interfacejni.SecruityInfo.(Unknown Source)
	at com.kaola.common.utils.g.b(Unknown Source)
	at com.kaola.app.d.a(Unknown Source)
	at com.kaola.app.d.doInBackground(Unknown Source)
	at android.os.AsyncTask$2.call(AsyncTask.java:292)
	at java.util.concurrent.FutureTask.run(FutureTask.java:237)
	at android.os.AsyncTask$SerialExecutor$1.run(AsyncTask.java:231)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1112)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:587)
	at java.lang.Thread.run(Thread.java:818)
```

原因分析：

	Caused by: java.lang.NullPointerException: Attempt to invoke virtual method 'com.netease.mobsecurity.impl.getInfo.IGenInfo com.netease.mobsecurity.impl.SecurityImplManager.getGenInfoComp()' on a null object reference

安全部门sdk的问题，猜测是Activity或Application结束时AsyncTask还在执行，结束后报NPE异常。  
解决办法：

* 安全部门提供异步调用获取设备id方案。
* DeviceInfoUtils.getDeviceId(getApplicationContext()); try-catch

## java.lang.RuntimeException: Canvas: trying to use a recycled bitmap android.graphics.Bitmap@57a98e98 (v2.4.2)

```
java.lang.RuntimeException: Canvas: trying to use a recycled bitmap android.graphics.Bitmap@57a98e98
	at android.graphics.Canvas.throwIfRecycled(Canvas.java:1026)
	at android.graphics.Canvas.setBitmap(Canvas.java:189)
	at android.view.View.buildDrawingCache(View.java:12978)
	at android.view.View.getDrawingCache(View.java:12796)
	at android.view.View.draw(View.java:13450)
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13763)
	at android.support.v4.view.ViewPager.draw(Unknown Source)
	at android.view.View.draw(View.java:13644)
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13642)v
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13642)
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13763)
	at android.view.View.draw(View.java:13644)
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13642)
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13763)
	at android.widget.FrameLayout.draw(FrameLayout.java:467)
	at android.widget.ScrollView.draw(ScrollView.java:1588)
	at android.view.View.draw(View.java:13644)
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13642)
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13642)
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13642)
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13763)
	at android.widget.FrameLayout.draw(FrameLayout.java:467)
	at android.view.View.draw(View.java:13644)
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13642)
	at android.view.ViewGroup.drawChild(ViewGroup.java:2930)
	at android.view.ViewGroup.dispatchDraw(ViewGroup.java:2799)
	at android.view.View.draw(View.java:13763)
	at android.widget.FrameLayout.draw(FrameLayout.java:467)
	at com.android.internal.policy.impl.PhoneWindow$DecorView.draw(PhoneWindow.java:2211)
	at android.view.ViewRootImpl.drawSoftware(ViewRootImpl.java:2288)
	at android.view.ViewRootImpl.draw(ViewRootImpl.java:2184)
	at android.view.ViewRootImpl.performDraw(ViewRootImpl.java:2052)
	at android.view.ViewRootImpl.performTraversals(ViewRootImpl.java:1861)
	at android.view.ViewRootImpl.doTraversal(ViewRootImpl.java:996)
	at android.view.ViewRootImpl$TraversalRunnable.run(ViewRootImpl.java:4358)
	at android.view.Choreographer$CallbackRecord.run(Choreographer.java:749)
	at android.view.Choreographer.doCallbacks(Choreographer.java:562)
	at android.view.Choreographer.doFrame(Choreographer.java:532)
	at android.view.Choreographer$FrameDisplayEventReceiver.run(Choreographer.java:735)
	at android.os.Handler.handleCallback(Handler.java:725)
	at android.os.Handler.dispatchMessage(Handler.java:92)
	at android.os.Looper.loop(Looper.java:137)
	at android.app.ActivityThread.main(ActivityThread.java:5054)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:511)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:793)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:560)
	at dalvik.system.NativeStart.main(Native Method)
```

原因分析：调用了`mBitmap.recycle();`后继续使用这个`mBitmap`的相关方法。  
解决办法：
	
```
if (mBitmap != null && !mBitmap.isRecycled()) {
    mBitmap.recycle();
    mBitmap = null; 
}
```

在使用`mBitmap`的时候记得判空。


## java.lang.IllegalStateException: Fragment already added: u{438fce18 #1 id=0x7f0a009b}

```
java.lang.IllegalStateException: Fragment already added: u{438fce18 #1 id=0x7f0a009b}
	at android.support.v4.app.q.a(Unknown Source)
	at android.support.v4.app.d.run(Unknown Source)
	at android.support.v4.app.q.f(Unknown Source)
	at android.support.v4.app.r.run(Unknown Source)
	at android.os.Handler.handleCallback(Handler.java:733)
	at android.os.Handler.dispatchMessage(Handler.java:95)
	at android.os.Looper.loop(Looper.java:136)
	at android.app.ActivityThread.main(ActivityThread.java:5315)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:515)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:864)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:680)
	at dalvik.system.NativeStart.main(Native Method)
```

原因分析：Fragment重复添加。  
解决办法：添加前判断存在性，或使用replace方法。

## java.lang.NullPointerException

```
java.lang.NullPointerException
	at android.webkit.WebViewClassic.loadDataWithBaseURL(WebViewClassic.java:3373)
	at android.webkit.WebView.loadDataWithBaseURL(WebView.java:852)
	at com.kaola.spring.ui.webview.m.run(Unknown Source)
	at android.os.Handler.handleCallback(Handler.java:615)
	at android.os.Handler.dispatchMessage(Handler.java:92)
	at android.os.Looper.loop(Looper.java:137)
	at android.app.ActivityThread.main(ActivityThread.java:4898)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:511)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:1008)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:775)
	at dalvik.system.NativeStart.main(Native Method)
```

原因分析：调用loadDataWithBaseURL()加载网页的时候未判空。

## java.lang.IllegalStateException: Cannot perform this operation because there is no current transaction.

```
java.lang.IllegalStateException: Cannot perform this operation because there is no current transaction.
	at android.database.sqlite.SQLiteSession.throwIfNoTransaction(SQLiteSession.java:915)
	at android.database.sqlite.SQLiteSession.endTransaction(SQLiteSession.java:398)
	at android.database.sqlite.SQLiteDatabase.endTransaction(SQLiteDatabase.java:522)
	at com.tencent.stat.n.b(Unknown Source)
	at com.tencent.stat.n.a(Unknown Source)
	at com.tencent.stat.q.run(Unknown Source)
	at android.os.Handler.handleCallback(Handler.java:725)
	at android.os.Handler.dispatchMessage(Handler.java:92)
	at android.os.Looper.loop(Looper.java:153)
	at android.os.HandlerThread.run(HandlerThread.java:60)
```
原因分析：可以调用多次beginTransaction()，但只能调用一次endTransaction()。这个异常就是调用多次endTransaction()造成的。

## android.database.sqlite.SQLiteException: Failed to change locale for db '/data/data/com.kaola/databases/webview.db' to 'zh_CN'.

```
android.database.sqlite.SQLiteException: Failed to change locale for db '/data/data/com.kaola/databases/webview.db' to 'zh_CN'.
	at android.database.sqlite.SQLiteConnection.setLocaleFromConfiguration(SQLiteConnection.java:390)
	at android.database.sqlite.SQLiteConnection.open(SQLiteConnection.java:218)
	at android.database.sqlite.SQLiteConnection.open(SQLiteConnection.java:193)
	at android.database.sqlite.SQLiteConnectionPool.openConnectionLocked(SQLiteConnectionPool.java:463)
	at android.database.sqlite.SQLiteConnectionPool.open(SQLiteConnectionPool.java:185)
	at android.database.sqlite.SQLiteConnectionPool.open(SQLiteConnectionPool.java:177)
	at android.database.sqlite.SQLiteDatabase.openInner(SQLiteDatabase.java:804)
	at android.database.sqlite.SQLiteDatabase.open(SQLiteDatabase.java:789)
	at android.database.sqlite.SQLiteDatabase.openDatabase(SQLiteDatabase.java:694)
	at android.app.ContextImpl.openOrCreateDatabase(ContextImpl.java:976)
	at android.app.ContextImpl.openOrCreateDatabase(ContextImpl.java:965)
	at android.content.ContextWrapper.openOrCreateDatabase(ContextWrapper.java:240)
	at android.webkit.WebViewDatabaseClassic.initDatabase(WebViewDatabaseClassic.java:146)
	at android.webkit.WebViewDatabaseClassic.init(WebViewDatabaseClassic.java:130)
	at android.webkit.WebViewDatabaseClassic.access$000(WebViewDatabaseClassic.java:35)
	at android.webkit.WebViewDatabaseClassic$1.run(WebViewDatabaseClassic.java:109)
Caused by: android.database.sqlite.SQLiteFullException: database or disk is full (code 13)
	at android.database.sqlite.SQLiteConnection.nativeExecute(Native Method)
	at android.database.sqlite.SQLiteConnection.execute(SQLiteConnection.java:552)
	at android.database.sqlite.SQLiteConnection.setLocaleFromConfiguration(SQLiteConnection.java:364)
	... 15 more
android.database.sqlite.SQLiteFullException: database or disk is full (code 13)
	at android.database.sqlite.SQLiteConnection.nativeExecute(Native Method)
	at android.database.sqlite.SQLiteConnection.execute(SQLiteConnection.java:552)
	at android.database.sqlite.SQLiteConnection.setLocaleFromConfiguration(SQLiteConnection.java:364)
	at android.database.sqlite.SQLiteConnection.open(SQLiteConnection.java:218)
	at android.database.sqlite.SQLiteConnection.open(SQLiteConnection.java:193)
	at android.database.sqlite.SQLiteConnectionPool.openConnectionLocked(SQLiteConnectionPool.java:463)
	at android.database.sqlite.SQLiteConnectionPool.open(SQLiteConnectionPool.java:185)
	at android.database.sqlite.SQLiteConnectionPool.open(SQLiteConnectionPool.java:177)
	at android.database.sqlite.SQLiteDatabase.openInner(SQLiteDatabase.java:804)
	at android.database.sqlite.SQLiteDatabase.open(SQLiteDatabase.java:789)
	at android.database.sqlite.SQLiteDatabase.openDatabase(SQLiteDatabase.java:694)
	at android.app.ContextImpl.openOrCreateDatabase(ContextImpl.java:976)
	at android.app.ContextImpl.openOrCreateDatabase(ContextImpl.java:965)
	at android.content.ContextWrapper.openOrCreateDatabase(ContextWrapper.java:240)
	at android.webkit.WebViewDatabaseClassic.initDatabase(WebViewDatabaseClassic.java:146)
	at android.webkit.WebViewDatabaseClassic.init(WebViewDatabaseClassic.java:130)
	at android.webkit.WebViewDatabaseClassic.access$000(WebViewDatabaseClassic.java:35)
	at android.webkit.WebViewDatabaseClassic$1.run(WebViewDatabaseClassic.java:109)
```
原因分析：在生成数据库db文件的时候使用了A区域，在打开的时候默认使用B区域打开，如A=Locale.US，B=Locale.CHINA，会产生这种问题。  
解决方法：

把

	myDatabase = SQLiteDatabase.openDatabase(myPath, null, SQLiteDatabase.OPEN_READWRITE);

修改成：  

	myDatabase = SQLiteDatabase.openDatabase(myPath, null, SQLiteDatabase.NO_LOCALIZED_COLLATORS | SQLiteDatabase.OPEN_READWRITE);

Ref：<http://stackoverflow.com/questions/19491675/failed-to-change-locale-for-db-data-data-my-easymedi-controller-databases-easy>

## Unable to stop activity {com.kaola/com.kaola.spring.ui.MainActivity}: java.lang.IllegalArgumentException: View not attached to window manager

```
java.lang.RuntimeException: Unable to stop activity {com.kaola/com.kaola.spring.ui.MainActivity}: java.lang.IllegalArgumentException: View not attached to window manager
	at android.app.ActivityThread.performStopActivityInner(ActivityThread.java:3393)
	at android.app.ActivityThread.handleStopActivity(ActivityThread.java:3446)
	at android.app.ActivityThread.access$900(ActivityThread.java:171)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1414)
	at android.os.Handler.dispatchMessage(Handler.java:107)
	at android.os.Looper.loop(Looper.java:194)
	at android.app.ActivityThread.main(ActivityThread.java:5433)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:525)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:853)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:620)
	at dalvik.system.NativeStart.main(Native Method)
Caused by: java.lang.IllegalArgumentException: View not attached to window manager
	at android.view.WindowManagerGlobal.findViewLocked(WindowManagerGlobal.java:385)
	at android.view.WindowManagerGlobal.removeView(WindowManagerGlobal.java:287)
	at android.view.WindowManagerImpl.removeViewImmediate(WindowManagerImpl.java:89)
	at android.widget.PopupWindow.dismiss(PopupWindow.java:1304)
	at com.kaola.spring.ui.MainActivity.q(Unknown Source)
	at com.kaola.spring.ui.MainActivity.onStop(Unknown Source)
	at android.app.Instrumentation.callActivityOnStop(Instrumentation.java:1280)
	at com.lbe.security.service.core.client.b.x.callActivityOnStop(Unknown Source)
	at android.app.Activity.performStop(Activity.java:5373)
	at android.app.ActivityThread.performStopActivityInner(ActivityThread.java:3390)
	... 11 more
java.lang.IllegalArgumentException: View not attached to window manager
	at android.view.WindowManagerGlobal.findViewLocked(WindowManagerGlobal.java:385)
	at android.view.WindowManagerGlobal.removeView(WindowManagerGlobal.java:287)
	at android.view.WindowManagerImpl.removeViewImmediate(WindowManagerImpl.java:89)
	at android.widget.PopupWindow.dismiss(PopupWindow.java:1304)
	at com.kaola.spring.ui.MainActivity.q(Unknown Source)
	at com.kaola.spring.ui.MainActivity.onStop(Unknown Source)
	at android.app.Instrumentation.callActivityOnStop(Instrumentation.java:1280)
	at com.lbe.security.service.core.client.b.x.callActivityOnStop(Unknown Source)
	at android.app.Activity.performStop(Activity.java:5373)
	at android.app.ActivityThread.performStopActivityInner(ActivityThread.java:3390)
	at android.app.ActivityThread.handleStopActivity(ActivityThread.java:3446)
	at android.app.ActivityThread.access$900(ActivityThread.java:171)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1414)
	at android.os.Handler.dispatchMessage(Handler.java:107)
	at android.os.Looper.loop(Looper.java:194)
	at android.app.ActivityThread.main(ActivityThread.java:5433)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:525)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:853)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:620)
	at dalvik.system.NativeStart.main(Native Method)
```

原因分析：MainActivity在执行`onStop()`的同时，调用了`showFindHobbyPopWindow()`并启动新的Activity  
解决办法：`isFinishing()`（没试过，不知道有没有用）

Ref：<http://stackoverflow.com/questions/22924825/view-not-attached-to-window-manager-crash>

## java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState

```
java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState
	at android.support.v4.app.s.v(Unknown Source)
	at android.support.v4.app.s.d(Unknown Source)
	at android.support.v4.app.n.onBackPressed(Unknown Source)
	at com.kaola.common.widgets.f.onClick(Unknown Source)
	at android.view.View.performClick(View.java:5207)
	at android.view.View$PerformClick.run(View.java:21177)
	at android.os.Handler.handleCallback(Handler.java:739)
	at android.os.Handler.dispatchMessage(Handler.java:95)
	at android.os.Looper.loop(Looper.java:148)
	at android.app.ActivityThread.main(ActivityThread.java:5432)
	at java.lang.reflect.Method.invoke(Native Method)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:735)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:625)
```

原因分析：在调用onSaveInstanceState（猜测是替换掉当前的Fragment）和commitAllowingStateLoss的时候更新了UI线程。

```
private void checkStateLoss() {
    if (mStateSaved) {
        throw new IllegalStateException(
                "Can not perform this action after onSaveInstanceState");
    }
    if (mNoTransactionsBecause != null) {
        throw new IllegalStateException(
                "Can not perform this action inside of " + mNoTransactionsBecause);
    }
}
```

`mStateSaved`只有在保存数据到`Parcel`的时候才会置为`true`。

Ref：<http://stackoverflow.com/questions/17184653/commitallowingstateloss-in-fragment-activities>

## java.lang.RuntimeException: Unable to start activity ComponentInfo{com.kaola/com.kaola.spring.ui.MainActivity}: android.content.res.Resources$NotFoundException: File res/layout/activity_main.xml from xml type layout resource ID #0x7f03003f

```
java.lang.RuntimeException: Unable to start activity ComponentInfo{com.kaola/com.kaola.spring.ui.MainActivity}: android.content.res.Resources$NotFoundException: File res/layout/activity_main.xml from xml type layout resource ID #0x7f03003f
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2187)
	at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2247)
	at android.app.ActivityThread.access$800(ActivityThread.java:138)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1199)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:136)
	at android.app.ActivityThread.main(ActivityThread.java:5106)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:515)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:818)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:634)
	at dalvik.system.NativeStart.main(Native Method)
Caused by: android.content.res.Resources$NotFoundException: File res/layout/activity_main.xml from xml type layout resource ID #0x7f03003f
	at android.content.res.Resources.loadXmlResourceParser(Resources.java:2504)
	at android.content.res.Resources.loadXmlResourceParser(Resources.java:2459)
	at android.content.res.Resources.getLayout(Resources.java:982)
	at android.view.LayoutInflater.inflate(LayoutInflater.java:395)
	at android.view.LayoutInflater.inflate(LayoutInflater.java:353)
	at com.android.internal.policy.impl.PhoneWindow.setContentView(PhoneWindow.java:290)
	at android.app.Activity.setContentView(Activity.java:1940)
	at com.kaola.spring.ui.MainActivity.onCreate(Unknown Source)
	at android.app.Activity.performCreate(Activity.java:5242)
	at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1087)
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2151)
	... 11 more
Caused by: java.io.FileNotFoundException: res/layout/activity_main.xml
	at android.content.res.AssetManager.openXmlAssetNative(Native Method)
	at android.content.res.AssetManager.openXmlBlockAsset(AssetManager.java:504)
	at android.content.res.Resources.loadXmlResourceParser(Resources.java:2486)
	... 21 more
android.content.res.Resources$NotFoundException: File res/layout/activity_main.xml from xml type layout resource ID #0x7f03003f
	at android.content.res.Resources.loadXmlResourceParser(Resources.java:2504)
	at android.content.res.Resources.loadXmlResourceParser(Resources.java:2459)
	at android.content.res.Resources.getLayout(Resources.java:982)
	at android.view.LayoutInflater.inflate(LayoutInflater.java:395)
	at android.view.LayoutInflater.inflate(LayoutInflater.java:353)
	at com.android.internal.policy.impl.PhoneWindow.setContentView(PhoneWindow.java:290)
	at android.app.Activity.setContentView(Activity.java:1940)
	at com.kaola.spring.ui.MainActivity.onCreate(Unknown Source)
	at android.app.Activity.performCreate(Activity.java:5242)
	at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1087)
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2151)
	at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2247)
	at android.app.ActivityThread.access$800(ActivityThread.java:138)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1199)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:136)
	at android.app.ActivityThread.main(ActivityThread.java:5106)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:515)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:818)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:634)
	at dalvik.system.NativeStart.main(Native Method)
Caused by: java.io.FileNotFoundException: res/layout/activity_main.xml
	at android.content.res.AssetManager.openXmlAssetNative(Native Method)
	at android.content.res.AssetManager.openXmlBlockAsset(AssetManager.java:504)
	at android.content.res.Resources.loadXmlResourceParser(Resources.java:2486)
	... 21 more
java.io.FileNotFoundException: res/layout/activity_main.xml
	at android.content.res.AssetManager.openXmlAssetNative(Native Method)
	at android.content.res.AssetManager.openXmlBlockAsset(AssetManager.java:504)
	at android.content.res.Resources.loadXmlResourceParser(Resources.java:2486)
	at android.content.res.Resources.loadXmlResourceParser(Resources.java:2459)
	at android.content.res.Resources.getLayout(Resources.java:982)
	at android.view.LayoutInflater.inflate(LayoutInflater.java:395)
	at android.view.LayoutInflater.inflate(LayoutInflater.java:353)
	at com.android.internal.policy.impl.PhoneWindow.setContentView(PhoneWindow.java:290)
	at android.app.Activity.setContentView(Activity.java:1940)
	at com.kaola.spring.ui.MainActivity.onCreate(Unknown Source)
	at android.app.Activity.performCreate(Activity.java:5242)
	at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1087)
	at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2151)
	at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2247)
	at android.app.ActivityThread.access$800(ActivityThread.java:138)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1199)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:136)
	at android.app.ActivityThread.main(ActivityThread.java:5106)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:515)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:818)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:634)
	at dalvik.system.NativeStart.main(Native Method)
```

原因分析：这个问题发生的比较诡异，机型不固定，系统版本不固定。目前只有几个资源文件找不到：
	
* `res/drawable-xxhdpi-v4/ic_comment_praise_unclick.png`
* `res/layout/activity_main.xml`
* `res/drawable-xxhdpi-v4/aftersale_select_logistics.png`
* `res/anim/home_search_view_in.xml`
* `res/drawable-xxhdpi-v4/bg_splash.png`
* `res/drawable-xxhdpi-v4/ic_goods_detail_share_white.png`
* `res/drawable-xhdpi-v4/home_search_icon.png`

大部分是去v4这个包下面去找资源，但我们的配置里好像没有资源文件是放在v4文件夹下面的。猜测可能是系统厂商改了获取资源路径的源码导致获取失败。

## java.lang.IllegalArgumentException: View=com.android.internal.policy.impl.PhoneWindow$DecorView{24e9c19a V.E..... R......D 0,0-1026,348} not attached to window manager

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

原因分析：非常典型的Activity已经被`finish()`，但Dialog还没有被`dimiss()`造成的异常。  
解决办法：在`onPause()`或`onStop()`方法调用`Dialog.dismiss()`。