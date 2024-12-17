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