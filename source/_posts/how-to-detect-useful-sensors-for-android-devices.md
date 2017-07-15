title: Android系统中Sensor及SensorManager探究
date: 2015-09-03 20:06:12
categories: Android
tags: [Tech,Android,Sensor]
---

# 传感器类型

最近在做一个收集Android传感器信息的应用,发现Android的传感器并没有想象中的那么好用. 与iOS不同的是,Android手机上的硬件设备可能来自不同的厂商,硬件信息千篇一律,完全掌握其对应的传感器基本是不可能的.在Android系统里,定义了26种传感器类型,分别是:

```java   
public static final String STRING_TYPE_ACCELEROMETER = "android.sensor.accelerometer";
public static final String STRING_TYPE_MAGNETIC_FIELD = "android.sensor.magnetic_field";
public static final String STRING_TYPE_ORIENTATION = "android.sensor.orientation";
public static final String STRING_TYPE_GYROSCOPE = "android.sensor.gyroscope";
public static final String STRING_TYPE_LIGHT = "android.sensor.light";
public static final String STRING_TYPE_PRESSURE = "android.sensor.pressure";
public static final String STRING_TYPE_TEMPERATURE = "android.sensor.temperature";
public static final String STRING_TYPE_PROXIMITY = "android.sensor.proximity";
public static final String STRING_TYPE_GRAVITY = "android.sensor.gravity";
public static final String STRING_TYPE_LINEAR_ACCELERATION = "android.sensor.linear_acceleration";
public static final String STRING_TYPE_ROTATION_VECTOR = "android.sensor.rotation_vector";
public static final String STRING_TYPE_RELATIVE_HUMIDITY = "android.sensor.relative_humidity";
public static final String STRING_TYPE_AMBIENT_TEMPERATURE = "android.sensor.ambient_temperature";
public static final String STRING_TYPE_MAGNETIC_FIELD_UNCALIBRATED = "android.sensor.magnetic_field_uncalibrated";
public static final String STRING_TYPE_GAME_ROTATION_VECTOR = "android.sensor.game_rotation_vector";
public static final String STRING_TYPE_GYROSCOPE_UNCALIBRATED = "android.sensor.gyroscope_uncalibrated";
public static final String STRING_TYPE_SIGNIFICANT_MOTION = "android.sensor.significant_motion";
public static final String STRING_TYPE_STEP_DETECTOR = "android.sensor.step_detector";
public static final String STRING_TYPE_STEP_COUNTER = "android.sensor.step_counter";
public static final String STRING_TYPE_GEOMAGNETIC_ROTATION_VECTOR = "android.sensor.geomagnetic_rotation_vector";
public static final String STRING_TYPE_HEART_RATE = "android.sensor.heart_rate";
public static final String SENSOR_STRING_TYPE_TILT_DETECTOR = "android.sensor.tilt_detector";
public static final String STRING_TYPE_WAKE_GESTURE = "android.sensor.wake_gesture";
public static final String STRING_TYPE_GLANCE_GESTURE = "android.sensor.glance_gesture";
public static final String STRING_TYPE_PICK_UP_GESTURE = "android.sensor.pick_up_gesture";
public static final String STRING_TYPE_WRIST_TILT_GESTURE = "android.sensor.wrist_tilt_gesture";
```


其中常用的有`加速度计`,`磁力计`,`方向传感器`,`陀螺仪`,`重力传感器`,`光线传感器`等.并不是每一台手机都包含了这26种类型的传感器,甚至有些低端的手机可能只有其中的几个.

# 获取手机支持的传感器信息

获取某个手机里所有的传感器信息只需要三行代码:

```
SensorManager mSensorManager = (SensorManager) context.getSystemService(Context.SENSOR_SERVICE);
List<Sensor> allSensors = mSensorManager.getSensorList(Sensor.TYPE_ALL);// 获得传感器列表
Log.v(TAG, allSensors.toString());
```

当然如何格式化输出可以自由定制,其中,一个Sensor包含了如下字段:

* name : 传感器名称
* vendor : 设备商名称
* version : 设备版本
* type : 类型
* max range : 最大量程
* resolution : 分辨率
* power : 功耗
* min delay : 最小延迟

注意,使用此方法获取的传感器可能有多个相同的名字,可能是因为手机里内置了两个相同的传感器.例如,在我的测试机小米3上就有两个温度传感器.

# 在Android中注册指定类型的传感器

最近做的应用,需要获取的传感器类型有:`线性加速度计`,`磁力计`,`重力传感器`,`陀螺仪`.在Android中使用传感器比较简单,可以通过下面的代码注册,这段代码实现需要在主类实现`SensorEventListener`接口.

```
private void registerSensorListener() {
    SensorDataRecorder mSensorManager = (SensorManager) context.getSystemService(Context.SENSOR_SERVICE);
    Sensor acc = mSensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);
    Sensor linearAcc = mSensorManager.getDefaultSensor(Sensor.TYPE_LINEAR_ACCELERATION);
    Sensor gravity = mSensorManager.getDefaultSensor(Sensor.TYPE_GRAVITY);
    Sensor gyroscope = mSensorManager.getDefaultSensor(Sensor.TYPE_GYROSCOPE);
    Sensor geoMagnetic = mSensorManager.getDefaultSensor(Sensor.TYPE_MAGNETIC_FIELD);
    Sensor rotationVector = mSensorManager.getDefaultSensor(Sensor.TYPE_ROTATION_VECTOR);
    if (linearAcc != null && gyroscope != null && geoMagnetic != null && gravity != null) {
        Log.v(TAG, "register all sensors");
        mSensorManager.registerListener(this, acc, SensorManager.SENSOR_DELAY_FASTEST);
        mSensorManager.registerListener(this, linearAcc, SensorManager.SENSOR_DELAY_FASTEST);
        mSensorManager.registerListener(this, gyroscope, SensorManager.SENSOR_DELAY_FASTEST);
        mSensorManager.registerListener(this, gravity, SensorManager.SENSOR_DELAY_FASTEST);
        mSensorManager.registerListener(this, geoMagnetic, SensorManager.SENSOR_DELAY_FASTEST);
        mSensorManager.registerListener(this, rotationVector, SensorManager.SENSOR_DELAY_FASTEST);
    }
}

```
实际上在注册传感器时,最终都会调用抽象方法

```
protected abstract boolean registerListenerImpl(SensorEventListener listener, Sensor sensor, int delayUs, Handler handler, int maxReportLatencyUs, int SensorManager.SENSOR_DELAY_FASTEST);
```
第一个参数便是注册事件回调;第二个即要注册的传感器;第三个延迟时间是通过第五个参数计算得到;第四个Handler表示事件可以被该Handler处理,可以为空,表示不使用Handler处理;第五个参数可定义传感器的采样频率,必须是系统中设定的几个值:

* `SENSOR_DELAY_NORMAL`
* `SENSOR_DELAY_UI`
* `SENSOR_DELAY_GAME`
* `SENSOR_DELAY_FASTEST`

采样频率逐个提高,最快是`SENSOR_DELAY_FASTEST`,1ms采样一次;第六个参数官方没有解释,表示不太清楚.在官方的文档解释中,可以通过返回值判断是否注册成功了某个传感器.然而在实际的测试中,即使注册成功了在回调中也不一定有返回值.下面就可以通过回调判断传感器类型并获取值:

```
@Override
public void onSensorChanged(SensorEvent event) {
    if (event != null && event.sensor != null) {
        if (event.values[0] == 0.0 && event.values[1] == 0.0 && event.values[2] == 0.0) { // 所有数值为0时认为传感器未工作
            Log.v(TAG, "sensor not work normally. " + event.sensor.getName());
            return;
        }
        switch (event.sensor.getType()) {
            case Sensor.TYPE_ACCELEROMETER:
                mAccValues = event.values.clone();
                mAccAccuracy = event.accuracy;
                mAllSensorValid |= LINEAR_ACC_VALID;
                mAllSensorValid |= GRAVITY_VALID;
                break;
            case Sensor.TYPE_LINEAR_ACCELERATION:
                mLinearAccValues = event.values.clone();
                mAccAccuracy = event.accuracy;
                mAllSensorValid |= LINEAR_ACC_VALID;
                break;
            case Sensor.TYPE_GYROSCOPE:
                mGyroscopeValues = event.values.clone();
                mAllSensorValid |= GYROSCOPE_VALID;
                break;
            case Sensor.TYPE_GRAVITY:
                mGravityValues = event.values.clone();
                mAllSensorValid |= GRAVITY_VALID;
                break;
            case Sensor.TYPE_MAGNETIC_FIELD:
                mGeoMagneticValues = event.values.clone();
                mMagneticAccuracy = event.accuracy;
                mAllSensorValid |= GEOMAGNETIC_VALID;
                break;
            case Sensor.TYPE_ROTATION_VECTOR:
                mRotationVectorValues = event.values.clone();
                // mAllSensorValid |= ROTATION_VECTOR_VALID;
                break;
            default:
                break;
        }
    }
}

```
其中,我注册了6个传感器.由于加速度传感器是通过线性加速度传感器和重力传感器计算出来的,前者等于后两者之和.比较坑的是,有些手机只能够检测到加速度传感器,而屏蔽了加速度传感器和重力传感器的值.这也是上面的代码中,为什么我获取到加速度的值后,将线性加速度传感器和重力传感器对应的比特位置为了有效值.

# 如何判断传感器是否可用

通过上面的分析,我们可以通过两种方式获取到Android手机里是否包含某个传感器

 * 通过获取传感器列表,再判断集合中是否包含某传感器
`mSensorManager.getSensorList(Sensor.TYPE_ALL)`
 * 通过注册某传感器,判断返回值

```
Sensor acc = mSensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);
boolean isAccValid = mSensorManager.registerListener(this, acc, SensorManager.SENSOR_DELAY_FASTEST);
```
然而这只能证明手机中确实是存在这个传感器硬件的,想要判断传感器是否可用,通过这两种方式还是无能为力.我采用了最笨的一种方法,去判断传感器是否可用,那就是在回调中判断传感器类型,如果能够返回不等于零的值,则证明该传感器可用.

```
public synchronized void checkSensor2(final ISensorCallback callback) {
    List<Sensor> allSensors = mSensorManager.getSensorList(Sensor.TYPE_ALL);// 获得传感器列表
    callback.checkResult("Total Sensor: " + allSensors.size(), null);
    for (final Sensor sensor : allSensors) {
        final long currentTime = System.currentTimeMillis();
        mSensorManager.registerListener(new SensorEventListener() {
            @Override
            public void onSensorChanged(SensorEvent event) {
                if (System.currentTimeMillis() - currentTime <= 5000) {
                    if (event.sensor == sensor) {
                        if (event.values.length != 0) {
                            float[] values = event.values;
                            callback.checkResult(
                                    "---------------------------\nSensor '" + sensor.getName()
                                            + "' is available\nTest Data: " + values[0] + " " + values[1] + " " + values[2],
                                    sensor);
                            mSensorManager.unregisterListener(this);
                        } else {
                            callback.checkResult(
                                    "---------------------------\nSensor '" + sensor.getName() + "' is unavailable",
                                    sensor);
                            mSensorManager.unregisterListener(this);
                        }
                    } else {
                        callback
                                .checkResult("---------------------------\nWrong sensor data received.\nSensor wanted: "
                                        + sensor.getName() + ", Sensor provided: " + event.sensor.getName(), sensor);
                        mSensorManager.unregisterListener(this);
                    }
                } else {
                    callback.checkResult(
                            "---------------------------\nSensor: '" + sensor.getName() + "' Timeout! ", sensor);
                    mSensorManager.unregisterListener(this);
                }
            }

            @Override
            public void onAccuracyChanged(Sensor sensor, int accuracy) {
            }
        }, sensor, SensorManager.SENSOR_DELAY_NORMAL);
    }
}

public interface ISensorCallback {
    void checkResult(String result, Sensor sensor);
}
```
在这段代码中,我设定了5秒钟来获取传感器的值,获取不到则返回获取某个传感器超时.我测试了几个手机,发现每个手机上具备的传感器都不一样.在华为的C199手机上,通过获取传感器类型可以得到11个传感器,但实际上只有7个传感器可用.