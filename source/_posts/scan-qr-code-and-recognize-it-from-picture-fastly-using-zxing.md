title: zxing扫描二维码和识别图片二维码及其优化策略
date: 2016-01-08 22:16:25
tags: [zxing, Android, QR Code, Optimization Strategy, Camera]
categories: Android

---

# 二维码介绍

Android中用于二维码相关的库比较少，并且大多数已经不再维护（具体可见<https://android-arsenal.com/tag/81>）。其中最常用的是zxing和zbar。

zxing项目是谷歌推出的用来识别多种格式条形码的开源项目，项目地址为<https://github.com/zxing/zxing>，zxing有多个人在维护，覆盖主流编程语言，也是目前还在维护的较受欢迎的二维码扫描开源项目之一。zbar则是主要用C来写的，速度极快，推出了iPhone的SDK和Android的相关调用方法（JNI），但这个项目已经有几年不维护了，目前并没有维护下去的意思，见<https://github.com/ZBar/ZBar>。

zxing的项目很庞大，主要的核心代码在`core`文件夹里面，也可以单独下载由这个文件夹打包而成的`jar`包，具体地址在<http://mvnrepository.com/artifact/com.google.zxing/core>，直接下载jar包也省去了通过`maven`编译的麻烦，如果喜欢折腾的，可以从<https://github.com/zxing/zxing/wiki/Getting-Started-Developing>获取帮助文档。

本文不分析二维码的生成原理和解析原理，感兴趣的可以参考陈皓的博客[二维码的生成细节和原理](http://coolshell.cn/articles/10590.html)。

# zxing基本使用

官方提供了zxing在Android机子上的使用例子，<https://github.com/zxing/zxing/tree/master/android>，作为官方的例子，zxing-android考虑了各种各样的情况，包括多种解析格式、解析得到的结果分类、长时间无活动自动销毁机制等。有时候我们需要根据自己的情况定制使用需求，因此会精简官方给的例子。在项目中，我们仅仅用来实现扫描二维码和识别图片二维码两个功能。为了实现高精度的二维码识别，在zxing原有项目的基础上，本文做了大量改进，使得二维码识别的效率有所提升。先来看看工程的项目结构。

```
.
├── QrCodeActivity.java
├── camera
│   ├── AutoFocusCallback.java
│   ├── CameraConfigurationManager.java
│   ├── CameraManager.java
│   └── PreviewCallback.java
├── decode
│   ├── CaptureActivityHandler.java
│   ├── DecodeHandler.java
│   ├── DecodeImageCallback.java
│   ├── DecodeImageThread.java
│   ├── DecodeManager.java
│   ├── DecodeThread.java
│   ├── FinishListener.java
│   └── InactivityTimer.java
├── utils
│   ├── QrUtils.java
│   └── ScreenUtils.java
└── view
    └── QrCodeFinderView.java
```

源码比较简单，这里不做过多地讲解，大部分方法都有注释。主要分为几大块，

- camera

主要实现相机的配置和管理，相机自动聚焦功能，以及相机成像回调（通过`byte[]`数组返回实际的数据）。

- decode

图片解析相关类。通过相机扫描二维码和解析图片使用两套逻辑。前者对实时性要求比较高，后者对解析结果要求较高，因此采用不同的配置。相机扫描主要在`DecodeHandler`里通过串行的方式解析，图片识别主要通过线程`DecodeImageThread`异步调用返回回调的结果。`FinishListener`和`InactivityTimer`用来控制长时间无活动时自动销毁创建的Activity，避免耗电。

- utils

图片二维码解析工具类，以及获取屏幕宽高的工具类。

- view

这个包里只有一个类`QrCodeFinderView`，官方原本是使用这个类绘制扫描区域框，并且必须在扫描区域里才能识别二维码。我把这个类稍作修改，仅仅用来展示扫描区域，实际在相机扫描二维码的时候，只要在`SurfaceView`区域范围内，结果都是有效的。

- QrCodeActivity

启动类，包含相机扫描二维码以及选择图片入口。

# zxing源码存在的问题及解决方案

zxing项目源码实现了基本的二维码扫描及图片识别程序，但下载过源码并直接运行的童鞋都知道，例子存在很多的问题，包括基本的识别精准度不高、扫描区域小、部分手机存在预览图形拉伸、默认横向扫描、还有自定义扫描界面困难等问题。

## 图形拉伸问题

先来了解一下为什么会产生图形拉伸。Android手机的屏幕分辨率可以说不胜枚举，不同型号的宽高比可能是不一样的，例如Nexus 5x、小米4的分辨率是1920X1080（当前主流手机的分辨率都是这个级别），Nexus 6p的分辨率达到2560X1440。而每台手机使用的摄像头型号更是千变万化，手机摄像头有一个成像的像素。例如普通的卡片数码相机，常常可以看到类似2304X1728、1600X1200、1027X768、640X480的字样，这些数字相乘得到的结果就代表了这个相机的成像分辨率。手机里内置的摄像头和卡片数码相机的成像原理是一样的，在摄像头预览的时候，最终都会生成连续固定像素的图片，这张图片会被投影到手机的屏幕上。**如果摄像头生成的预览图片宽高比和手机屏幕像素宽高比（准确地说是和相机预览屏幕宽高比）不一样的话，投影的结果肯定就是图片被拉伸。**

原项目其实有解决图形拉伸的问题，并且用了很细致的办法，考虑了各种机型的兼容性问题首先来看zxing是怎么解决图形拉伸问题的。在`CameraConfigurationManager`类里，初始化相机配置

```
/**
 * Reads, one time, values from the camera that are needed by the app.
 */
void initFromCameraParameters(OpenCamera camera) {
    Camera.Parameters parameters = camera.getCamera().getParameters();
    WindowManager manager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
    Display display = manager.getDefaultDisplay();

    // 判断屏幕方向，是否有需要从自然角度旋转到显示器角度
    int displayRotation = display.getRotation();
    int cwRotationFromNaturalToDisplay;
    switch (displayRotation) {
        case Surface.ROTATION_0:
            cwRotationFromNaturalToDisplay = 0;
            break;
        case Surface.ROTATION_90:
            cwRotationFromNaturalToDisplay = 90;
            break;
        case Surface.ROTATION_180:
            cwRotationFromNaturalToDisplay = 180;
            break;
        case Surface.ROTATION_270:
            cwRotationFromNaturalToDisplay = 270;
            break;
        default:
            // Have seen this return incorrect values like -90
            if (displayRotation % 90 == 0) {
                cwRotationFromNaturalToDisplay = (360 + displayRotation) % 360;
            } else {
                throw new IllegalArgumentException("Bad rotation: " + displayRotation);
            }
    }
    Log.i(TAG, "Display at: " + cwRotationFromNaturalToDisplay);

    //判断相机的方向，根据前后摄像机判断是否有需要旋转
    int cwRotationFromNaturalToCamera = camera.getOrientation();
    Log.i(TAG, "Camera at: " + cwRotationFromNaturalToCamera);

    // Still not 100% sure about this. But acts like we need to flip this:
    if (camera.getFacing() == CameraFacing.FRONT) {
        cwRotationFromNaturalToCamera = (360 - cwRotationFromNaturalToCamera) % 360;
        Log.i(TAG, "Front camera overriden to: " + cwRotationFromNaturalToCamera);
    }

    //根据屏幕方向和相机方向判断是否有需要进行旋转
    cwRotationFromDisplayToCamera = (360 + cwRotationFromNaturalToCamera - cwRotationFromNaturalToDisplay) % 360;
    Log.i(TAG, "Final display orientation: " + cwRotationFromDisplayToCamera);
    if (camera.getFacing() == CameraFacing.FRONT) {
        Log.i(TAG, "Compensating rotation for front camera");
        cwNeededRotation = (360 - cwRotationFromDisplayToCamera) % 360;
    } else {
        cwNeededRotation = cwRotationFromDisplayToCamera;
    }
    Log.i(TAG, "Clockwise rotation from display to camera: " + cwNeededRotation);

    Point theScreenResolution = new Point();
    display.getSize(theScreenResolution);
    screenResolution = theScreenResolution;
    Log.i(TAG, "Screen resolution in current orientation: " + screenResolution);
    // 寻找最佳的预览宽高值
    cameraResolution = CameraConfigurationUtils.findBestPreviewSizeValue(parameters, screenResolution);
    Log.i(TAG, "Camera resolution: " + cameraResolution);
    bestPreviewSize = CameraConfigurationUtils.findBestPreviewSizeValue(parameters, screenResolution);
    Log.i(TAG, "Best available preview size: " + bestPreviewSize);

    boolean isScreenPortrait = screenResolution.x < screenResolution.y;
    boolean isPreviewSizePortrait = bestPreviewSize.x < bestPreviewSize.y;

    if (isScreenPortrait == isPreviewSizePortrait) {
        previewSizeOnScreen = bestPreviewSize;
    } else {
        previewSizeOnScreen = new Point(bestPreviewSize.y, bestPreviewSize.x);
    }
    Log.i(TAG, "Preview size on screen: " + previewSizeOnScreen);
}
```

再来看一下`CameraConfigurationUtils.findBestPreviewSizeValue(Camera.Parameters,Point)`方法

```
public static Point findBestPreviewSizeValue(Camera.Parameters parameters, Point screenResolution) {

    // 获取当前手机支持的屏幕预览尺寸
    List<Camera.Size> rawSupportedSizes = parameters.getSupportedPreviewSizes();
    if (rawSupportedSizes == null) {
        Log.w(TAG, "Device returned no supported preview sizes; using default");
        Camera.Size defaultSize = parameters.getPreviewSize();
        if (defaultSize == null) {
            throw new IllegalStateException("Parameters contained no preview size!");
        }
        return new Point(defaultSize.width, defaultSize.height);
    }

    // 对这些尺寸根据像素值（即宽乘高的值）进行从大到小排序（感谢@丁森指正错误）
    List<Camera.Size> supportedPreviewSizes = new ArrayList<>(rawSupportedSizes);
    Collections.sort(supportedPreviewSizes, new Comparator<Camera.Size>() {
        @Override
        public int compare(Camera.Size a, Camera.Size b) {
            int aPixels = a.height * a.width;
            int bPixels = b.height * b.width;
            if (bPixels < aPixels) {
                return -1;
            }
            if (bPixels > aPixels) {
                return 1;
            }
            return 0;
        }
    });

    double screenAspectRatio = (double) screenResolution.x / (double) screenResolution.y;

    Iterator<Camera.Size> it = supportedPreviewSizes.iterator();
    while (it.hasNext()) {
        Camera.Size supportedPreviewSize = it.next();
        int realWidth = supportedPreviewSize.width;
        int realHeight = supportedPreviewSize.height;
        // 首先把不符合最小预览像素值的尺寸排除
        if (realWidth * realHeight < MIN_PREVIEW_PIXELS) {
            it.remove();
            continue;
        }

        boolean isCandidatePortrait = realWidth < realHeight;
        int maybeFlippedWidth = isCandidatePortrait ? realHeight : realWidth;
        int maybeFlippedHeight = isCandidatePortrait ? realWidth : realHeight;
        double aspectRatio = (double) maybeFlippedWidth / (double) maybeFlippedHeight;
        double distortion = Math.abs(aspectRatio - screenAspectRatio);
        // 根据宽高比判断是否满足最大误差要求（默认最大值为0.15，即宽高比默认不能超过给定比例的15%）
        if (distortion > MAX_ASPECT_DISTORTION) {
            it.remove();
            continue;
        }

        if (maybeFlippedWidth == screenResolution.x && maybeFlippedHeight == screenResolution.y) {
            Point exactPoint = new Point(realWidth, realHeight);
            Log.i(TAG, "Found preview size exactly matching screen size: " + exactPoint);
            return exactPoint;
        }
    }

    // 如果没有精确匹配到合适的尺寸，则使用最大的尺寸，这样设置便是预览图像可能产生拉伸的根本原因。
    if (!supportedPreviewSizes.isEmpty()) {
        Camera.Size largestPreview = supportedPreviewSizes.get(0);
        Point largestSize = new Point(largestPreview.width, largestPreview.height);
        Log.i(TAG, "Using largest suitable preview size: " + largestSize);
        return largestSize;
    }

    // 如果没有找到合适的尺寸，就返回默认设定的尺寸
    Camera.Size defaultPreview = parameters.getPreviewSize();
    if (defaultPreview == null) {
        throw new IllegalStateException("Parameters contained no preview size!");
    }
    Point defaultSize = new Point(defaultPreview.width, defaultPreview.height);
    Log.i(TAG, "No suitable preview sizes, using default: " + defaultSize);
    return defaultSize;
}
```

从注释中已经可以清楚地看到zxing项目在寻找最佳尺寸值的方法：

* 首先，查找手机支持的预览尺寸集合，如果集合为空，就返回默认的尺寸；否则，对尺寸集合根据尺寸的像素从小到大进行排序；
* 其次，移除不满足最小像素要求的所有尺寸；
* 在者，在剩余的尺寸集合中，剔除预览宽高比与屏幕分辨率宽高比之差的绝对值大于0.15的所有尺寸；
* 最后，寻找能够精确的与屏幕宽高匹配上的预览尺寸，如果存在则返回该宽高比；如果不存在，则使用尺寸集合中最大的那个尺寸。如果说尺寸集合已经在前面的过滤中被全部排除，则返回相机默认的尺寸值。

zxing寻找最佳预览尺寸的前三步剔除了部分不符合要求的尺寸集合，在最后一步，如果没有精确匹配到与屏幕分辨率一样的尺寸，则使用最大的尺寸。问题的关键就在这里，**最大的尺寸宽高比与屏幕宽高比相差可能很大（根据剔除规则，差距可能达到15%）（感谢@丁森指正）。**

根据这个规则，我修改了寻找最佳尺寸的源码，**将算法的核心从最大的尺寸改为比例最接近的尺寸**，这样便能够最原始地接近屏幕分辨率的宽高比，即拉伸几乎看不出来。首先定义一个比较器，用来对支持的预览尺寸集合进行排序。

```
/**
 * 预览尺寸与给定的宽高尺寸比较器。首先比较宽高的比例，在宽高比相同的情况下，根据宽和高的最小差进行比较。
 */
private static class SizeComparator implements Comparator<Camera.Size> {

    private final int width;
    private final int height;
    private final float ratio;

    SizeComparator(int width, int height) {
        if (width < height) {
            this.width = height;
            this.height = width;
        } else {
            this.width = width;
            this.height = height;
        }
        this.ratio = (float) this.height / this.width;
    }

    @Override
    public int compare(Camera.Size size1, Camera.Size size2) {
        int width1 = size1.width;
        int height1 = size1.height;
        int width2 = size2.width;
        int height2 = size2.height;

        float ratio1 = Math.abs((float) height1 / width1 - ratio);
        float ratio2 = Math.abs((float) height2 / width2 - ratio);
        int result = Float.compare(ratio1, ratio2);
        if (result != 0) {
            return result;
        } else {
            int minGap1 = Math.abs(width - width1) + Math.abs(height - height1);
            int minGap2 = Math.abs(width - width2) + Math.abs(height - height2);
            return minGap1 - minGap2;
        }
    }
}
```
目的就是根据宽高比来排序，然后调用方法取最大的那个值就可以了。

```
/**
 * 通过对比得到与宽高比最接近的尺寸（如果有相同尺寸，优先选择）
 *
 * @param surfaceWidth 需要被进行对比的原宽
 * @param surfaceHeight 需要被进行对比的原高
 * @param preSizeList 需要对比的预览尺寸列表
 * @return 得到与原宽高比例最接近的尺寸
 */
protected Camera.Size findCloselySize(int surfaceWidth, int surfaceHeight, List<Camera.Size> preSizeList) {
    Collections.sort(preSizeList, new SizeComparator(surfaceWidth, surfaceHeight));
    return preSizeList.get(0);
}
```

最后在初始化相机尺寸的时候分别对预览尺寸值和图片尺寸值都设定为比例最接近屏幕尺寸的尺寸值就可以了。本文使用的方法精简了zxing项目的步骤，实际上项目中使用的前三步滤除还是很有必要的，这里为了简短略去了。

```
void initFromCameraParameters(Camera camera) {
    Camera.Parameters parameters = camera.getParameters();
    mCameraResolution = findCloselySize(ScreenUtils.getScreenWidth(mContext), ScreenUtils.getScreenHeight(mContext),
        parameters.getSupportedPreviewSizes());
    Log.e(TAG, "Setting preview size: " + mCameraResolution.width + "-" + mCameraResolution.height);
    mPictureResolution = findCloselySize(ScreenUtils.getScreenWidth(mContext),
        ScreenUtils.getScreenHeight(mContext), parameters.getSupportedPictureSizes());
    Log.e(TAG, "Setting picture size: " + mPictureResolution.width + "-" + mPictureResolution.height);
}
```

## 扫描精度问题

使用过zxing自带的二维码扫描程序来识别二维码的童鞋应该知道，zxing二维码的扫描程序很慢，而且有可能扫不出来。zxing在配置相机参数和二维码扫描程序参数的时候，配置都比较保守，兼顾了低端手机，并且兼顾了多种条形码的识别。如果说仅仅是拿zxing项目来扫描和识别二维码的话，完全可以对项目中的一些配置做精简，并针对二维码的识别做优化。

### PlanarYUVLuminanceSource

官方的解码程序主要是下边这段代码：

```
private void decode(byte[] data, int width, int height) {
    long start = System.currentTimeMillis();
    Result rawResult = null;
    // 构造基于平面的YUV亮度源，即包含二维码区域的数据源
    PlanarYUVLuminanceSource source = activity.getCameraManager().buildLuminanceSource(data, width, height);
    if (source != null) {
        // 构造二值图像比特流，使用HybridBinarizer算法解析数据源
        BinaryBitmap bitmap = new BinaryBitmap(new HybridBinarizer(source));
        try {
            // 采用MultiFormatReader解析图像，可以解析多种数据格式
            rawResult = multiFormatReader.decodeWithState(bitmap);
        } catch (ReaderException re) {
            // continue
        } finally {
            multiFormatReader.reset();
        }
    }
	···
	// Hanlder处理解析失败或成功的结果
	···
}
```

再来看看YUV亮度源是怎么构造的，在`CameraManager`里，首先获取预览图像的聚焦框矩形`getFramingRect()`，这个聚焦框的矩形大小是根据屏幕的宽高值来做计算的，官方定义了最小和最大的聚焦框大小，分别是`240*240`和`1200*675`，即最多的聚焦框大小为屏幕宽高的5/8。获取屏幕的聚焦框大小后，还需要做从屏幕分辨率到相机分辨率的转换才能得到预览聚焦框的大小，这个转换在`getFramingRectInPreview()`里完成。这样便完成了亮度源的构造。

```
private static final int MIN_FRAME_WIDTH = 240;
private static final int MIN_FRAME_HEIGHT = 240;
private static final int MAX_FRAME_WIDTH = 1200; // = 5/8 * 1920
private static final int MAX_FRAME_HEIGHT = 675; // = 5/8 * 1080

/**
 * A factory method to build the appropriate LuminanceSource object based on the format of the preview buffers, as
 * described by Camera.Parameters.
 *
 * @param data A preview frame.
 * @param width The width of the image.
 * @param height The height of the image.
 * @return A PlanarYUVLuminanceSource instance.
 */
public PlanarYUVLuminanceSource buildLuminanceSource(byte[] data, int width, int height) {
    // 取得预览框内的矩形
    Rect rect = getFramingRectInPreview();
    if (rect == null) {
        return null;
    }
    // Go ahead and assume it's YUV rather than die.
    return new PlanarYUVLuminanceSource(data, width, height, rect.left, rect.top, rect.width(), rect.height(),
        false);
}

/**
 * Like {@link #getFramingRect} but coordinates are in terms of the preview frame, not UI / screen.
 *
 * @return {@link Rect} expressing barcode scan area in terms of the preview size
 */
public synchronized Rect getFramingRectInPreview() {
    if (framingRectInPreview == null) {
        Rect framingRect = getFramingRect();
        if (framingRect == null) {
            return null;
        }
        // 获取相机分辨率和屏幕分辨率
        Rect rect = new Rect(framingRect);
        Point cameraResolution = configManager.getCameraResolution();
        Point screenResolution = configManager.getScreenResolution();
        if (cameraResolution == null || screenResolution == null) {
            // Called early, before init even finished
            return null;
        }
        // 根据相机分辨率和屏幕分辨率的比例对屏幕中央聚焦框进行调整
        rect.left = rect.left * cameraResolution.x / screenResolution.x;
        rect.right = rect.right * cameraResolution.x / screenResolution.x;
        rect.top = rect.top * cameraResolution.y / screenResolution.y;
        rect.bottom = rect.bottom * cameraResolution.y / screenResolution.y;
        framingRectInPreview = rect;
    }
    return framingRectInPreview;
}

/**
 * Calculates the framing rect which the UI should draw to show the user where to place the barcode. This target
 * helps with alignment as well as forces the user to hold the device far enough away to ensure the image will be in
 * focus.
 *
 * @return The rectangle to draw on screen in window coordinates.
 */
public synchronized Rect getFramingRect() {
    if (framingRect == null) {
        if (camera == null) {
            return null;
        }
        // 获取屏幕的尺寸像素
        Point screenResolution = configManager.getScreenResolution();
        if (screenResolution == null) {
            // Called early, before init even finished
            return null;
        }
        // 根据屏幕的宽高找到最合适的矩形框宽高值
        int width = findDesiredDimensionInRange(screenResolution.x, MIN_FRAME_WIDTH, MAX_FRAME_WIDTH);
        int height = findDesiredDimensionInRange(screenResolution.y, MIN_FRAME_HEIGHT, MAX_FRAME_HEIGHT);

        // 取屏幕中间的，宽为width，高为height的矩形框
        int leftOffset = (screenResolution.x - width) / 2;
        int topOffset = (screenResolution.y - height) / 2;
        framingRect = new Rect(leftOffset, topOffset, leftOffset + width, topOffset + height);
        Log.d(TAG, "Calculated framing rect: " + framingRect);
    }
    return framingRect;
}

private static int findDesiredDimensionInRange(int resolution, int hardMin, int hardMax) {
    int dim = 5 * resolution / 8; // Target 5/8 of each dimension
    if (dim < hardMin) {
        return hardMin;
    }
    if (dim > hardMax) {
        return hardMax;
    }
    return dim;
}
```
这段代码并没有什么问题，也完全符合逻辑。但为什么在扫描的时候这么难扫到二维码呢，原因在于官方为了减少解码的数据，提高解码效率和速度，采用了**裁剪无用区域的方式**。这样会带来一定的问题，整个二维码数据需要完全放到聚焦框里才有可能被识别，并且在`buildLuminanceSource(byte[],int,int)`这个方法签名中，传入的byte数组便是图像的数据，并没有因为裁剪而使数据量减小，而是采用了取这个数组中的部分数据来达到裁剪的目的。对于目前CPU性能过剩的大多数智能手机来说，这种裁剪显得没有必要。如果把解码数据换成采用全幅图像数据，这样在识别的过程中便不再拘束于聚焦框，也使得二维码数据可以铺满整个屏幕。这样用户在使用程序来扫描二维码时，尽管不完全对准聚焦框，也可以识别出来。这属于一种策略上的让步，给用户造成了错觉，但提高了识别的精度。

解决办法很简单，就是不仅仅使用聚焦框里的图像数据，而是采用全幅图像的数据。

```
public PlanarYUVLuminanceSource buildLuminanceSource(byte[] data, int width, int height) {
    // 直接返回整幅图像的数据，而不计算聚焦框大小。
    return new PlanarYUVLuminanceSource(data, width, height, 0, 0, width, height, false);
}
```

### DecodeHintType

在使用zxing解析二维码时，允许事先进行相关配置，这个文件通过`Map<DecodeHintType, ?>`键值对来保存，然后使用方法`public void setHints(Map<DecodeHintType,?> hints)`来设置到相应的解码器中。DecodeHintType是一个枚举类，其中有几个重要的枚举值，

* POSSIBLE_FORMATS(List.class)

用于列举支持的解析格式，一共有17种，在`com.google.zxing.BarcodeFormat`里定义。官方默认支持所有的格式。

* TRY_HARDER(Void.class)

是否使用HARDER模式来解析数据，如果启用，则会花费更多的时间去解析二维码，对精度有优化，对速度则没有。

* CHARACTER_SET(String.class)

解析的字符集。这个对解析也比较关键，最好定义需要解析数据对应的字符集。

如果项目仅仅用来解析二维码，完全没必要支持所有的格式，也没有必要使用`MultiFormatReader`来解析。所以在配置的过程中，我移除了所有与二维码不相关的代码。直接使用`QRCodeReader`类来解析，字符集采用utf-8，使用Harder模式，并且把可能的解析格式只定义为`BarcodeFormat.QR_CODE`，这对于直接二维码扫描解析无疑是帮助最大的。

```
private final Map<DecodeHintType, Object> mHints;
DecodeHandler(QrCodeActivity activity) {
    this.mActivity = activity;
    mQrCodeReader = new QRCodeReader();
    mHints = new Hashtable<>();
    mHints.put(DecodeHintType.CHARACTER_SET, "utf-8");
    mHints.put(DecodeHintType.TRY_HARDER, Boolean.TRUE);
    mHints.put(DecodeHintType.POSSIBLE_FORMATS, BarcodeFormat.QR_CODE);
}
```

# 二维码图像识别精度探究

## 图像/像素编码格式

Android相机预览的时候支持几种不同的格式，从图像的角度（[ImageFormat](http://developer.android.com/intl/zh-cn/reference/android/graphics/ImageFormat.html)）来说有NV16、NV21、YUY2、YV12、RGB_565和JPEG，从像素的角度（[PixelFormat](http://developer.android.com/intl/zh-cn/reference/android/graphics/PixelFormat.html)）来说，有YUV422SP、YUV420SP、YUV422I、YUV420P、RGB565和JPEG，它们之间的对应关系可以从`Camera.Parameters.cameraFormatForPixelFormat(int)`方法中得到。

```
private String cameraFormatForPixelFormat(int pixel_format) {
    switch(pixel_format) {
    case ImageFormat.NV16:      return PIXEL_FORMAT_YUV422SP;
    case ImageFormat.NV21:      return PIXEL_FORMAT_YUV420SP;
    case ImageFormat.YUY2:      return PIXEL_FORMAT_YUV422I;
    case ImageFormat.YV12:      return PIXEL_FORMAT_YUV420P;
    case ImageFormat.RGB_565:   return PIXEL_FORMAT_RGB565;
    case ImageFormat.JPEG:      return PIXEL_FORMAT_JPEG;
    default:                    return null;
    }
}
```

目前大部分Android手机摄像头设置的默认格式是`yuv420sp`，其原理可参考文章[《图文详解YUV420数据格式》](http://www.cnblogs.com/azraelly/archive/2013/01/01/2841269.html)。编码成YUV的所有像素格式里，yuv420sp占用的空间是最小的。既然如此，zxing当然会考虑到这种情况。因此针对YUV编码的数据，有`PlanarYUVLuminanceSource`这个类去处理，而针对RGB编码的数据，则使用`RGBLuminanceSource`去处理。在下节介绍的图像识别算法中我们可以知道，大部分二维码的识别都是基于二值化的方法，在色域的处理上，YUV的二值化效果要优于RGB，并且RGB图像在处理中不支持旋转。因此，一种优化的思路是讲所有ARGB编码的图像转换成YUV编码，再使用`PlanarYUVLuminanceSource`去处理生成的结果。

```
/**
 * RGB转YUV420sp
 *
 * @param yuv420sp inputWidth * inputHeight * 3 / 2
 * @param argb inputWidth * inputHeight
 * @param width image width
 * @param height image height
 */
private static void encodeYUV420SP(byte[] yuv420sp, int[] argb, int width, int height) {
    // 帧图片的像素大小
    final int frameSize = width * height;
    // ---YUV数据---
    int Y, U, V;
    // Y的index从0开始
    int yIndex = 0;
    // UV的index从frameSize开始
    int uvIndex = frameSize;

    // ---颜色数据---
    int R, G, B;
    int rgbIndex = 0;

    // ---循环所有像素点，RGB转YUV---
    for (int j = 0; j < height; j++) {
        for (int i = 0; i < width; i++) {

            R = (argb[rgbIndex] & 0xff0000) >> 16;
            G = (argb[rgbIndex] & 0xff00) >> 8;
            B = (argb[rgbIndex] & 0xff);
            //
            rgbIndex++;

            // well known RGB to YUV algorithm
            Y = ((66 * R + 129 * G + 25 * B + 128) >> 8) + 16;
            U = ((-38 * R - 74 * G + 112 * B + 128) >> 8) + 128;
            V = ((112 * R - 94 * G - 18 * B + 128) >> 8) + 128;

            Y = Math.max(0, Math.min(Y, 255));
            U = Math.max(0, Math.min(U, 255));
            V = Math.max(0, Math.min(V, 255));

            // NV21 has a plane of Y and interleaved planes of VU each sampled by a factor of 2
            // meaning for every 4 Y pixels there are 1 V and 1 U. Note the sampling is every other
            // pixel AND every other scan line.
            // ---Y---
            yuv420sp[yIndex++] = (byte) Y;
            // ---UV---
            if ((j % 2 == 0) && (i % 2 == 0)) {
                //
                yuv420sp[uvIndex++] = (byte) V;
                //
                yuv420sp[uvIndex++] = (byte) U;
            }
        }
    }
}

```

Android中读取一张图片一般是通过`BitmapFactory.decodeFile(imgPath, options)`这个方法去得到这张图片的Bitmap数据，Bitmap是由ARGB值编码得到的，因此如果需要转换成YUV，还需要做一点小小的变换。

```
private static byte[] yuvs;
/**
 * 根据Bitmap的ARGB值生成YUV420SP数据。
 *
 * @param inputWidth image width
 * @param inputHeight image height
 * @param scaled bmp
 * @return YUV420SP数组
 */
public static byte[] getYUV420sp(int inputWidth, int inputHeight, Bitmap scaled) {
    int[] argb = new int[inputWidth * inputHeight];

    scaled.getPixels(argb, 0, inputWidth, 0, 0, inputWidth, inputHeight);

    /**
     * 需要转换成偶数的像素点，否则编码YUV420的时候有可能导致分配的空间大小不够而溢出。
     */
    int requiredWidth = inputWidth % 2 == 0 ? inputWidth : inputWidth + 1;
    int requiredHeight = inputHeight % 2 == 0 ? inputHeight : inputHeight + 1;

    int byteLength = requiredWidth * requiredHeight * 3 / 2;
    if (yuvs == null || yuvs.length < byteLength) {
        yuvs = new byte[byteLength];
    } else {
        Arrays.fill(yuvs, (byte) 0);
    }

    encodeYUV420SP(yuvs, argb, inputWidth, inputHeight);

    scaled.recycle();

    return yuvs;
}

```

这里面有几个坑，在方法里已经列出来了。首先，如果每次都生成新的YUV数组，不知道在扫一扫解码时要进行GC多少次。。。所以就采用了静态的数组变量来存储数据，只有当当前的长宽乘积超过数组大小时，才重新生成新的*`yuvs`*。其次，如果鉴于YUV的特性，长宽只能是偶数个像素点，否则可能会造成数组溢出(不信可以尝试)。最后，使用完了Bitmap要记得回收，那玩意吃内存不是随便说说的。

## 二维码图像识别算法选择

二维码扫描精度和许多因素有关，最关键的因素是扫描算法。目前在图形识别领域中，较常用的二维码识别算法主要有两种，分别是`HybridBinarizer`和`GlobalHistogramBinarizer`，这两种算法都是基于二值化，即将图片的色域变为黑白两个颜色，然后提取图形中的二维码矩阵。实际上，zxing中的HybridBinarizer继承自GlobalHistogramBinarizer，并在此基础上做了功能性的改进。援引官方介绍：

**This Binarizer(GlobalHistogramBinarizer) implementation uses the old ZXing global histogram approach. It is suitable for low-end mobile devices which don't have enough CPU or memory to use a local thresholding algorithm. However, because it picks a global black point, it cannot handle difficult shadows and gradients. Faster mobile devices and all desktop applications should probably use HybridBinarizer instead.**

**This class(HybridBinarizer) implements a local thresholding algorithm, which while slower than the GlobalHistogramBinarizer, is fairly efficient for what it does. It is designed for high frequency images of barcodes with black data on white backgrounds. For this application, it does a much better job than a global blackpoint with severe shadows and gradients. However it tends to produce artifacts on lower frequency images and is therefore not a good general purpose binarizer for uses outside ZXing. This class extends GlobalHistogramBinarizer, using the older histogram approach for 1D readers, and the newer local approach for 2D readers. 1D decoding using a per-row histogram is already inherently local, and only fails for horizontal gradients. We can revisit that problem later, but for now it was not a win to use local blocks for 1D.** ···

`GlobalHistogramBinarizer`算法**适合于低端的设备，对手机的CPU和内存要求不高**。但它选择了全部的黑点来计算，因此无法处理阴影和渐变这两种情况。HybridBinarizer算法在执行效率上要慢于GlobalHistogramBinarizer算法，但识别相对更有效。它专门为**以白色为背景的连续黑色块二维码图像**解析而设计，也更适合用来解析具有**严重阴影和渐变**的二维码图像。

网上对这两种算法的解析并不多，目前仅找到一篇文章详解了GlobalHistogramBinarizer算法，详见<http://www.chongchonggou.com/g_4200594.html>。有时间再看一下相关源码。

> 注：原文链接为<http://kuangjianwei.blog.163.com/blog/static/190088953201361015055110/>，已经打不开了，现暂时替换为<http://www.chongchonggou.com/g_4200594.html>，下同。

zxing项目官方默认使用的是HybridBinarizer二值化方法。在实际的测试中，和官方的介绍大致一样。然而目前的大部分二维码都是黑色二维码，白色背景的。不管是二维码扫描还是二维码图像识别，使用GlobalHistogramBinarizer算法的效果要稍微比HybridBinarizer好一些，识别的速度更快，对低分辨的图像识别精度更高。

除了这两种算法，我相信在图像识别领域肯定还有更好的算法存在，目前受限于知识水平，对二值化算法这一块还比较陌生，期待以后能够深入理解并改进目前的开源算法(\*^__^\*)……

## 图像大小对识别精度的影响

这点是测试中无意发现的。现在的手机摄像头拍照出现的照片像素都很高，动不动就1200W像素，1600W像素，甚至是2000W都不稀奇，但照片的成像质量不一定高。将一张高分辨率的图片按原分辨率导入Android手机，很容易产生OOM。我们来计算一下，导入一张1200W像素的图片需要的内存，假设图片是`4000px*3000px`，如果导入的图片采用ARGB_8888编码形式，则每个像素需要占用4个Bytes(分别存储ARGB值)来存储，则需要`4000*3000*4bytes=45.776MB`的内存，这在有限的移动资源里，显然是不能忍受的。

通过上一节对图像算法的简单研究，在`GlobalHistogramBinarizer`中，是从图像中均匀取5行（覆盖整个图像高度），每行取中间五分之四作为样本；以灰度值为X轴，每个灰度值的像素个数为Y轴建立一个直方图，从直方图中取点数最多的一个灰度值，然后再去给其他的灰度值进行分数计算，按照点数乘以与最多点数灰度值的距离的平方来进行打分，选分数最高的一个灰度值。接下来在这两个灰度值中间选取一个区分界限，取的原则是尽量靠近中间并且要点数越少越好。界限有了以后就容易了，与整幅图像的每个点进行比较，如果灰度值比界限小的就是黑，在新的矩阵中将该点置1，其余的就是白，为0。（摘自[zxing源码分析——QR码部分](http://www.chongchonggou.com/g_4200594.html)）

根据算法的实现，可以知道图像的分辨率对二维码的取值是有影响的。并不是图像的分辨率越高就越容易取到二维码。高分辨率的图像对Android的内存资源占用也很可怕。所以在测试的过程中，我尝试将图片压缩成不同大小分辨率，然后再进行图片的二维码识别。

```
/**
 * 根据给定的宽度和高度动态计算图片压缩比率
 * 
 * @param options Bitmap配置文件
 * @param reqWidth 需要压缩到的宽度
 * @param reqHeight 需要压缩到的高度
 * @return 压缩比
 */
public static int calculateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;

    if (height > reqHeight || width > reqWidth) {

        final int halfHeight = height / 2;
        final int halfWidth = width / 2;

        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while ((halfHeight / inSampleSize) > reqHeight && (halfWidth / inSampleSize) > reqWidth) {
            inSampleSize *= 2;
        }
    }

    return inSampleSize;
}

/**
 * 将图片根据压缩比压缩成固定宽高的Bitmap，实际解析的图片大小可能和#reqWidth、#reqHeight不一样。
 * 
 * @param imgPath 图片地址
 * @param reqWidth 需要压缩到的宽度
 * @param reqHeight 需要压缩到的高度
 * @return Bitmap
 */
public static Bitmap decodeSampledBitmapFromFile(String imgPath, int reqWidth, int reqHeight) {

    // First decode with inJustDecodeBounds=true to check dimensions
    final BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeFile(imgPath, options);

    // Calculate inSampleSize
    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);

    // Decode bitmap with inSampleSize set
    options.inJustDecodeBounds = false;
    return BitmapFactory.decodeFile(imgPath, options);
}

```

Android图片优化需要通过在解析图片的时候，设置`BitmapFactory.Options.inSampleSize`的值，根据比例压缩图片大小。在进行图片二维码解析的线程中，通过设置不同的图片大小，来测试二维码的识别率。这个测试过程我忘记保存了，只记得测试了压缩成最大宽高值为2048、1024、512、256和128像素的包含二维码的图片，但实际的测试结果是，当`MAX_PICTURE_PIXEL=256`的时候识别率最高。

**此结论不具备理论支持，有兴趣的童鞋可以自己动手尝试。^_^**

## 相机预览倍数设置及聚焦时间调整

如果使用zxing默认的相机配置，会发现需要离二维码很近才能够识别出来，但这样会带来一个问题——聚焦困难。解决办法就是调整相机预览倍数以及减小相机聚焦的时间。

通过测试可以发现，每个手机的最大放大倍数几乎是不一样的，这可能和摄像头的型号有关。如果设置成一个固定的值，那可能会产生在某些手机上过度放大，某些手机上放大的倍数不够。索性相机的参数设定里给我们提供了最大的放大倍数值，通过取放大倍数值的N分之一作为当前的放大倍数，就完美地解决了手机的适配问题。

```
// 需要判断摄像头是否支持缩放
Parameters parameters = camera.getParameters();
if (parameters.isZoomSupported()) {
	// 设置成最大倍数的1/10，基本符合远近需求
    parameters.setZoom(parameters.getMaxZoom() / 10);
}
```

zxing默认的相机聚焦时间是`2s`，可以根据扫描的视觉适当调整。聚焦时间的调整也很简单，在`AutoFocusCallback`这个类里，调整`AUTO_FOCUS_INTERVAL_MS`这个值就可以了。

# 二维码扫描视觉调整

二维码扫描视觉的绘制在`ViewfinderView.java`完成，官方是继承了View然后在`onDraw()`方法中实现了视图的绘制。

```
@Override
public void onDraw(Canvas canvas) {
    if (cameraManager == null) {
        return; // not ready yet, early draw before done configuring
    }
    Rect frame = cameraManager.getFramingRect();
    Rect previewFrame = cameraManager.getFramingRectInPreview();
    if (frame == null || previewFrame == null) {
        return;
    }
    int width = canvas.getWidth();
    int height = canvas.getHeight();

    // 绘制聚焦框外的暗色透明层
    paint.setColor(resultBitmap != null ? resultColor : maskColor);
    canvas.drawRect(0, 0, width, frame.top, paint);
    canvas.drawRect(0, frame.top, frame.left, frame.bottom + 1, paint);
    canvas.drawRect(frame.right + 1, frame.top, width, frame.bottom + 1, paint);
    canvas.drawRect(0, frame.bottom + 1, width, height, paint);

    if (resultBitmap != null) {
        // 如果扫描结果不为空，则把扫描的结果填充到聚焦框中
        paint.setAlpha(CURRENT_POINT_OPACITY);
        canvas.drawBitmap(resultBitmap, null, frame, paint);
    } else {

        // 画一根红色的激光线表示二维码解码正在进行
        paint.setColor(laserColor);
        paint.setAlpha(SCANNER_ALPHA[scannerAlpha]);
        scannerAlpha = (scannerAlpha + 1) % SCANNER_ALPHA.length;
        int middle = frame.height() / 2 + frame.top;
        canvas.drawRect(frame.left + 2, middle - 1, frame.right - 1, middle + 2, paint);

        float scaleX = frame.width() / (float) previewFrame.width();
        float scaleY = frame.height() / (float) previewFrame.height();

        List<ResultPoint> currentPossible = possibleResultPoints;
        List<ResultPoint> currentLast = lastPossibleResultPoints;
        int frameLeft = frame.left;
        int frameTop = frame.top;

        // 绘制解析过程中可能扫描到的关键点，使用黄色小圆点表示
        if (currentPossible.isEmpty()) {
            lastPossibleResultPoints = null;
        } else {
            possibleResultPoints = new ArrayList<>(5);
            lastPossibleResultPoints = currentPossible;
            paint.setAlpha(CURRENT_POINT_OPACITY);
            paint.setColor(resultPointColor);
            synchronized (currentPossible) {
                for (ResultPoint point : currentPossible) {
                    canvas.drawCircle(frameLeft + (int) (point.getX() * scaleX),
                        frameTop + (int) (point.getY() * scaleY), POINT_SIZE, paint);
                }
            }
        }
        if (currentLast != null) {
            paint.setAlpha(CURRENT_POINT_OPACITY / 2);
            paint.setColor(resultPointColor);
            synchronized (currentLast) {
                float radius = POINT_SIZE / 2.0f;
                for (ResultPoint point : currentLast) {
                    canvas.drawCircle(frameLeft + (int) (point.getX() * scaleX),
                        frameTop + (int) (point.getY() * scaleY), radius, paint);
                }
            }
        }
        // 重绘聚焦框里的内容，不需要重绘整个界面。
        postInvalidateDelayed(ANIMATION_DELAY, frame.left - POINT_SIZE, frame.top - POINT_SIZE,
            frame.right + POINT_SIZE, frame.bottom + POINT_SIZE);
    }
}
```

我给它做了一点小改变，效果差不多，代码更简洁一些。由于代码中我不是根据屏幕的宽高动态计算聚焦框的大小，因此这里省去了从CameraManager获取FramingRect和FramingRectInPreview这两个矩形的过程。我在聚焦框外加了四个角，目前大部分二维码产品基本都是这么设计的吧，当然也可以使用图片来代替。总之视觉定制是因人而异，这里不做过多介绍。

```
@Override
public void onDraw(Canvas canvas) {
    if (isInEditMode()) {
        return;
    }
    Rect frame = mFrameRect;
    if (frame == null) {
        return;
    }
    int width = canvas.getWidth();
    int height = canvas.getHeight();

    // 绘制焦点框外边的暗色背景
    mPaint.setColor(mMaskColor);
    canvas.drawRect(0, 0, width, frame.top, mPaint);
    canvas.drawRect(0, frame.top, frame.left, frame.bottom + 1, mPaint);
    canvas.drawRect(frame.right + 1, frame.top, width, frame.bottom + 1, mPaint);
    canvas.drawRect(0, frame.bottom + 1, width, height, mPaint);

    drawFocusRect(canvas, frame);
    drawAngle(canvas, frame);
    drawText(canvas, frame);
    drawLaser(canvas, frame);

    // Request another update at the animation interval, but only repaint the laser line,
    // not the entire viewfinder mask.
    postInvalidateDelayed(ANIMATION_DELAY, frame.left, frame.top, frame.right, frame.bottom);
}

/**
 * 画聚焦框，白色的
 * 
 * @param canvas
 * @param rect
 */
private void drawFocusRect(Canvas canvas, Rect rect) {
    // 绘制焦点框（黑色）
    mPaint.setColor(mFrameColor);
    // 上
    canvas.drawRect(rect.left + mAngleLength, rect.top, rect.right - mAngleLength, rect.top + mFocusThick, mPaint);
    // 左
    canvas.drawRect(rect.left, rect.top + mAngleLength, rect.left + mFocusThick, rect.bottom - mAngleLength,
        mPaint);
    // 右
    canvas.drawRect(rect.right - mFocusThick, rect.top + mAngleLength, rect.right, rect.bottom - mAngleLength,
        mPaint);
    // 下
    canvas.drawRect(rect.left + mAngleLength, rect.bottom - mFocusThick, rect.right - mAngleLength, rect.bottom,
        mPaint);
}

/**
 * 画粉色的四个角
 * 
 * @param canvas
 * @param rect
 */
private void drawAngle(Canvas canvas, Rect rect) {
    mPaint.setColor(mLaserColor);
    mPaint.setAlpha(OPAQUE);
    mPaint.setStyle(Paint.Style.FILL);
    mPaint.setStrokeWidth(mAngleThick);
    int left = rect.left;
    int top = rect.top;
    int right = rect.right;
    int bottom = rect.bottom;
    // 左上角
    canvas.drawRect(left, top, left + mAngleLength, top + mAngleThick, mPaint);
    canvas.drawRect(left, top, left + mAngleThick, top + mAngleLength, mPaint);
    // 右上角
    canvas.drawRect(right - mAngleLength, top, right, top + mAngleThick, mPaint);
    canvas.drawRect(right - mAngleThick, top, right, top + mAngleLength, mPaint);
    // 左下角
    canvas.drawRect(left, bottom - mAngleLength, left + mAngleThick, bottom, mPaint);
    canvas.drawRect(left, bottom - mAngleThick, left + mAngleLength, bottom, mPaint);
    // 右下角
    canvas.drawRect(right - mAngleLength, bottom - mAngleThick, right, bottom, mPaint);
    canvas.drawRect(right - mAngleThick, bottom - mAngleLength, right, bottom, mPaint);
}

private void drawText(Canvas canvas, Rect rect) {
    int margin = 40;
    mPaint.setColor(mTextColor);
    mPaint.setTextSize(getResources().getDimension(R.dimen.text_size_13sp));
    String text = getResources().getString(R.string.qr_code_auto_scan_notification);
    Paint.FontMetrics fontMetrics = mPaint.getFontMetrics();
    float fontTotalHeight = fontMetrics.bottom - fontMetrics.top;
    float offY = fontTotalHeight / 2 - fontMetrics.bottom;
    float newY = rect.bottom + margin + offY;
    float left = (ScreenUtils.getScreenWidth(mContext) - mPaint.getTextSize() * text.length()) / 2;
    canvas.drawText(text, left, newY, mPaint);
}

private void drawLaser(Canvas canvas, Rect rect) {
    // 绘制焦点框内固定的一条扫描线（红色）
    mPaint.setColor(mLaserColor);
    mPaint.setAlpha(SCANNER_ALPHA[mScannerAlpha]);
    mScannerAlpha = (mScannerAlpha + 1) % SCANNER_ALPHA.length;
    int middle = rect.height() / 2 + rect.top;
    canvas.drawRect(rect.left + 2, middle - 1, rect.right - 1, middle + 2, mPaint);

}
```

# 总结

使用zxing进行二维码的编解码是非常方便的，zxing的API覆盖了多种主流编程语言，具有良好的扩展性和可定制性。文中进行了二维码基本功能介绍，zxing项目基本使用方法，zxing项目中目前存在的缺点及改进方案，以及自己在进行zxing项目二次开发的摸索过程中总结出的提高二维码扫描的方法。文中还有许多不足的地方，对源码的理解还不够深，特别是二维码解析关键算法（`GlobalHistogramBinarizer`和`HybridBinarizer`）。这些算法需要投入额外的时间去理解，对于目前以业务为导向的App开发来说，还存在优化的空间，期待将来有一天能像微信的二维码扫描一样快速，精确。

项目源码：<https://github.com/iluhcm/QrCodeScanner>