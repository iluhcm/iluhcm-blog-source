title: Android长按及拖动事件探究
tags: [Android,View,OnLongClickListener,OnDragListener,GestureDetector,OnTouchListener]
date: 2015-09-07 18:22:43
categories: Android
---

Android中长按拖动还是比较常见的.比如Launcher中的图标拖动及屏幕切换,ListView中item顺序的改变,新闻类App中新闻类别的顺序改变等.下面就这个事件做一下分析.

# Android Long Click Event

就目前而言,Android中实现长按事件响应有几种方式,包括:

* 设置View.OnLongClickListener监听器
* 通过GestureDetector.OnGestureListener间接获取长按事件
* 实现View.OnTouchListener,然后在回调中通过MotionEvent判断是否触发了长按事件

下面分别介绍这三种方式.

## View.OnLongClickListener

对于Android中的任何一个View,都可以实现长按事件监听,并回调这个事件.在`View`类里,定义了`OnLongClickListener `.

```
/**
 * Interface definition for a callback to be invoked when a view has been clicked and held.
 */
public interface OnLongClickListener {
    /**
     * Called when a view has been clicked and held.
     *
     * @param v The view that was clicked and held.
     *
     * @return true if the callback consumed the long click, false otherwise.
     */
    boolean onLongClick(View v);
}
```

默认情况下,View类是不支持长按的,由`LONG_CLICKABLE`这个标记控制.如果设置了监听器,则会默认打开支持长按的开关,并回调上面的`boolean onLongClick(View v)`方法.从注释的返回值中可以看到,如果这个回调消费了长按事件,则返回`true`,否则返回`false`.这和`View`类中的各种触摸事件传递是一致的.

```
/**
 * Register a callback to be invoked when this view is clicked and held. If this view is not
 * long clickable, it becomes long clickable.
 *
 * @param l The callback that will run
 *
 * @see #setLongClickable(boolean)
 */
public void setOnLongClickListener(@Nullable OnLongClickListener l) {
    if (!isLongClickable()) {
        setLongClickable(true);
    }
    getListenerInfo().mOnLongClickListener = l;
}
```

其中, `getListenerInfo()`返回一个包含了一个View类中所有的监听器事件的静态内部类`ListenerInfo`.

### 简单实例

```
ImageView imageView = new ImageView(this);
imageView.setImageResource(R.drawable.ic_launcher);
imageView.setOnLongClickListener(new View.OnLongClickListener() {
    @Override
    public boolean onLongClick(View v) {
        Log.v(TAG, "perform long click.");
        return false;
    }
});
```

## GestureDetector.OnGestureListener

`GestureDetector`提供了丰富的手势识别功能.除了支持长按事件监听外,还支持多种手势事件监听.在`GestureDetector.OnGestureListener`这个监听器中,提供了6种手势监听回调:

* boolean onDown(MotionEvent e);
* void onShowPress(MotionEvent e);
* boolean onSingleTapUp(MotionEvent e);
* boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY);
* void onLongPress(MotionEvent e);
* boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY);

几乎包含了一次界面触摸操作所能想到的所有操作.其中,可以通过`void onLongPress(MotionEvent e)`来实现长按监听.

### 简单实例

```
package com.amap.mock.activity;

import android.app.Activity;
import android.content.Context;
import android.content.Intent;
import android.os.Bundle;
import android.util.Log;
import android.view.GestureDetector;
import android.view.MotionEvent;
import android.view.View;

import com.amap.mock.R;

/**
 * Created by xingli on 8/19/15.
 * 
 * An example of performing click event.
 */
public class GestureActivity extends Activity implements GestureDetector.OnGestureListener {

    private static final String TAG = GestureActivity.class.getSimpleName();

    private GestureDetector gestureDetector;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test);
        gestureDetector = new GestureDetector(this, this);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        return gestureDetector.onTouchEvent(event);
    }

    @Override
    public boolean onDown(MotionEvent e) {
        Log.v(TAG,"onDown");
        return false;
    }

    @Override
    public void onShowPress(MotionEvent e) {
        Log.v(TAG,"onShowPress");
    }

    @Override
    public boolean onSingleTapUp(MotionEvent e) {
        Log.v(TAG,"onSingleTapUp");
        return false;
    }

    @Override
    public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
        Log.v(TAG,"onScroll");
        return false;
    }

    @Override
    public void onLongPress(MotionEvent e) {
        Log.v(TAG,"onLongPress");
    }

    @Override
    public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
        Log.v(TAG,"onFling");
        return false;
    }
}

```

### GestureDetector长按事件原理解析

在上面的例子中,我们看到在`GestureDetector`这个类中,实现了`onTouchEvent()`方法,直接代替`View`类中的`onTouchEvent()`方法,即可实现触摸事件的检测.下面是`GestureDetector.onTouchEvent()`的部分关键源码:

```
public boolean onTouchEvent(MotionEvent ev) {
    if (mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onTouchEvent(ev, 0);
    }

    final int action = ev.getAction();
    ...
    boolean handled = false;

    switch (action & MotionEvent.ACTION_MASK) {
    case MotionEvent.ACTION_POINTER_DOWN:
        ...
        break;
   case MotionEvent.ACTION_POINTER_UP:
        ...
        break;
   case MotionEvent.ACTION_DOWN:
        if (mDoubleTapListener != null) {
            boolean hadTapMessage = mHandler.hasMessages(TAP);
            if (hadTapMessage) mHandler.removeMessages(TAP);
            if ((mCurrentDownEvent != null) && (mPreviousUpEvent != null) && hadTapMessage &&
                    isConsideredDoubleTap(mCurrentDownEvent, mPreviousUpEvent, ev)) {
                // This is a second tap
                mIsDoubleTapping = true;
                // Give a callback with the first tap of the double-tap
                handled |= mDoubleTapListener.onDoubleTap(mCurrentDownEvent);
                // Give a callback with down event of the double-tap
                handled |= mDoubleTapListener.onDoubleTapEvent(ev);
            } else {
                // This is a first tap
                mHandler.sendEmptyMessageDelayed(TAP, DOUBLE_TAP_TIMEOUT);
            }
        }

        mDownFocusX = mLastFocusX = focusX;
        mDownFocusY = mLastFocusY = focusY;
        if (mCurrentDownEvent != null) {
            mCurrentDownEvent.recycle();
        }
        mCurrentDownEvent = MotionEvent.obtain(ev);
        mAlwaysInTapRegion = true;
        mAlwaysInBiggerTapRegion = true;
        mStillDown = true;
        mInLongPress = false;
        mDeferConfirmSingleTap = false;

        if (mIsLongpressEnabled) {
            mHandler.removeMessages(LONG_PRESS);
            mHandler.sendEmptyMessageAtTime(LONG_PRESS, mCurrentDownEvent.getDownTime()
                    + TAP_TIMEOUT + LONGPRESS_TIMEOUT);
        }
        mHandler.sendEmptyMessageAtTime(SHOW_PRESS, mCurrentDownEvent.getDownTime() + TAP_TIMEOUT);
        handled |= mListener.onDown(ev);
        break;

    case MotionEvent.ACTION_MOVE:
        if (mInLongPress || mInContextClick) {
            break;
        }
        final float scrollX = mLastFocusX - focusX;
        final float scrollY = mLastFocusY - focusY;
        if (mIsDoubleTapping) {
            // Give the move events of the double-tap
            handled |= mDoubleTapListener.onDoubleTapEvent(ev);
        } else if (mAlwaysInTapRegion) {
            final int deltaX = (int) (focusX - mDownFocusX);
            final int deltaY = (int) (focusY - mDownFocusY);
            int distance = (deltaX * deltaX) + (deltaY * deltaY);
            if (distance > mTouchSlopSquare) {
                handled = mListener.onScroll(mCurrentDownEvent, ev, scrollX, scrollY);
                mLastFocusX = focusX;
                mLastFocusY = focusY;
                mAlwaysInTapRegion = false;
                mHandler.removeMessages(TAP);
                mHandler.removeMessages(SHOW_PRESS);
                mHandler.removeMessages(LONG_PRESS);
            }
            if (distance > mDoubleTapTouchSlopSquare) {
                mAlwaysInBiggerTapRegion = false;
            }
        } else if ((Math.abs(scrollX) >= 1) || (Math.abs(scrollY) >= 1)) {
            handled = mListener.onScroll(mCurrentDownEvent, ev, scrollX, scrollY);
            mLastFocusX = focusX;
            mLastFocusY = focusY;
        }
        break;
    case MotionEvent.ACTION_UP:
        mStillDown = false;
        MotionEvent currentUpEvent = MotionEvent.obtain(ev);
        if (mIsDoubleTapping) {
            // Finally, give the up event of the double-tap
            handled |= mDoubleTapListener.onDoubleTapEvent(ev);
        } else if (mInLongPress) {
            mHandler.removeMessages(TAP);
            mInLongPress = false;
        } else if (mAlwaysInTapRegion && !mIgnoreNextUpEvent) {
            handled = mListener.onSingleTapUp(ev);
            if (mDeferConfirmSingleTap && mDoubleTapListener != null) {
                mDoubleTapListener.onSingleTapConfirmed(ev);
            }
        } else if (!mIgnoreNextUpEvent) {
           // A fling must travel the minimum tap distance
            final VelocityTracker velocityTracker = mVelocityTracker;
            final int pointerId = ev.getPointerId(0);
            velocityTracker.computeCurrentVelocity(1000, mMaximumFlingVelocity);
            final float velocityY = velocityTracker.getYVelocity(pointerId);
            final float velocityX = velocityTracker.getXVelocity(pointerId);

            if ((Math.abs(velocityY) > mMinimumFlingVelocity)
                    || (Math.abs(velocityX) > mMinimumFlingVelocity)){
                handled = mListener.onFling(mCurrentDownEvent, ev, velocityX, velocityY);
            }
        }
        if (mPreviousUpEvent != null) {
            mPreviousUpEvent.recycle();
        }
        // Hold the event we obtained above - listeners may have changed the original.
        mPreviousUpEvent = currentUpEvent;
        if (mVelocityTracker != null) {
            // This may have been cleared when we called out to the
            // application above.
            mVelocityTracker.recycle();
            mVelocityTracker = null;
        }
        mIsDoubleTapping = false;
        mDeferConfirmSingleTap = false;
        mIgnoreNextUpEvent = false;
        mHandler.removeMessages(SHOW_PRESS);
        mHandler.removeMessages(LONG_PRESS);
        break;

    case MotionEvent.ACTION_CANCEL:
        cancel();
        break;
    }

    if (!handled && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(ev, 0);
    }
    return handled;
}
```

在`onTouchEvent()`方法中,很明显是通过Handler来传递触摸事件并触发相关的回调的.因为Handler是通过一个串行的队列来处理消息的,可以防止并发触摸操作时产生行为逻辑的混乱.在此方法中,可以看到对MotionEvent事件的处理,就长按事件来说,分为:

* MotionEvent.ACTION_DOWN
* MotionEvent.ACTION_MOVE
* MotionEvent.ACTION_UP

在`MotionEvent.ACTION_DOWN`阶段,程序做了3件事:

1. 判断双击事件(我们不关心是否双击,因为没有设置这个监听器,也不是本文讨论的重点);
2. 进行初始化操作.包括:
   * mDownFocusX = mLastFocusX = focusX;  
     mDownFocusY = mLastFocusY = focusY; // 记录焦点坐标,用于判断在按下的过程中是否发生了手指的移动
   * mAlwaysInTapRegion = true;			// 按下了相应的区域,判断单击事件并制定后来的事件响应机制
   * mAlwaysInBiggerTapRegion = true;	// 按下了相应的大区域,判断双击事件
   * mStillDown = true;						// 用于判断用户是轻轻触摸了一下还是一直按下
   * mInLongPress = false;					// 判断是否正在长按
   * mDeferConfirmSingleTap = false;		// 用于处理是否是一次`TAP`事件
3. 通过发送延时消息来判断触不触发长按事件:

```
if (mIsLongpressEnabled) {
    mHandler.removeMessages(LONG_PRESS);
    mHandler.sendEmptyMessageAtTime(LONG_PRESS, mCurrentDownEvent.getDownTime()
            + TAP_TIMEOUT + LONGPRESS_TIMEOUT);
}
```
默认的`TAP_TIMEOUT`是100ms,`LONGPRESS_TIMEOUT`是500ms.这两个参数在`ViewConfiguration.java`类中有定义,并且暂时不提供API更改触发值.

```
/**
 * Defines the default duration in milliseconds before a press turns into
 * a long press
 */
private static final int DEFAULT_LONG_PRESS_TIMEOUT = 500;

/**
 * Defines the duration in milliseconds we will wait to see if a touch event
 * is a tap or a scroll. If the user does not move within this interval, it is
 * considered to be a tap.
 */
private static final int TAP_TIMEOUT = 100;
```
接下来,在`MotionEvent.ACTION_MOVE`阶段,程序判断比较简单.

1. 如果正在长按或者是在上下文中点击,则跳出循环;
```
if (mInLongPress || mInContextClick) {
    break;
}
```
2. 判断是不是双击事件(`mAlwaysInTapRegion`); 
3. 如果不是双击事件,则判断是不是还在之前触摸的那个区域(`mAlwaysInTapRegion`);如果是,由于触发了`ACTION_MOVE`事件,那么说明手指已经移动过了N个单位距离,这时候,需要判断这个距离是不是大于某个阈值`mTouchSlopSquare`,其中
`mTouchSlopSquare=configuration.getScaledTouchSlop()^2`,  
在配置文件中默认值为`8dip`.如果大于这个阈值,则说明移动确实发生了,这时候:
    * handled = mListener.onScroll(mCurrentDownEvent, ev, scrollX, scrollY);	//设置滚动监听回调
    * mLastFocusX = focusX;
    * mLastFocusY = focusY; // 重新设置触摸焦点
    * mAlwaysInTapRegion = false;	// 重置触摸区域判断
    * mHandler.removeMessages(TAP);
    * mHandler.removeMessages(SHOW_PRESS);
    * mHandler.removeMessages(LONG_PRESS);	// 移除所有触摸相关的消息事件
4. 如果以上两项都不符合,那么则确定为滚动事件,并重置焦点:
    * handled = mListener.onScroll(mCurrentDownEvent, ev, scrollX, scrollY);
    * mLastFocusX = focusX;
    * mLastFocusY = focusY;

处理完成移动事件后,到了`MotionEvent.ACTION_UP`阶段,程序主要判断当前处于哪个阶段,然后分别针对这个阶段做事件清除,资源回收,重置各种触摸状态.由于程序比较简单,就不再详细分析这个阶段的消息了.

## View.OnTouchListener

看了`GestureDetector.onTouchEvent`的源码后,是不是觉得长按事件检测与处理很简单?接下来的这种方法就是借鉴了第二种方法来实现的,主要原理就是利用Handler发送延时消息来判断是不是触发了长按事件.不过我是在`MotionEvent.ACTION_MOVE`阶段来判断长按事件,这样做的原因留给后面来分析.先看代码:

```
package com.amap.mock.activity;

import android.os.Handler;
import android.util.Log;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewConfiguration;

/**
 * Created by xingli on 9/7/15.
 * 
 * A long press event detector.
 */
public class LongPressHandler implements View.OnTouchListener {
    private static final String TAG = LongPressHandler2.class.getSimpleName();

    // Default long press time threshold.
    private static final long LONG_PRESS_TIME_THRESHOLD = 500;
    // Long press event message handler.
    private Handler mHandler = new Handler();
    // The long press time threshold.
    private long mPressTimeThreshold;
    // Record start point and end point to judge whether user has moved while performing long press event.
    private DoublePoint mTouchStartPoint = new DoublePoint();
    private DoublePoint mTouchEndPoint = new DoublePoint();
    // The long press thread.
    private final LongPressThread mLongPressThread = new LongPressThread();
    // Inset in pixels to look for touchable content when the user touches the edge of the screen.
    private final float mTouchSlop;
    // The long press callback.
    private OnLongPressListener listener;

    public LongPressHandler2(View view) {
        this(view, LONG_PRESS_TIME_THRESHOLD);
    }

    public LongPressHandler2(View view, long holdTime) {
        view.setOnTouchListener(this);
        mTouchSlop = ViewConfiguration.get(view.getContext()).getScaledEdgeSlop();
        Log.v(TAG, "touch slop:" + mTouchSlop);
        mPressTimeThreshold = holdTime;
    }

    @Override
    public boolean onTouch(View v, MotionEvent event) {
        int action = event.getAction();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                mTouchStartPoint.x = event.getRawX();
                mTouchStartPoint.y = event.getRawY();
                addLongPressCallback();
                break;
            case MotionEvent.ACTION_MOVE:
                mTouchEndPoint.x = event.getRawX();
                mTouchEndPoint.y = event.getRawY();
                // If user is pressing and dragging, then we make a callback.
                if (mLongPressThread.mLongPressing) {
                    resetLongPressEvent();
                    if (listener != null) {
                        return listener.onLongPressed(event);
                    }
                    break;
                }
                // If user has moved before activating long press event, then the event should be reset.
                if (calculateDistanceBetween(mTouchStartPoint, mTouchEndPoint) > mTouchSlop) {
                    resetLongPressEvent();
                }
                break;
            case MotionEvent.ACTION_UP:
                if (mLongPressThread.mLongPressing) {
                    resetLongPressEvent();
                    // Must set true and left the child know we have handled this event.
                    return true;
                }
            default:
                resetLongPressEvent();
                break;
        }
        return false;
    }

    public void setOnLongPressListener(OnLongPressListener listener) {
        this.listener = listener;
    }

    /**
     * Reset the long press event.
     */
    private void resetLongPressEvent() {
        if (mLongPressThread.mAdded) {
            mHandler.removeCallbacks(mLongPressThread);
            mLongPressThread.mAdded = false;
        }
        mLongPressThread.mLongPressing = false;
    }

    /**
     * Add long press event handler.
     */
    private void addLongPressCallback() {
        if (!mLongPressThread.mAdded) {
            mLongPressThread.mLongPressing = false;
            mHandler.postDelayed(mLongPressThread, mPressTimeThreshold);
            mLongPressThread.mAdded = true;
        }
    }

    /**
     * Calculate distance between two point.
     * 
     * @param before previous point
     * @param after next point
     * @return the distance
     */
    private double calculateDistanceBetween(DoublePoint before, DoublePoint after) {
        return Math.sqrt(Math.pow((before.x - after.x), 2) + Math.pow((before.y - after.y), 2));
    }

    /**
     * Judge whether the long press event happens.
     *
     * The time threshold of default activated event is {@see #LONG_PRESS_TIME_THRESHOLD}
     */
    private static class LongPressThread implements Runnable {
        // A flag to set whether the long press event happens.
        boolean mLongPressing = false;
        // A flag to set whether this thread has been added to the handler.
        boolean mAdded = false;

        @Override
        public void run() {
            mLongPressing = true;
        }
    }

    private static class DoublePoint {
        public double x;
        public double y;
    }

    /**
     * The long press listener.
     */
    public interface OnLongPressListener {
        /**
         * Notified when a long press occurs with the initial on down {@link MotionEvent} that trigged it.
         */
        boolean onLongPressed(MotionEvent event);
    }

}

```

这段代码可能没有`GestureDetector`这个类写的这么规范和完整,但至少能够实现长按触发并且实现事件的回调.使用这个类有以下两个限制:

- 在`LongPressHandler`这个类的构造函数中,设置了`OnTouchListener`监听器,因此如果这个View在其他地方也设置了同样的监听器,有可能不起作用,以最后一个初始化该监听器的类其作用为标准;
- 必须设置`View`为可点击的.即`View.setClickable(true)`.显然,不可点击的话就没有长按事件了.

## 长按事件小结

经过上面的分析,我们通过三种方式实现了长按事件的检测及事件回调处理,分别是`View.OnLongClickListener`,`GestureDetector.OnGestureListener`以及`View.OnTouchListener`.

如果仅仅是考虑长按事件,那么直接设置`View.OnLongClickListener`监听器是最方便的实现;如果需要监听多种触摸事件,那么显然`GestureDetector.OnGestureListener`是理想的选择,并且在`GestureDetector`类内部已经实现了一个简单的监听器实现`GestureDetector.SimpleOnGestureListener`,这个类没有实现任何功能,需要子类覆盖相应的方法来响应事件回调;如果要实现长按拖拽呢,显然以上两个类是没有办法满足要求的,因此,扩展`View.OnTouchListener`类是个不错的选择,在文章最后,会介绍如何扩展来实现长按拖拽事件.

# Android Drag Event

拖拽事件和长按事件一样,是直接得到`View`类支持的.在`View.ListenerInfo`类中,定义了`View.OnDragListener`监听器,不过需要配合上边的`View.OnLongClickListener`来使用,否则单单有这个监听器是不起作用的.目前,实现拖拽事件的方法有两种:

- 设置`View.OnDragListener`和`View.OnLongClickListener`监听器,在长按事件响应时开始拖拽,通过回调判断拖拽事件
- 通过`View.layout(int,int,int,int)`方法直接修改View的位置

## View.OnDragListener

先来看看`View.OnDragListener`的定义:

```
/**
 * Interface definition for a callback to be invoked when a drag is being dispatched
 * to this view.  The callback will be invoked before the hosting view's own
 * onDrag(event) method.  If the listener wants to fall back to the hosting view's
 * onDrag(event) behavior, it should return 'false' from this callback.
 */
public interface OnDragListener {
    /**
     * Called when a drag event is dispatched to a view. This allows listeners
     * to get a chance to override base View behavior.
     *
     * @param v The View that received the drag event.
     * @param event The {@link android.view.DragEvent} object for the drag event.
     * @return {@code true} if the drag event was handled successfully, or {@code false}
     * if the drag event was not handled. Note that {@code false} will trigger the View
     * to call its {@link #onDragEvent(DragEvent) onDragEvent()} handler.
     */
    boolean onDrag(View v, DragEvent event);
}
```

恩,虽然仅仅是个接口,然而有许多注意事项.这个接口会在屏幕响应拖拽事件时调用,并且会在View类中的`View.onDrag(event)`方法之前调用.如果需要系统继续调用`View.onDrag(event)`方法,那么这个监听器回调应该返回`false`,让事件传递到下一层.

前面说了,仅仅设置`View.OnDragListener`监听器是不够的,因为系统并不会主动去触发这个事件监听,而是通过`View.startDrag(ClipData, DragShadowBuilder, Object, int)`这个方法,这个方法会在View类的顶层根视图`ViewRootImpl`中处理拖拽事件,注意,`ViewRootImpl`并非继承自`View`.下面是一个简单的例子.

### 简单实例

```
package com.amap.mock.activity;

import android.app.Activity;
import android.content.ClipData;
import android.content.ClipDescription;
import android.os.Bundle;
import android.util.Log;
import android.view.DragEvent;
import android.view.View;
import android.widget.ImageView;
import android.widget.RelativeLayout;

import com.amap.mock.R;

/**
 * Created by xingli on 9/9/15.
 *
 * An example of performing drag event.
 */
public class DragActivity extends Activity implements View.OnDragListener, View.OnLongClickListener {

    private static final String TAG = DragActivity.class.getSimpleName();
    private ImageView mIvLogo;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test);
        mIvLogo = (ImageView) findViewById(R.id.iv_logo);
        mIvLogo.setClickable(true);
        mIvLogo.setOnLongClickListener(this);
        mIvLogo.setOnDragListener(this);
    }

    @Override
    public boolean onDrag(View v, DragEvent event) {
        RelativeLayout.LayoutParams layoutParams = (RelativeLayout.LayoutParams) v.getLayoutParams();
        switch (event.getAction()) {
            case DragEvent.ACTION_DRAG_STARTED:
                Log.d(TAG, "Action is DragEvent.ACTION_DRAG_STARTED");
                // Do nothing
                break;
            case DragEvent.ACTION_DRAG_ENTERED:
                Log.d(TAG, "Action is DragEvent.ACTION_DRAG_ENTERED");
                break;
            case DragEvent.ACTION_DRAG_EXITED:
                Log.d(TAG, "Action is DragEvent.ACTION_DRAG_EXITED");
                break;
            case DragEvent.ACTION_DRAG_LOCATION:
                Log.d(TAG, "Action is DragEvent.ACTION_DRAG_LOCATION");
                break;
            case DragEvent.ACTION_DRAG_ENDED:
                Log.d(TAG, "Action is DragEvent.ACTION_DRAG_ENDED");
                // Do nothing
                break;
            case DragEvent.ACTION_DROP:
                Log.d(TAG, "ACTION_DROP event");
                // Do nothing
                break;
            default:
                break;
        }
        return true;
    }

    @Override
    public boolean onLongClick(View v) {
        ClipData.Item item = new ClipData.Item("");
        String[] mimeTypes = { ClipDescription.MIMETYPE_TEXT_PLAIN };
        ClipData dragData = new ClipData("", mimeTypes, item);
        // Instantiates the drag shadow builder.
        View.DragShadowBuilder myShadow = new View.DragShadowBuilder(v);
        // Starts the drag
        v.startDrag(dragData, // the data to be dragged
            myShadow, // the drag shadow builder
            null, // no need to use local data
            0 // flags (not currently used, set to 0)
        );
        return true;
    }
}

```

首先需要监听长按事件,然后在触发长按事件后,便可以开始拖动了.拖动的时候会回调`View.onDrag()`方法.其中,在`DragEvent`中定义了几个动作,表示拖动过程:

- `DragEvent.ACTION_DRAG_STARTED`

调用`View.startDrag()`并获得拖动的阴影后进入这个阶段

- `DragEvent.ACTION_DRAG_ENTERED`

系统会把带有这个类型的拖拽事件发送给当前布局中所有的View对象的拖拽事件监听器,如果要继续接收拖拽事件,包括可能的放下事件,View对象的拖拽事件监听器必须返回true.

- `DragEvent.ACTION_DRAG_LOCATION`

当接收到`ACTION_DRAG_ENTERED`事件,并且拖动的影子与原来的View还有重叠的区域时,进入这个状态,只要还在拖动并且符合要求,则这个状态是会被调用多次的.

- `DragEvent.ACTION_DRAG_EXITED`

当接收到`ACTION_DRAG_ENTERED`事件及至少一次`ACTION_DRAG_LOCATION`事件,并且拖动的影子与原来的View没有重叠的区域,即影子与View分离时,进入这个状态,此状态只会在不重叠的一瞬间调用一次.

- `DragEvent.ACTION_DROP`

当用户在一个View对象之上释放了拖拽影子，这个对象的拖拽事件监听器就会收到这种操作类型。如果这个监听器在响应`ACTION_DRAG_STARTED`拖拽事件中返回了true，那么这种操作类型只会发送给一个View对象。如果用户在没有被注册监听器的View对象上释放了拖拽影子，或者用户没有在当前布局的任何部分释放操作影子，这个操作类型就不会被发送。如果View对象成功的处理放下事件，监听器要返回true，否则应该返回false。

- `DragEvent.ACTION_DRAG_ENDED`

当系统结束拖拽操作时，View对象拖拽监听器会接收这种事件操作类型。这种操作类型之前不一定是`ACTION_DROP`事件。如果系统发送了一个`ACTION_DROP`事件，那么接收`ACTION_DRAG_ENDED`操作类型不意味着放下操作成功了。监听器必须调用getResult()方法来获得响应`ACTION_DROP`事件中的返回值。如果`ACTION_DROP`事件没有被发送，那么getResult()会返回false。

## View.layout(int,int,int,int)

上面的方法有个致命的弱点,那就是图标没办法放到指定拖动的点,而只能实现拖动的效果.因为Android系统在设计的时候,`View.OnDragListener`并不是用来进行图标拖动的,而是文字的复制粘贴,我们只是强行地将它作用于图标的拖拽.但上面这种方案也是可以解决这个问题的,那就是先移除原来的图标,然后再在新的位置重绘图标,不过这略显麻烦了,对于内存吃紧的Android系统来说,这无疑是雪上加霜.

下面我们通过重新布局图标的Layout来实现拖动效果.要实现这种效果,就要用到上面介绍的长按事件中的第三种方案,采用设置`View.OnTouchListener`监听器来监听触摸事件,并在`MotionEvent.ACTION_MOVE`中处理拖动.

对于上面的`LongPressHandler`类,还需要做小小的修改,因为上面这个类触发了一次长按回调后,就顺便移除了这个回调,后面的触摸事件就接收不了监听了.解决方案也很简单,在下面这段代码中,把移除监听注销掉就可以了.

```
case MotionEvent.ACTION_MOVE:
    mTouchEndPoint.x = event.getRawX();
    mTouchEndPoint.y = event.getRawY();
    // If user is pressing and dragging, then we make a callback.
    if (mLongPressThread.mLongPressing) {
        // 注销下面这行,实现长期监听.
        // resetLongPressEvent();
        if (listener != null) {
            return listener.onLongPressed(event);
        }
        break;
    }
```

然后,我们在主类中调用这个方法的回调,在回调里进行图标的拖拽.

```
package com.amap.mock.activity;

import android.app.Activity;
import android.os.Bundle;
import android.view.Gravity;
import android.view.MotionEvent;
import android.widget.FrameLayout;
import android.widget.ImageView;

import com.amap.mock.R;

/**
 * Created by xingli on 9/9/15.
 *
 * An example of performing long press and drag event.
 */
public class DragActivity extends Activity {

    private static final String TAG = DragActivity.class.getSimpleName();
    private ImageView mIvLogo;
    private LongPressHandler longPressHandler;
    private int statusBarHeight;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test);
        mIvLogo = (ImageView) findViewById(R.id.iv_logo);
        mIvLogo.setClickable(true);
        setDragEnable(true);
    }

    public void setDragEnable(boolean enable) {
        if (enable) {
            if (longPressHandler == null) {
                longPressHandler = new LongPressHandler(mIvLogo);
            }
            longPressHandler.setOnLongPressListener(new LongPressHandler.OnLongPressListener() {
                @Override
                public boolean onLongPressed(MotionEvent event) {
                    FrameLayout.LayoutParams params = (FrameLayout.LayoutParams) mIvLogo.getLayoutParams();
                    params.gravity = Gravity.TOP | Gravity.LEFT;
                    switch (event.getAction()) {
                        case MotionEvent.ACTION_MOVE:
                            int x = (int) event.getRawX();
                            int y = (int) event.getRawY();
                            int width = mIvLogo.getMeasuredWidth();
                            int height = mIvLogo.getMeasuredHeight();
                            int l = x - width / 2;
                            int r = x + width / 2;
                            int t = y - height / 2 - getStatusBarHeight();
                            int b = y + height / 2 - getStatusBarHeight();
                            params.leftMargin = l;
                            params.topMargin = t;
                            mIvLogo.layout(l, t, r, b);
                            return true;
                    }
                    return false;
                }
            });
        } else {
            if (longPressHandler != null) {
                longPressHandler.setOnLongPressListener(null);
                longPressHandler = null;
            }
        }
    }

    /**
     * Get the status bar height.
     * 
     * @return the height
     */
    public int getStatusBarHeight() {
        if (statusBarHeight == 0) {
            int resourceId = getResources().getIdentifier("status_bar_height", "dimen", "android");
            if (resourceId > 0) {
                statusBarHeight = getResources().getDimensionPixelSize(resourceId);
            }
        }
        return statusBarHeight;
    }
}

```

其中,Activity的布局:

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical">

    <ImageView
        android:id="@+id/iv_logo"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/ic_launcher" />

</FrameLayout>
```

这段代码的关键思想在于,在拖动时,动态计算当前位置的坐标,然后调用`View.layout()`方法对这个View重新布局.有几个需要注意的地方:

- 记得算上动态栏的高度,可以通过`getResources().getDimensionPixelSize(resourceId);`得到某个资源的像素值;
- 需要计算图像的中心点,否则移动的距离以你图标的左上角来计算;
- 在上面的源码中,我还做了一件事,就是获取图标的布局,重新设置Margin值.其实如果你除了拖动以外不进行别的操作的话,没必要进行这样的设置.但如果你需要重绘这个按钮的话,那么重绘的时候是按照旧的LayoutParams来绘制的,这样会造成图标又回到了原来的地方.我在设置FloatingActionButton的时候就遇到了这样的问题,具体代码可参考[Github源码](https://github.com/iluhcm/CircularFloatingActionMenu/blob/master/src/main/java/com/oguzdev/circularfloatingactionmenu/library/FloatingActionMenu.java).
- 如果不想长按拖动,而是直接拖动,那么修改长按触发阈值`LONG_PRESS_TIME_THRESHOLD`,或者通过`public LongPressHandler(View view, long holdTime)`构造函数来实例化`LongPressHandler`就可以了.

## 触摸事件小结

经过上面的分析,我们通过两种方式实现了触摸事件的实现及事件回调处理,分别是View.OnDragListener接口和View.layout()方法.

实际上触摸事件是和长按事件分不开的,只是触发时间的长短阈值设置不同罢了.在第一种方法中,通过调用`View.startDrag()`方法触发拖拽事件,通过设置`View.OnDragListener`设置事件回调,便可以在回调中处理拖拽事件.但是这种方法的应用场景并不是图标拖拽,而是文字的复制粘贴,原始的视图是不会移动的.一般我们会通过覆盖`View.onTouchEvent()`或者设置`View.OnTouchListener`监听器来监听滑动事件,并在`MotionEvent.ACTION_MOVE`状态中处理拖拽问题,这便是第二种方案的思想.

# Source Code

长按拖拽的源码在[Github](https://github.com/iluhcm/CircularFloatingActionMenu)上,欢迎star&fork:)

# Reference

<http://grepcode.com/file/repo1.maven.org/maven2/org.robolectric/android-all/5.0.0_r2-robolectric-1/android/view/View.java>
<http://www.ablanxue.com/prone_4213_1.html>
<http://www.yiibai.com/android/android_drag_and_drop.html>
<http://developer.android.com/guide/topics/ui/drag-drop.html>
<http://www.jcodecraeer.com/a/anzhuokaifa/developer/2013/0311/1003.html>