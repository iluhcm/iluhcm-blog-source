title: Android通知栏介绍与适配总结
date: 2017-03-12 14:40:29
tags: [Android, Notification, Icon, Emoji]
categories: Android
---

由于历史原因，Android在发布之初对通知栏Notification的设计相当简单，而如今面对各式各样的通知栏玩法，谷歌也不得不对其进行更新迭代调整，增加新功能的同时，也在不断地改变样式，试图迎合更多人的口味。本文总结了Android通知栏的版本迭代过程，在通知栏开发过程中所遇到的各种各样的坑，以及一些解决技巧，特别的，对于大众期盼的Android 7.0的到来，通知栏又会发生怎样的改变呢？接下来一一进行介绍。

# Android通知栏发展历史

首先来看一张各个Android版本通知栏消息的全家福。

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_changelog.jpg" alt="Drawing" style="width: 1000px;"/>

(点击查看大图)

Android通知栏从最初的Android1.1系统一直到如今的7.X版本，发生了翻天覆地的变化。从图中可以看出，1.X-2.2版本的通知栏采用了白色背景和黑色字体；2.3-4.X版本，默认背景变成了黑色，而主标题采用白色字体，内容为灰色字体。从Android5.0开始，又更改为白色背景和黑色字体。当然，这只是原生的Android系统通知栏默认颜色，许多厂商对每个Android的版本都尝试了各式各样的修改，在此不一一介绍。

下面分别介绍每个版本的更新和修改记录。

## Android 1.X 修改记录[^1]

Android 1.X版本也就是第一个Android诞生的版本。从Android1.1版本开始，提供基本的通知栏消息功能，包含小图标、主标题、副标题和时间这四个元素。右上角有一个清除通知栏消息的按钮。需要说明的是，Android从一开始就提供了清除通知栏消息的功能并且保留至今，而iOS到现在都没有提供清除按钮。

## Android 2.X 修改记录[^2]

Android 2.X版本的通知栏消息功能上并未发生变化，右上角的“clear notifications”缩减为了“clear”。2.2版本以前沿用了1.5的通知栏样式，从2.3版本开始重新设计，改成了暗色背景。

## Android 3.X 修改记录[^3]

Android 3.X版本是专为Pad而设计的系统。通知栏消息带来了一些新的功能。

* 非永久的通知栏消息的右边增加了“X”按钮，点击后该条通知可以立即清除。
* 增加了RemoteControlClient，即远程控制媒体应用的功能。
* 增加了LargeIcon，可以使用大图展示通知栏消息。

## Android 4.1 修改记录[^4] [^5]

Android 4.1版本的通知栏在3.X版本的基础上进行了大量修改。增加了不少新功能。

* 增加了Style
* 增加了通知栏按钮
* 支持通知栏展示的优先级配置
* 通知栏背景改为黑色透明

### 通知栏样式

Android 4.1通知栏最大的变化就是增加了丰富多样的Style样式。通过设置样式，可以展示更大区域的通知消息，如展示大图和多行文字，也可以展示类似邮箱收发信的样式，同时支持自定义按钮并增加点击事件。但需要注意的是，只有最顶部的那条通知栏消息可以默认展示Style样式，其他消息默认是以普通样式展示。Style可以通过`Notification.Builder.setStyle(Style)`进行设置。具体支持的样式有：

#### Notification.BigPictureStyle

大图样式，即除了普通的通知栏消息内容外，可以在通知栏消息下方展示一张大图，最大高度支持256dp。

#### Notification.BigTextStyle

多行文字样式，可以支持多行文字的展示。经测试，在不同手机上能够支持的行数不一样，测试过的机子，最大支持12行。

#### Notification.InboxStyle

收件箱样式。支持展示具有一串消息内容的会话样式，适用于短信、邮件、IM等。

### 通知栏按钮

通知栏消息不管是普通样式还是Style样式，都支持两个按钮同时出现在一条通知栏消息的底部，通过这两个按钮，可以自定义一系列动作，包括回复信息和邮件，点赞等。通过`Notification.Builder.addAction(Action)`添加按钮。

### 通知栏优先级

Android 4.1通知栏增加了优先级的配置，优先级高的消息可以展示在最上方。谷歌设计优先级的初衷是根据不同的优先级来防止用户整天被各种莫名其妙的通知栏消息骚扰，重要的通知则应该适当提高优先级，使得用户可以快速地看到并回应，不重要的通知则降低优先级，防止用户被打扰。优先级一共有5个级别，分别是：
	
	// 默认优先级
    public static final int PRIORITY_DEFAULT = 0;
    // 低优先级
    public static final int PRIORITY_LOW = -1;
    // 最低优先级
    public static final int PRIORITY_MIN = -2;
    // 高优先级
    public static final int PRIORITY_HIGH = 1;
    // 最高优先级
    public static final int PRIORITY_MAX = 2;


## Android 4.3 修改记录[^6]

Android 4.3通知栏没有发生大的变化。主要增加了两个小功能。

* 增加了`Notification Access`Api，允许可穿戴设备远程控制通知栏消息。
* 增加了`NotificationListenerService`，允许接收到系统通知栏列表的变化

## Android 5.X 修改记录[^7]

Android 5.X系统相较于以前的版本，可以说是一个真正可以和iOS抗衡的系统。材料设计给Android系统注入了新的活力，相应的通知栏消息也相较于上一个版本进行了改版。所发生的变化有：

* 通知栏修改为白色背景，暗色字体，以适应材料设计风格。
* **系统会忽略所有`non-alpha`通道的图标，包括按钮图标和主图标**。
* 可以通过`setColor()`方法在图标后设置一个背景色。
* 通知消息的声音将通过`STREAM_RING`或者`STREAM_NOTIFICATION`控制，以前是通过`STREAM_MUSIC`控制。
* 锁屏状态下，可以控制通知栏消息的隐私程度。
* 移除了`RemoteControlClient`，更改为`NotificationCompat.MediaStyle`实现。
* 增加了`Heads-up`通知，即通过状态栏浮动窗口展示通知消息。

## Android 6.X 修改记录[^8]

* 移除了`Notification.setLatestEventInfo()`方法，通过持有`Notification.Builder`，然后使用`build()`方法可以更新同一个通知栏实例。
* 允许用户控制应用通知的优先级。
* 加入了免打扰模式（`Do Not Disturb`）。
* 增加了`getActiveNotifications()`方法获取当前展示的通知消息。

## Android 7.X 修改记录

* 通知栏样式全面改版，小图标在左上角，大图标在右边，小图标、App应用名、副标题、数量和时间在第一行，第二行是主标题，第三行是内容。
* 增加了`Notification.DecoratedCustomViewStyle()`和`Notification.DecoratedMediaCustomViewStyle()`，帮助更好的装饰带有`RemoteViews`的通知栏消息。
* 需要动态设置`Builder.setShowWhen(true)`才会显示时间。
* 支持`Action`的直接回复，通过`RemoteInput`实现，且回复的消息内容支持立即添加到通知栏。
* 支持通知消息组，相似的消息在达到一定数量后会按照消息组来显示。
* 增加了`NotificationManager.areNotificationsEnabled`告知应用是否开启了通知权限。

# Android通知栏踩坑与填坑指南

## 魅族5.X手机，大图显示问题
	
### 问题详情

Flyme系统对原生Android源码做了修改，采用`BigPictureStyle`方式显示大图通知栏的时候，消息与大图重合了，如下图。

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/魅族MX5.jpg" alt="Drawing" style="width: 300px;"/>

### 解决方案

首先说一下为什么会有解决方案。展示大图这个功能开发完成后，拿去给产品演示。碰巧产品的机型就是一魅族手机T_T，结果当然是不能接受的，然后又一个巧合的事情出现了，那就是产品的手机里，京东App推了一条带大图的广告，他们居然能够解决这个问题！于是，我开始研究解决方案。

首先，通过`BigPictureStyle`来实现大图功能肯定是走不通的，因为事实就摆着行不通的嘛。京东的App肯定是通过RemoteViews来实现的。于是，开始走弯路，尝试通过RemoteViews来展示大图。但是谷歌规定，自定义布局展示的通知栏消息最大高度是`64dp`。那么，京东的App是怎么实现的？在尝试了各种方法以后，最后又是通过投机取巧的方式解决了问题：

```
private void showBigPictureNotificationWithMZ(Context context) {
    NotificationManager notificationManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
    Notification.Builder builder = new Notification.Builder(context);
    Notification notification = generateNotification(builder);
    notification.bigContentView = mRemoteViews;
    notificationManager.notify(notifyId, notification);
}
```

**需要先生成`Notification`的实例，然后手动给`notification.bigContentView`赋值，再`notify`，就可以了**

## 顶部状态栏(StatusBar)小图标显示异常

### 问题详情

当通知来的时候，如果不在通知栏浏览，会在顶部状态栏出现一个向上翻滚动画的通知消息，这条通知消息左边是一个小图标。部分系统这个小图标显示异常，是一个纯灰色的正方形，如下图。

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/small_icon_exception.png" alt="Drawing" style="width: 300px;"/>

### <a name="小图标显示异常"></a>解决方案

首先产生灰色图标的原因就是5.0系统引入了材料设计，谷歌强制使用**带有alpha通道的图标，并且RGB的alpha值必须是0**(实测不为0也是可以的，但系统会忽略所有RGB值)。因此，使用JPG的图片是不行的，最好的代替方案就是一张背景透明的PNG图片。

## Android 7.X机型，通知栏小图标显示成灰色

### 问题详情

这个问题跟第二个有点类似，在7.0系统及以上，有部分应用的小图标是灰色的，大图可以正常显示。碰巧的是，显示异常的小图标，颜色都是灰色的。

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_small_icon_exception_android_7.0.png" alt="Drawing" style="width: 300px;"/>

### 解决方案

与[小图标显示异常解决方案](#小图标显示异常)类似，将小图标替换为透明背景的PNG图片。

## <a name="RemoteViews显示异常"></a>RemoteViews显示异常


### 问题详情

由于系统提供的通知栏消息类型有时候不能满足要求，部分通知栏消息采用自定义RemoteViews来实现。采用RemoteViews，特别是手动生成Bitmap然后直接传给一个自定义Layout，再通过setContentView方式设置通知栏消息时，会存在各种各样的坑。

Android通知栏的背景色有几种情况，白色、暗色、暗色透明和黑色。如果生成的Bitmap带背景色，这个背景色就很难选择。如果选择黑色背景，那么在白色通知栏的机型上就很难看。因此不能完全在各个系统上面完美展示出来。如果不带背景色，那么字体颜色也面临同样的困惑。试想，如果在白色的背景上显示白色的文字，用户看到白茫茫一片，是什么感受？

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_remoteviews_exception.png" alt="Drawing" style="width: 300px;"/>

另一方面，大部分厂商对原生的Android系统都会有各种各样的改造，通知栏的样式也不例外。如果按照原生的样式来设计，那么在大部分国内厂商的机子上显示都和正常的普通通知栏消息不一样。例如华为6.0系统的机子，原生系统的时间线在右上角，华为的在左边，这样会给用户带来错觉。

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/华为荣耀7.jpg" alt="Drawing" style="width: 300px;"/>

### 解决方案

详见[RemoteViews适配](#RemoteViews适配)一节。

## 大尺寸小图标在部分机型上显示不正确

### 问题详情

这个问题主要在部分机型的4.X系统上遇见，小图标大小没有按照`24dp`裁剪，而是采用了桌面图标一样的大小`96dp`。具体适配不正常的机型有HTC Desire 820、Lenovo A320T。

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/Desire820.jpg" alt="Drawing" style="width: 300px;"/>

### 解决方案

按照标准来，小图标大小为`24dp`，大图标为桌面icon图标大小`96dp`。具体可参考[这里](http://iconhandbook.co.uk/reference/chart/android/)[^14]

## 部分机型不支持Style

具体机型见下图以及后面统计的表格。顺便提下，小米是其中之一，不知道他们为什么不支持额外的这些Style。

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_big_picture_style_test.jpg" alt="Drawing" style="width: 1000px;"/>

(点击查看大图)

## 通知栏更新频率

### 问题详情

每个应用基本都有自更新的逻辑，App开机的时候提示用户升级，点击升级按钮后在Notification出现一个下载带进度条的通知。应用一般是在开启一个工作线程在后台下载，然后在下载的过程中通过回调更新通知栏中的进度条。我们知道，下载进度的快慢是不可控的，如果每次下载中的回调都去更新通知栏，那么可能几百毫秒、几十毫秒、甚至几毫秒就更新一次通知栏，应用可能就会ANR，甚至崩溃。

### 解决方案

控制通知栏更新频率，一般控制在0.5s或者1s就可以了。在某一个更新时间间隔内下载的进度回调直接丢弃，需要注意的是下载完成的回调，需要实时回调通知栏消息显示下载完成。

## 恶心的后台通知和“守护”通知

### 问题详情

这个坑我不愿多介绍，只说结果。但凡存在后台通知或者“守护”通知的应用，在7.0系统以后都会原形毕露。还没有适配7.0的应用，可长点心儿吧~

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_background_notification.png" alt="Drawing" style="width: 300px;"/> <img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_daemon_notification.png" alt="Drawing" style="width: 300px;"/>

### 解决方案

请弃坑。

## 小米推送SDK接入问题

### 问题详情

为了提升推送到达，考拉接入了小米推送的SDK。小米推送分为通知栏消息和透传消息，通知栏消息属于系统级推送，在MIUI的机子上可以在进程被杀死的情况下也能收到应用推送。然而有个问题，小米认为应用在前台时，不会回调任何方法；小米认为应用在后台的时候，收到通知栏消息的同时，会回调`onNotificationMessageArrived`方法。这时候就要小心翼翼地处理这条消息了。因为如果你的应用前后台判断逻辑和小米的不一样，那么就有可能小米帮你发了一条通知栏消息，你自己又发了一遍，造成通知栏消息的重复发送(这个坑考拉踩过T_T)。另一方面，在7.0系统的机子上，主标题和小图标的颜色是可以改变的，目前小米推送SDK没有开放这个接口供调用方定制。

### 解决方案

目前只能解决第一个问题——前后台判断的问题。应用是否在后台可以根据以下代码进行判断。在Android 5.0以上，可以通过`ActivityManager.RunningAppProcessInfo`判断，Android 5.0及以下版本通过`ActivityManager.RunningTaskInfo`判断。经测试，这个方案在Android 4.4以上结果是可以完全匹配的。

```
public static boolean isAppInBackgroundInternal(Context context) {
    ActivityManager manager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    if (Build.VERSION.SDK_INT > Build.VERSION_CODES.LOLLIPOP) {
        List<ActivityManager.RunningAppProcessInfo> runningProcesses = manager.getRunningAppProcesses();
        if (!ListUtils.isEmpty(runningProcesses)) {
            for (ActivityManager.RunningAppProcessInfo runningProcess : runningProcesses) {
                if (runningProcess.importance == ActivityManager.RunningAppProcessInfo.IMPORTANCE_FOREGROUND) {
                    return false;
                }
            }
        }
    } else {
        List<ActivityManager.RunningTaskInfo> task = manager.getRunningTasks(1);
        if (!ListUtils.isEmpty(task)) {
            ComponentName info = task.get(0).topActivity;
            if (null != info) {
                return !isKaolaProcess(info.getPackageName());
            }
        }
    }
    return true;
}

```

# Android通知栏适配

## <a name="RemoteViews适配"></a>RemoteViews适配

由于系统自带的通知栏消息样式不能完全满足产品们脑洞大开的需求，有时候我们需要自定义布局样式展示通知栏消息。Android系统可以将自定义布局通过`setContent`(7.X系统推荐使用`setCustomContentView`)设置到`Notification.Builder`中，来实现样式的更变。`setContent`方法需要传入一个`RemoteViews`对象，它是一个普通的数据类型，不是View，作用是供其他进程展示视图。RemoteViews只支持4种基本的布局[^9]:

* FrameLayout
* LinearLayout
* RelativeLayout
* GridLayout

这些布局下面只支持几种视图控件:

* AnalogClock
* Button
* Chronometer
* ImageButton
* ImageView
* ProgressBar
* TextView
* ViewFlipper
* ListView
* GridView
* StackView
* AdapterViewFlipper

只能通过上述组合生成一个RemoteViews。

### 自定义布局与视图

除了上面提到的布局与控件，有没有办法自定义布局与视图呢？我们知道，任何一个View，都可以生成一个`Bitmap`对象，支持的视图控件里有`ImageView`，可以通过`ImageView.setBitmapResource()`将自定义视图设置到一个`ImageView`中，然后再随便放到一个布局上，就可以实现通知栏消息的任意布局。理想是美好的，但现实是残酷的。使用这种方式自定义的布局，会存在与原生的通知栏消息样式不一致的可能，包括小图标/大图标的大小，字体的大小与颜色，时间的显示方式(不同版本的时间显示位置和样式都不一样)。下面解决一个最关键，也最致命的问题——字体颜色。如果字体颜色和背景颜色一样，那这条通知栏消息就没法看了，如[RemoteViews显示异常](#RemoteViews显示异常)一节介绍的一样。

解决字体颜色和背景颜色一样的问题有三种解决方案，分别是：

* 背景色固定不透明，字体颜色与背景色形成反差。（360和京东的做法）
* 背景色透明，字体颜色采用系统原生的`notification_style`。
* 背景色透明，通过特殊方式拿到通知栏字体颜色和字体大小。

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_jd.png" alt="Drawing" style="width: 300px;"/>


其中，第一种方案简单，能够兼容所有厂商机型。例如京东固定背景色为黑色，字体为红色。这种方式的唯一缺陷是样式上不能与普通通知栏消息重合，在白色背景的通知栏上极为显眼。第二种方式，通过阅读源码可知，系统的通知栏标题和内容采用的颜色分别是`@android:color/primary_text_dark`和`@android:color/secondary_text_dark`，但踩过坑之后发现并非所有的机型默认都是这两个颜色，有可能获取不到值。因此这种方案只能作为参考，不能用于实际环境中。最后详细介绍一下第三种方式。

### Android默认字体颜色获取

这种方案有一点投机取巧，是网上寻找代替方案时在简书上找到的，作者是[hackware](http://www.jianshu.com/p/426d85f34561)。思路就是通过`Notification.Builder`生成一条空的`Notification`，但不调用`notify()`方法，然后通过这条`Notification`想办法获取里面的布局元素，通过遍历，就能拿到对应的字体和颜色了。具体看代码：

```
private static final String NOTIFICATION_TITLE = "notification_title";
public static final int INVALID_COLOR = -1; // 无效颜色
private static int notificationTitleColor = INVALID_COLOR; // 获取到的颜色缓存

/**
 * 获取系统通知栏主标题颜色，根据Activity继承自AppCompatActivity或FragmentActivity采取不同策略。
 *
 * @param context 上下文环境
 * @return 系统主标题颜色
 */
public static int getNotificationColor(Context context) {
    try {
        if (notificationTitleColor == INVALID_COLOR) {
            if (context instanceof AppCompatActivity) {
                notificationTitleColor = getNotificationColorCompat(context);
            } else {
                notificationTitleColor = getNotificationColorInternal(context);
            }
        }
    } catch (Exception ignored) {
    }
    return notificationTitleColor;
}


/**
 * 通过一个空的Notification拿到Notification.contentView，通过{@link RemoteViews#apply(Context, ViewGroup)}方法返回通知栏消息根布局实例。
 *
 * @param context 上下文
 * @return 系统主标题颜色
 */
private static int getNotificationColorInternal(Context context) {
    Notification.Builder builder = new Notification.Builder(context);
    builder.setContentTitle(NOTIFICATION_TITLE);
    Notification notification = builder.build();
    try {
        ViewGroup root = (ViewGroup) notification.contentView.apply(context, new FrameLayout(context));
        TextView titleView = (TextView) root.findViewById(android.R.id.title);
        if (null == titleView) {
            iteratorView(root, new Filter() {
                @Override
                public void filter(View view) {
                    if (view instanceof TextView) {
                        TextView textView = (TextView) view;
                        if (NOTIFICATION_TITLE.equals(textView.getText().toString())) {
                            notificationTitleColor = textView.getCurrentTextColor();
                        }
                    }
                }
            });
            return notificationTitleColor;
        } else {
            return titleView.getCurrentTextColor();
        }
    } catch (Exception e) {
        DebugLog.e(e.getMessage());
        return getNotificationColorCompat(context);
    }
}

/**
 * 使用getNotificationColorInternal()方法，Activity不能继承自AppCompatActivity（实测5.0以下机型可以，5.0及以上机型不行），
 * 大致的原因是默认通知布局文件中的ImageView（largeIcon和smallIcon）被替换成了AppCompatImageView，
 * 而在5.0及以上系统中，AppCompatImageView的setBackgroundResource(int)未被标记为RemotableViewMethod，导致apply时抛异常。
 *
 * @param context 上下文
 * @return 系统主标题颜色
 */
private static int getNotificationColorCompat(Context context) {
    try {
        Notification.Builder builder = new Notification.Builder(context);
        Notification notification = builder.build();
        int layoutId = notification.contentView.getLayoutId();
        ViewGroup root = (ViewGroup) LayoutInflater.from(context).inflate(layoutId, null);
        TextView titleView = (TextView) root.findViewById(android.R.id.title);
        if (null == titleView) {
            return getTitleColorIteratorCompat(root);
        } else {
            return titleView.getCurrentTextColor();
        }
    } catch (Exception e) {
    }
    return INVALID_COLOR;
}

private static void iteratorView(View view, Filter filter) {
    if (view == null || filter == null) {
        return;
    }
    filter.filter(view);
    if (view instanceof ViewGroup) {
        ViewGroup viewGroup = (ViewGroup) view;
        for (int i = 0; i < viewGroup.getChildCount(); i++) {
            View child = viewGroup.getChildAt(i);
            iteratorView(child, filter);
        }
    }
}

private static int getTitleColorIteratorCompat(View view) {
    if (view == null) {
        return INVALID_COLOR;
    }
    List<TextView> textViews = getAllTextViews(view);
    int maxTextSizeIndex = findMaxTextSizeIndex(textViews);
    if (maxTextSizeIndex != Integer.MIN_VALUE) {
        return textViews.get(maxTextSizeIndex).getCurrentTextColor();
    }
    return INVALID_COLOR;
}

private static int findMaxTextSizeIndex(List<TextView> textViews) {
    float max = Integer.MIN_VALUE;
    int maxIndex = Integer.MIN_VALUE;
    int index = 0;
    for (TextView textView : textViews) {
        if (max < textView.getTextSize()) {
        	// 找到字号最大的字体，默认把它设置为主标题字号大小
            max = textView.getTextSize();
            maxIndex = index;
        }
        index++;
    }
    return maxIndex;
}

/**
 * 实现遍历View树中的TextView，返回包含TextView的集合。
 *
 * @param root 根节点
 * @return 包含TextView的集合
 */
private static List<TextView> getAllTextViews(View root) {
    final List<TextView> textViews = new ArrayList<>();
    iteratorView(root, new Filter() {
        @Override
        public void filter(View view) {
            if (view instanceof TextView) {
                textViews.add((TextView) view);
            }
        }
    });
    return textViews;
}
    
private interface Filter {
    void filter(View view);
}
```

使用这种方法，我们统计并测试了大厂商的部分机型，得到如下表格：

| 厂商  | 机型  | 系统通知标题颜色 | 背景主题 | 是否支持大图/多行文字 |系统版本 |标题大小 |能否获取App推送是否打开 | 备注|
|:-------------|:---------------| :-------------| :-------------| :-------------| :-------------| :-------------| :-------------| :-------------|
|华为|	荣耀V8|	未知|	暗色|	支持| 6.0|	14dp/19dp| 能 |通知栏显示需要授权|
| 华为 | P8 | -637534209 | 暗色透明 | 支持 | 5.0.1 | |能 | |
| 华为 | 荣耀4A | -637534209 | 黑色，透明度不高 | 支持 | 5.0.1 | | 能| |
| 华为 | 荣耀7 | -637534209 | 黑色，透明度不高 | 支持 | 5.0.2 | |能 | |
| 华为 | MATE8 | -637534209 | 黑色 | 支持/支持12 | 6 | 14dp/16dp+ | 能| |
| 华为 | G7plus | -637534209 | 黑色 | 支持 | 5.1 | 14dp |能 | |
| 华为 | 荣耀4X | -637534209 | 黑色，透明度不高 | 支持 | 5.0.2 | 14dp/16dp+ |能 | |
| 小米 | 3 | -452984832 | 灰色，不透明 | 不支持/不支持 | 4.4.4 |  | 能 | 工作线程获取系统颜色时，会产生java.lang.reflect.InvocationTargetException，带emoji表情时，系统显示颜色和普通推送一样 |
| 小米 | 5s | -452984832 | 白色，不透明 | 不支持 | 6.0.1 | 13.33dp/18dp | 能 | |
| 红米 | NOTE | -1 | 灰色，不透明 | 不支持 | 4.4.4 |  | 能 | 带emoji表情时，系统显示颜色和普通推送一样 |
| 红米 | NOTE3 | -1 | 灰色，透明 | 不支持 | 5.1.1 |  | 能 | 带emoji表情时，系统显示颜色和普通推送一样，但样式与普通推送稍有不同 |
| 三星 | S4 | -16777216 | 白色，不透明 | 支持 | 5.0.1 | 17dp/17dp | 能 | 大图需要下拉才能显示 |
| 三星 | S5 | -16777216 | 白色，不透明 | 支持 | 5 |  | 能 | 大图需要下拉才能显示 |
| 三星 | S6 | -14342875 | 白色，不透明 | 支持 | 6.0.1 | 17dp/17dp | 能 | 大图不需要可以显示，系统版本6.0.1 |
| 三星 | Note2 | -1644826 | 黑色 | 支持 | | | 能| |
| 三星 | S6+ | -14342875 | 白色，不透明 | 支持/支持 | 5.1.1 | 16dp |能 | |
| 三星 | SM-N9100 | -16777216 | 白色，不透明 | 支持 | 17dp/18dp+ |  | 能 | 5.0.1 |
| 魅族 | MX5 | -1 | 灰色，不透明 | 支持/支持6 | 5.1 | 18dp/21dp | 能 | 小通知和大图重叠；带emoji表情时，系统显示内容颜色为Android自带颜色，与魅族的系统推送颜色不太一样 |
| 魅族 | MX3 |  |  |  | 4.3 |  | 不能 | 系统是联通定制的，版本也比较低，估计还不支持该API。 |
| 魅蓝 | NOTE2 | -1 | 灰色，透明度不高 | 支持 | 5.1 | 18dp/21dp | 能 | 小通知和大图重叠；带emoji表情时，统显示内容颜色为Android自带颜色，与魅族的系统推送颜色不太一样 |
| HTC | Desire820 | -1 | 黑色，不透明 | 支持 | 4.4.4 |  | 未测试 | 有不可清除的系统消息存在时，大图默认不显示；普通推送小icon图标显示不正确；自定义emoji表情跟MX5类似；工作线程获取系统颜色时，会产生java.lang.reflect.InvocationTargetException |
| HTC | M9w | -570425344 | 白色 | 支持 | | |未测试 | |
| OPPO | R7plus | -1 | 黑色，透明度不高 | 不支持 | 5 | 16dp/16dp |能  | 带emoji表情时，系统显示颜色和普通推送一样 |
| OPPO | R3 |  | 黑色，透明度不高 | 不支持 | 4.3 | 18dp/18dp |未测试 | |
| 一加 | ONE | -570425344 | 白色 | 支持 | 5.1.1 | 16dp/16dp | 能| |
| 一加 | One plus 3 |  | 白色 | 支持/支持13 | 7.0 | |能 | |
| LG | Nexus5 | -570425344 | 白色 | 支持/支持12 | 6.0.1 | 16dp/16dp | 能 | |
| 乐视 | MAX | -1 | 暗色，透明 | 支持 | 6.0.1 | 16dp/16dp |能 | |
| ViVO | X5L | -1 | 暗色 | 支持 | 4.4.2 | |未测试 | |
| ViVO | X5M | -1 | 暗色 | 支持 | 5.0.2 | |能 | |
| 锤子 | T1 |  | 黑色 | 不支持 | 4.4.2 | | 未测试| |
| 金立 | S5.1 | -1 | 黑色，不透明 | 支持 | 4.3 |  | 未测试 | 工作线程获取系统颜色时，会产生java.lang.reflect.InvocationTargetException |
| 联想 | K910 | -1 | 暗色 | 支持 | 4.2.2 | | 不能| |
| 联想 | A320T | -1 | 黑色，不透明 | 支持 | 4.4.4 | | 未测试| |

可以看到，通知栏存在各式各样的背景色，字体大小和颜色也不尽相同。通过上述方法，有一部分机型是拿不到系统通知栏颜色的，但通过观察可以发现，所有拿不到字体颜色的机型都是暗色或黑色背景***(实测7.0此经验失效)***，因此可以使用白色字体。

### 考拉RemoteViews适配方案

经过上述调研与测试，我们的适配方案如下：

* 获取系统通知标题颜色，如果能够获取到，那么标题、内容和时间的颜色都设置为标题颜色。
* 获取不到的情况下，遍历系统通知里的所有文字，取字号最大的那条文字的颜色作为标题、内容和时间的颜色。
* 以上两个步骤的实现在`getNotificationColor()`方法里。如果还获取不到，那么标题和内容采用Android原生系统提供的，其中标题是`@android:color/primary_text_dark`，内容是`@android:color/secondary_text_dark`。
* 有一点需要说明的是，以上适配只适合在Android 7.0以下系统。Android 7.0+修改了Notification，采用`@android:color/primary_text_dark`和`@android:color/secondary_text_dark`已经获取不到颜色值了，考虑到7.0所采用的通知栏主色调是白色，因此目前暂时的解决方案是遇到7.0的系统采用黑色字体。面对众多厂商的源码修改，目前测试有ZUK的7.0系统为暗色背景，暂时的解决方案是根据机型适配。

## Emoji表情适配[^12]

对于Android emoji表情的适配，我想只有体验过的人才知道这里面有多少坑。我试图通过谷歌了解Android在每一个版本对应的emoji表情的支持情况，最终发现没有一篇文章或一个网页能够完全列出emoji表情在Android上的修改历史。于是我只能自己动手，构建一张Android emoji表情支持版本对比的表格。

| Android系统版本 | emoji版本 | Unicode版本 | Unicode emoji发布日期 | Android支持版本日期 | emoji表情数 | 新增表情链接/更新日志  | 备注|
|:-------------|:---------------| :-------------| :-------------| :-------------| :-------------| :-------------| :-------------|
| 7.1 | 4 | 9.0 | 2016.07.21 | 2016.10.20 | 2374 | <http://emojipedia.org/google/android-7.1/new/> <http://blog.emojipedia.org/android-7-1-emoji-changelog/> | 成为首个支持Unicode 9标准的系统!|
| 7.0 | 3 | 9.0 | 2016.07.21 | 2016.08.22 | 1791 | <http://emojipedia.org/google/android-7.0/new/> <http://blog.emojipedia.org/android-7-0-emoji-changelog/> | |
| 6.0.1 | 1 | 8.0 | 2015.07.17 | 2015.12.07 | 1294 | <http://emojipedia.org/google/android-6.0.1/new/> <http://blog.emojipedia.org/android-6-0-1-emoji-changelog/> | |
| 6.0 | 1 | 7.0 | 2014.07.16 | | | | |
| 5.0 | 1 | 6.1 | 2012.02 | 2014.11.03 | 1090 | <http://emojipedia.org/google/android-5.0/new/> <http://blog.emojipedia.org/android-50-emoji-changelog/> | |
| 4.4 | 1 | 6.0 | 2010.1 | 2014.11.01 | 850 | <http://emojipedia.org/google/android-4.4/new/> |首个支持彩色Emoji表情的Android系统|
| 4.3 | 1 | 6.0 | 2010.1 | 2013.07.24 | 717 | <http://emojipedia.org/google/android-4.3/new/>|首个支持Emoji表情的Android系统，但颜色是黑白的|

首先需要说明的是为什么会有Emoji版本和Unicode版本？Emoji实际上可以说是Unicode下的一个子集，Unicode的版本更新，除了Emoji表情发生变化以外，还有许多其他的字符集定义发生变化，Emoji版本是跟随着Unicode版本的更新而逐渐迭代更新的。可以看到，基本上一个Unicode版本对应着一个Emoji版本。目前最新的Unicode版本规划是`Unicode 11.0`[^10]，最新的Emoji版本规划是`Emoji 6.0`[^11]，实际待发布版本是`Unicode 10.0`和`Emoji 5.0`，将在2017年中旬发布。

当然这只是官方Android系统所支持的emoji版本，面对众多的厂商对源码大刀阔斧的修改，结果又是怎样呢？我们拿了33个表情来进行测试，其中大部分表情是`Unicode 6.0`标准，后面几个表情是`Unicode8.0`标准。最终得到了如下结果。

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_emoji_test.jpg" alt="Drawing" style="width: 1000px;"/>
(点击查看大图)

实际测试结果与上面的表格基本匹配。特别表扬一下**魅族**，在5.X系统上就已经支持了`Unicode 8.0`标准！乐视的系统在6.0.1系统上的表现指明支持的是`Unicode 7.0`标准，实际上Android原生已经支持`Unicode 8.0`标准了。

因此，emoji表情的适配其实相对较简单，就是根据不同的系统版本实现不同的支持。当然，如果需要简化那么让只需要让运营配置**Unicode 6版本的emoji表情**就能够适配4.4+版本的系统了！至于4.4以下版本，可以把常用的Emoji表情放到资源文件中，遇到文本中包含Emoji字符时，手动替换成资源文件中的Emoji图片，再通过上述RemoteViews方式来显示。

## Android Nougat+适配

从上面的介绍中，大家可以发现，Android 7.0系统以后通知栏消息改版了。援引官方在`Notifications public deck`中介绍的一张图，

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_7.0_changelog.png" alt="Drawing" style="width: 1000px;"/>

除了基本的样式发生变化，在7.0中也做了部分接口上的修改。其中，我们需要“拥抱变化”的内容有：

### 使用non-alpha图标

在5.0修改记录中，有一条**系统会忽略所有`non-alpha`通道的图标，包括按钮图标和主图标**。这句话是什么意思呢？实际上，Android从5.0系统开始，对于通知栏图标的设计进行了修改。现在Google要求，所有应用程序的通知栏图标，应该只使用alpha图层来进行绘制，而不应该包括RGB图层。通俗点来讲，就是让我们的通知栏图标不要带颜色就可以了[^13]。这也是上面的截图中为什么这么多应用都显示不出icon图标，而是显示成灰色的正方形。原因就是他们用了带背景的图片。

### Notification.Builder.setColor()方法

5.0加入的`Notification.Builder.setColor()`方法，原本是渲染小图标的背景色的，7.0以后，改成了渲染的是通知栏消息第一行的颜色。

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_5.1_setcolor.png" alt="Drawing" style="width: 300px;"/>
<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_7.0_setcolor.png" alt="Drawing" style="width: 300px;"/>

### 7.0系统默认不显示时间

7.0系统以后需要显式调用`Notification.Builder.setShownWhen(true)`才会显示时间，并且格式调整为具体发布时间相差N小时/N天的形式（见上图）。

### RemoteViews样式调整

如果要适配7.0以后的样式，可以使用以下两个包裹的Style，将RemoteViews封装在内容区域。

* Notification.DecoratedCustomViewStyle()
* Notification.DecoratedMediaCustomViewStyle()

<img src="http://7xl6ic.com1.z0.glb.clouddn.com/img/android_notification/notification_7.0_remoteviews.png" alt="Drawing" style="width: 300px;"/>

图中是未包裹与包裹时候的展示效果。可以说适配RemoteViews是非常蛋疼的一件事，应用可以根据需要来选择是否使用`DecoratedCustomViewStyle`。

如果不是必要，建议不要使用RemoteViews。考拉之前是为了兼容在不同手机厂商上展示的emoji表情不一致，以及兼容低版本系统，而在包含emoji表情的消息推送中使用了RemoteViews。经过测试，前者的兼容完全没有必要，在下一版的日常版本发布中会移除RemoteViews相关内容，改用原生通知栏消息实现。

# Android O通知栏新特性一览

就在笔者即将发布这篇文章的时候，Android O系统发布了预览版！由上面的讨论可知，几乎每个Android版本都修改了Notification，相信Android O也不例外。笔者迫不及待地查看了新系统特性以及修改历史，果不其然，Android O的通知栏消息又双叒叕改版了！关于新系统的通知栏消息改变如下：[^15]

- Notification channels

> Android O 还引入了通知渠道，这是全新的由应用定义的通知内容类别。借助渠道，开发者可以让用户对不同种类的通知进行精细控制，用户可以单独拦截或更改每个渠道的行为，而不是统一管理应用的所有通知。[^16]

简单说就是增加了应用级别的通知栏消息分组功能。举个例子，用户可以分别控制微信群组和微信个人在通知栏的显示级别，群组消息混杂，可以调整较低的显示级别；而个人消息相对重要，可以调整为较高的级别。

- Snoozing

有点类似闹钟的打盹儿功能。用户可以让一条打盹儿了的通知栏消息再次出现在通知栏上。开发者可以移除或更新一条打盹儿消息，但更新这条消息不会让已经处于打盹儿状态的通知栏消息再次展示到通知栏上。

- Notification timeouts

创建一条通知栏消息时，支持设置消息有效期，超过有效期后通知栏消息会被系统取消。通过`Notification.Builder.setTimeout()`方法设置。

- Notification dismissal

新系统提供了API，区分一条通知是被用户移除或者被应用（即开发者）移除。通过`NotificationListenerService.onNotificationRemoved()`方法可以监听得到。

- Background colors

新系统提供了API设置通知栏消息的背景颜色。值得注意的是，应当谨慎使用这个API，只有当消息非常紧急，必须通知到用户的时候，才需要设置背景色。例如，可以为一个正在导航的应用，或者来电设置一个背景色。可以通过`Notification.Builder.setColor()`或者`Notification.Builder.setColorized()`设置。

由于目前没有真机/模拟器，笔者在这里有一个疑惑。之前`Notification.Builder.setColor()`这个方法在Android N上设置的是通知栏消息第一行的颜色，包括图标、应用名称、副标题等。而在Android O上变成了修改整个消息的背景色？在真机/模拟器available的时候将测试一下。

- Messaging style

设置了Messaging style风格的消息在新系统上能够展示更多的内容。消息导向(messaging-related)的通知栏消息应该使用`MessageStyle`风格代替原生消息。开发者也可以使用新的`addHistoricMessage()`方法将消息添加到通知栏中，以便提供对话的上下文信息。

## Notification channels

Android O系统引入了通知渠道，类似于分组的概念。通知渠道需要开发者手动创建，一个应用可以创建多个通知渠道，用户可以分别管理应用的每个通知渠道，管理页面由系统提供统一的UI。所有分配到同一个渠道的消息，表现都一样。当用户修改了某个渠道的以下任一种行为，都会同步到该渠道的任何一条消息：

- Importance
- Sound
- Lights
- Vibration
- Show on lockscreen
- Override do not disturb

一旦通知渠道被创建并提交到通知栏管理器（NotificationManager），那么开发者就没有权限修改通知渠道的任何配置了，所有的配置只能由用户修改。用户可以到“设置”页面，或者长按通知栏消息改变通知渠道的配置。

## Notification Priority and Importance

Android O系统弃用了旧的通知栏优先级，并提出了通知栏消息重要性这个概念。通知栏上的消息展示顺序不再由优先级控制，也无法使用重要性来控制。重要性可以控制消息展示在什么地方，例如默认级别`IMPORTANCE_DEFAULT(3)`可以展示在任意地方，如通知栏、状态栏、锁屏，可以发出通知声音，但不直接展示给用户，即不会弹出`heads-up`通知。重要性一共有6个级别：

- IMPORTANCE_NONE(0)
- IMPORTANCE_MIN(1)
- IMPORTANCE_LOW(2)
- IMPORTANCE_DEFAULT(3)
- IMPORTANCE_HIGH(4)
- IMPORTANCE_MAX(5)

开发者只能设置`IMPORTANCE_NONE(0)`至`IMPORTANCE_HIGH(4)`级别，最高级别不能通过代码控制。重要性控制也是针对通知渠道级别的，具体有没有全局性控制得看具体的模拟器设置。

# 总结

本文介绍了Android通知栏消息随着系统的更新所发生的变化以及在各个版本的Android系统通知栏消息适配过程中所产生的一些问题，并提供解决思路。随着Android版本的逐渐迭代，可以预见Android通知栏消息将会支持越来越多可配的样式，也逐渐地把权限交给用户控制，包括消息的展示以及隐私的设置。文中提到的后台通知以及“守护”通知，是对付部分应用为了常驻内存保持进程不被杀而采取的措施，这对于国内的Android生态来说无疑是利好的。

# 参考链接

1. <https://arstechnica.com/gadgets/2016/10/building-android-a-40000-word-history-of-googles-mobile-os/8/#h1>
2. <https://arstechnica.com/gadgets/2016/10/building-android-a-40000-word-history-of-googles-mobile-os/10/#2.0eclair>
3. <https://arstechnica.com/gadgets/2016/10/building-android-a-40000-word-history-of-googles-mobile-os/16/#honeycomb>
4. <https://developer.android.com/about/versions/android-4.1.html>
5. <http://www.androidpolice.com/2012/07/04/getting-to-know-android-4-1-part-2-the-glorious-new-notifications-size-matters/>
6. <https://developer.android.com/about/versions/android-4.3.html>
7. <https://developer.android.com/about/versions/android-5.0-changes.html#BehaviorNotifications>
8. <https://developer.android.com/about/versions/marshmallow/android-6.0-changes.html>
9. <https://developer.android.com/guide/topics/appwidgets/index.html#CreatingLayout>
10. <http://emojipedia.org/unicode-11.0/>
11. <http://emojipedia.org/emoji-6.0/>
12. <http://emojipedia.org>
13. <http://blog.csdn.net/guolin_blog/article/details/50945228>
14. <http://iconhandbook.co.uk/reference/chart/android/>
15. <https://developer.android.com/preview/api-overview.html>
16. <http://developers.googleblog.cn/2017/03/android-o-developer-preview.html>

--------

本文所收集的内容与图片均来自互联网以及本人编撰，如有侵权请联系本人立即删除。

--------

[^1]: <https://arstechnica.com/gadgets/2016/10/building-android-a-40000-word-history-of-googles-mobile-os/8/#h1>
[^2]: <https://arstechnica.com/gadgets/2016/10/building-android-a-40000-word-history-of-googles-mobile-os/10/#2.0eclair>
[^3]: <https://arstechnica.com/gadgets/2016/10/building-android-a-40000-word-history-of-googles-mobile-os/16/#honeycomb>
[^4]: <https://developer.android.com/about/versions/android-4.1.html>
[^5]: <http://www.androidpolice.com/2012/07/04/getting-to-know-android-4-1-part-2-the-glorious-new-notifications-size-matters/>
[^6]: <https://developer.android.com/about/versions/android-4.3.html>
[^7]: <https://developer.android.com/about/versions/android-5.0-changes.html#BehaviorNotifications>
[^8]: <https://developer.android.com/about/versions/marshmallow/android-6.0-changes.html>
[^9]: <https://developer.android.com/guide/topics/appwidgets/index.html#CreatingLayout>
[^10]: <http://emojipedia.org/unicode-11.0/>
[^11]: <http://emojipedia.org/emoji-6.0/>
[^12]: <http://emojipedia.org>
[^13]: <http://blog.csdn.net/guolin_blog/article/details/50945228>
[^14]: <http://iconhandbook.co.uk/reference/chart/android/>
[^15]: <https://developer.android.com/preview/api-overview.html>
[^16]: <http://developers.googleblog.cn/2017/03/android-o-developer-preview.html>