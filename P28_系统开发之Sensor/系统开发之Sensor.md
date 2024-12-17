# P28: 系统开发之Sensor

<img src="../flower/flower_p28.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

# sensor的整体架构


![sensor的整体架构_01](sensor的整体架构_01.png)


![sensor的整体架构_02](sensor的整体架构_02.png)


![sensor的整体架构_03](sensor的整体架构_03.png)



---


## Application层：

第一层是应用层：代表上层使用frameworks的接口注册或使用一个sensor：

第一步：获取SensorManager对象

第二步：获取Sensor对象

第三步：注册Sensor对象

第四步：重写onAccuracyChanged，onSensorChanged这两个方法

第五步：注销Sensor对象

```java
public class SensorActivity extends Activity implements SensorEventListener {

  private SensorManager mSensorManager;
  private Sensor mSensor;

  @Override
  public final void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);
    //第一步：通过getSystemService获得SensorManager实例对象
    mSensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
    //第二步：通过SensorManager实例对象获得想要的传感器对象:参数决定获取哪个传感器
    mSensor = mSensorManager.getDefaultSensor(Sensor.TYPE_LIGHT);
  }
  
  //第四步：必须重写的两个方法：onAccuracyChanged，onSensorChanged
  /**
   * 传感器精度发生改变的回调接口
   */
  @Override
  public final void onAccuracyChanged(Sensor sensor, int accuracy) {
    //TODO 在传感器精度发生改变时做些操作，accuracy为当前传感器精度
  }
  
  /**
   * 传感器事件值改变时的回调接口：执行此方法的频率与注册传感器时的频率有关
   */
  @Override
  public final void onSensorChanged(SensorEvent event) {
    // 大部分传感器会返回三个轴方向x,y,x的event值，值的意义因传感器而异
    float x = event.values[0];
    float y = event.values[1];
    float z = event.values[2];
    //TODO 利用获得的三个float传感器值做些操作
  }
  
  /**
   * 第三步：在获得焦点时注册传感器并让本类实现SensorEventListener接口
   */
  @Override
  protected void onResume() {
    super.onResume();
    /*
     *第一个参数：SensorEventListener接口的实例对象
     *第二个参数：需要注册的传感器实例
     *第三个参数：传感器获取传感器事件event值频率：
   *              SensorManager.SENSOR_DELAY_FASTEST = 0：对应0微秒的更新间隔，最快，1微秒 = 1 % 1000000秒
   *              SensorManager.SENSOR_DELAY_GAME = 1：对应20000微秒的更新间隔，游戏中常用
   *              SensorManager.SENSOR_DELAY_UI = 2：对应60000微秒的更新间隔
   *              SensorManager.SENSOR_DELAY_NORMAL = 3：对应200000微秒的更新间隔
   *              键入自定义的int值x时：对应x微秒的更新间隔
     *
     */
    mSensorManager.registerListener(this, mSensor, SensorManager.SENSOR_DELAY_NORMAL);
  }
  
  /**
   * 第五步：在失去焦点时注销传感器
   */
  @Override
  protected void onPause() {
    super.onPause();
    mSensorManager.unregisterListener(this);
  }
}
```

设备不一定支持你需要的sensor，使用前可以先判断下可用性

```java
private SensorManager mSensorManager;
...
mSensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
if (mSensorManager.getDefaultSensor(Sensor.TYPE_MAGNETIC_FIELD) != null){
    // Success! There's a magnetometer.
} else {
    // Failure! No magnetometer.
}

//或者
List<Sensor> deviceSensors = mSensorManager.getSensorList(Sensor.TYPE_ALL);
```

---

## Frameworks层： 

包含SensorService， 接收HAl层传上来的 sensor event 数据，并让应用层做出相应的动作，如陀螺仪转动，横竖屏切换等。

frameworks java:

frameworks\base\core\java\android\hardware\SensorManager.java
frameworks\base\core\java\android\hardware\SystemSensorManager.java

```java
    private final HashMap<SensorEventListener, SensorEventQueue> mSensorListeners =
            new HashMap<SensorEventListener, SensorEventQueue>();//注册的sensor监听回调接口
```

sensor事件回调接口：

frameworks\base\core\java\android\hardware\SensorEventListener.java
frameworks\base\core\java\android\hardware\SensorListener.java

jni:

frameworks\base\core\jni\android_hardware_SensorManager.cpp


native:

服务端 SensorService：

frameworks\native\services\sensorservice\SensorService.h
frameworks\native\services\sensorservice\SensorService.cpp


---

## HAL层： 

![HAL层](HAL层.png)

SensorDevice 的poll函数读取数据


hardware:

```java
hardware/interfaces/sensors/2.0/multihal/Android.bp
u:r:hal_sensors_default:s0     system          806      1 12565056  5644 0                   0 S android.hardware.sensors@2.0-service.multihal

 vendor/qcom/proprietary/sensors-see/sensorcal-hidl-impl/Android.bp
u:r:vendor_hal_sensorscalibrate_qti_default:s0 system 839 1 12350328 2188 0                  0 S vendor.qti.hardware.sensorscalibrate@1.0-service

 vendor/qcom/proprietary/sensors-see/sensordaemon/Android.bp
u:r:vendor_sensors_qti:s0      system          979      1 12437944  1372 0                   0 S sensors.qti
```




```java
hardware/interfaces/sensors/
vendor\mediatek\proprietary\hardware\sensor
hardware/libhardware/include/hardware/sensors.h
```


Google为Sensor提供了统一的HAL接口，不同的硬件厂商需要根据该接口来实现并完成具体的硬件抽象层，Android中Sensor的HAL接口定义在：

hardware/libhardware/include/hardware/sensors.h

```java
/*
 * Sensor string types for Android defined sensor types.
 * For Android defined sensor types, string type will be override in sensor service and thus no
 * longer needed to be added to sensor_t data structure.
 * These definitions are going to be removed soon.
 */
#define SENSOR_STRING_TYPE_ACCELEROMETER                "android.sensor.accelerometer"
#define SENSOR_STRING_TYPE_MAGNETIC_FIELD               "android.sensor.magnetic_field"
#define SENSOR_STRING_TYPE_ORIENTATION                  "android.sensor.orientation"
#define SENSOR_STRING_TYPE_GYROSCOPE                    "android.sensor.gyroscope"
#define SENSOR_STRING_TYPE_LIGHT                        "android.sensor.light"
#define SENSOR_STRING_TYPE_PRESSURE                     "android.sensor.pressure"
#define SENSOR_STRING_TYPE_TEMPERATURE                  "android.sensor.temperature"
#define SENSOR_STRING_TYPE_PROXIMITY                    "android.sensor.proximity"
#define SENSOR_STRING_TYPE_GRAVITY                      "android.sensor.gravity"
#define SENSOR_STRING_TYPE_LINEAR_ACCELERATION          "android.sensor.linear_acceleration"
#define SENSOR_STRING_TYPE_ROTATION_VECTOR              "android.sensor.rotation_vector"
#define SENSOR_STRING_TYPE_RELATIVE_HUMIDITY            "android.sensor.relative_humidity"
#define SENSOR_STRING_TYPE_AMBIENT_TEMPERATURE          "android.sensor.ambient_temperature"
#define SENSOR_STRING_TYPE_MAGNETIC_FIELD_UNCALIBRATED  "android.sensor.magnetic_field_uncalibrated"
#define SENSOR_STRING_TYPE_GAME_ROTATION_VECTOR         "android.sensor.game_rotation_vector"
#define SENSOR_STRING_TYPE_GYROSCOPE_UNCALIBRATED       "android.sensor.gyroscope_uncalibrated"
#define SENSOR_STRING_TYPE_SIGNIFICANT_MOTION           "android.sensor.significant_motion"
#define SENSOR_STRING_TYPE_STEP_DETECTOR                "android.sensor.step_detector"
#define SENSOR_STRING_TYPE_STEP_COUNTER                 "android.sensor.step_counter"
#define SENSOR_STRING_TYPE_GEOMAGNETIC_ROTATION_VECTOR  "android.sensor.geomagnetic_rotation_vector"
#define SENSOR_STRING_TYPE_HEART_RATE                   "android.sensor.heart_rate"
#define SENSOR_STRING_TYPE_TILT_DETECTOR                "android.sensor.tilt_detector"
#define SENSOR_STRING_TYPE_WAKE_GESTURE                 "android.sensor.wake_gesture"
#define SENSOR_STRING_TYPE_GLANCE_GESTURE               "android.sensor.glance_gesture"
#define SENSOR_STRING_TYPE_PICK_UP_GESTURE              "android.sensor.pick_up_gesture"
#define SENSOR_STRING_TYPE_WRIST_TILT_GESTURE           "android.sensor.wrist_tilt_gesture"
#define SENSOR_STRING_TYPE_DEVICE_ORIENTATION           "android.sensor.device_orientation"
#define SENSOR_STRING_TYPE_POSE_6DOF                    "android.sensor.pose_6dof"
#define SENSOR_STRING_TYPE_STATIONARY_DETECT            "android.sensor.stationary_detect"
#define SENSOR_STRING_TYPE_MOTION_DETECT                "android.sensor.motion_detect"
#define SENSOR_STRING_TYPE_HEART_BEAT                   "android.sensor.heart_beat"
#define SENSOR_STRING_TYPE_DYNAMIC_SENSOR_META          "android.sensor.dynamic_sensor_meta"
#define SENSOR_STRING_TYPE_ADDITIONAL_INFO              "android.sensor.additional_info"
#define SENSOR_STRING_TYPE_LOW_LATENCY_OFFBODY_DETECT   "android.sensor.low_latency_offbody_detect"
#define SENSOR_STRING_TYPE_ACCELEROMETER_UNCALIBRATED   "android.sensor.accelerometer_uncalibrated"
#define SENSOR_STRING_TYPE_HINGE_ANGLE                  "android.sensor.hinge_angle"
```

---

## kernel层：

硬件驱动

kernel:

```java
vendor/qcom/proprietary/sensors-see
kernel/msm-4.19/drivers/sensors
kernel/drivers/misc/mediatek/sensor_bio
```

---


```java

```


```java

```


```java

```


```java

```

```java

```



# 参考资料








---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p28-系统开发之Sensor)

---

# 结束语

<img src="../Images/end_001.png">