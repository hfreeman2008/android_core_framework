# P23: 系统开发之浅谈VibratorManagerService

<img src="../flower/flower_p23.png">

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#参考资料)

---


# VibratorManagerService 作用

vibrator管理震动，这个服务比较简单，我们可以参考这个服务来自己做一个服务


---

# 获取 VibratorManagerService 的方式

```java
方式1
VibratorManager vibratorManager = mContext.getSystemService(VibratorManager.class);

方式2
Vibrator mVibrator = (Vibrator) context.getSystemService(Context.VIBRATOR_SERVICE);

方式3
IVibratorManagerService mService = IVibratorManagerService.Stub.asInterface(
        ServiceManager.getService(Context.VIBRATOR_MANAGER_SERVICE));
```

---

# VibratorManagerService 调用流程

![VibratorManagerService_call](VibratorManagerService_call.png)

图一 VibratorManagerService调用流程


以getVibratorIds()为例，查看VibratorManagerService调用流程：

(1)app应用中调用getVibratorIds:

```java
VibratorManager vibratorManager = mContext.getSystemService(VibratorManager.class);
int[] vibratorIds = vibratorManager.getVibratorIds();
```

(2)VibratorManager.java定义getVibratorIds

```java
/**
 * List all available vibrator ids, returning a possible empty list.
 *
 * @return An array containing the ids of the vibrators available on the device.
 */
@NonNull
public abstract int[] getVibratorIds();
```

(3)VibratorManager的子类SystemVibratorManager实现getVibratorIds

```java
public int[] getVibratorIds() {
    synchronized (mLock) {
        if (mVibratorIds != null) {
            return mVibratorIds;
        }
        try {
            if (mService == null) {
                Log.w(TAG, "Failed to retrieve vibrator ids; no vibrator manager service.");
            } else {
                return mVibratorIds = mService.getVibratorIds();
            }
        } catch (RemoteException e) {
            e.rethrowFromSystemServer();
        }
        return new int[0];
    }
}
```

(4)IVibratorManagerService.aidl定义getVibratorIds

```java
int[] getVibratorIds();
VibratorInfo getVibratorInfo(int vibratorId);
boolean isVibrating(int vibratorId);
boolean registerVibratorStateListener(int vibratorId, in IVibratorStateListener listener);
boolean unregisterVibratorStateListener(int vibratorId, in IVibratorStateListener listener);
boolean setAlwaysOnEffect(int uid, String opPkg, int alwaysOnId,
        in CombinedVibration vibration, in VibrationAttributes attributes);
void vibrate(int uid, String opPkg, in CombinedVibration vibration,
        in VibrationAttributes attributes, String reason, IBinder token);
void cancelVibrate(int usageFilter, IBinder token);
```


(5)VibratorManagerService调用getVibratorIds
```java
@Override // Binder call
public int[] getVibratorIds() {
    return Arrays.copyOf(mVibratorIds, mVibratorIds.length);
}
```

---

# 启动 VibratorManagerService 服务：

SystemServer.java

```java
t.traceBegin("StartVibratorManagerService");
mSystemServiceManager.startService(VibratorManagerService.Lifecycle.class);
t.traceEnd();
```


---


# VibratorManagerService类图


![VibratorManagerService类图](VibratorManagerService类图.png)


---

# handler消息

有一个handler:

```java
private final Handler mHandler;
```


---

# dump信息

```java
adb shell dumpsys lights
 
Dumping vibrator manager service to proto...

Dumping vibrator manager service to text...
```


---

# 日志开关

```java
private static final boolean DEBUG = false;
```

---


# framework相关文件：

在frameworks中,涉及到LightsManager的类主要有:

```java

frameworks/base/services/core/java/com/android/server/lights/VibratorManagerService.java
frameworks/base/services/core/java/com/android/server/BatteryService.java
NotificationManagerService.java
PowerManagerService.java
LocalDisplayAdapter.java
```

---

# 相机界面屏蔽所有震动提醒

在doVibratorOn方法中添加一个白名单，以避免震动

```java
// add begin
// Module Vibrator
//add black activity list for vibrator
import android.app.Activity;
import android.content.ComponentName;
//  add end


// add begin
// Module Vibrator
// add black activity list for vibrator
private String getTopActivityName() {
String topActivityName = "";
//get top activity name
ActivityManager am = (ActivityManager) mContext.getSystemService(Activity.ACTIVITY_SERVICE);
if(am != null
&& am.getRunningTasks(1) != null
&& am.getRunningTasks(1).size() > 0
&& am.getRunningTasks(1).get(0) != null) {
ComponentName comp = am.getRunningTasks(1).get(0).topActivity;
if(comp != null) {
    topActivityName = comp.getClassName();
}
}
return topActivityName;
}

private String getTopActivityPkgName() {
    String topPkgName = "";
    //get top activity package name
    ActivityManager am = (ActivityManager) mContext.getSystemService(Activity.ACTIVITY_SERVICE);
    if(am != null
    && am.getRunningTasks(1) != null
    && am.getRunningTasks(1).size() > 0
    && am.getRunningTasks(1).get(0) != null) {
        ComponentName comp = am.getRunningTasks(1).get(0).topActivity;
    if(comp != null) {
        topPkgName = comp.getPackageName();
    }
    }
    return topPkgName;
}

private String topActivityPkgNameTemp = "";
private String topActivityNameTemp = "";

private boolean isVibratorBlackActivity() {
    boolean result = false;
    topActivityPkgNameTemp = getTopActivityPkgName();
    topActivityNameTemp = getTopActivityName();
    if (DEBUG) {
        Slog.d(TAG, "topActivityPkgNameTemp:" + topActivityPkgNameTemp);
        Slog.d(TAG, "topActivityNameTemp:" + topActivityNameTemp);
    }
    if("com.android.camera".equals(topActivityPkgNameTemp)
    && "com.android.camera.CameraActivity".equals(topActivityNameTemp)){
        Slog.d(TAG, "Turning off vibrator when current activity in black list");
        result = true;
    }
    return result;
}
//  add end


private void doVibratorOn(long millis, int amplitude, int uid, AudioAttributes attrs) {
Trace.traceBegin(Trace.TRACE_TAG_VIBRATOR, "doVibratorOn");
try {
synchronized (mInputDeviceVibrators) {
if (amplitude == VibrationEffect.DEFAULT_AMPLITUDE) {
amplitude = mDefaultVibrationAmplitude;
}

//  add begin
// Module Vibrator
// add black activity list for vibrator
if(isVibratorBlackActivity()){
    return;
}
//  add end

if (DEBUG) {
Slog.d(TAG, "Turning vibrator on for " + millis + " ms" +
" with amplitude " + amplitude + ".");
}

```

---


# JNI

com_android_server_lights_VibratorManagerService.cpp

jni调用本地接口:

```java
static native void setLight_native(int light, int color, int mode,
        int onMS, int offMS, int brightnessMode);
```

frameworks/base/services/core/jni/com_android_server_lights_VibratorManagerService.cpp

```java
static void setLight_native(
        JNIEnv* /* env */,
        jobject /* clazz */,
        jint light,
        jint colorARGB,
        jint flashMode,
        jint onMS,
        jint offMS,
        jint brightnessMode) {

    if (!validate(light, flashMode, brightnessMode)) {
```



```java
static const JNINativeMethod method_table[] = {
    { "setLight_native", "(IIIIII)V", (void*)setLight_native },
};

int register_android_server_VibratorManagerService(JNIEnv *env) {
    return jniRegisterNativeMethods(env, "com/android/server/lights/VibratorManagerService",
            method_table, NELEM(method_table));
}
```

---

# HAL


```java
hardware\qcom\display\liblight
hardware/qcom/display/liblight/lights.c

hardware/interfaces/light
```

高通项目如何定位hal源码：
```java
adb shell ps -Z | findstr -i light
u:r:hal_light_default:s0       system          806      1 12347756  2652 0                   0 S android.hardware.lights-service.qti

out/target/product/bengal/vendor/bin/hw/android.hardware.lights-service.qti

搜索：
android.hardware.lights-service.qti
高通项目：
device\qcom\vendor-common\lights\Android.bp

device\qcom\vendor-common\lights\
  目录文件不多：
Android.bp
android.hardware.lights-qti.rc
android.hardware.lights-qti.xml
Lights.cpp
Lights.h
main.cpp


搜索关键的：
hw_get_module (LIGHTS_HARDWARE_MODULE_ID
可以定位到lib库
```


---

# liblight库

高通：
```java
./out/target/product/bengal/vendor/lib64/hw/lights.bengal.so
./out/target/product/bengal/vendor/lib/hw/lights.bengal.so

hardware/qcom/display/liblight/lights.c

hardware/qcom/display/liblight/ 目录文件：
Android.mk
lights.c
NOTICE
```

lights.c

- 节点信息：
```java
char const*const LCD_FILE
        = "/sys/class/leds/lcd-backlight/brightness";

char const*const LCD_FILE2
        = "/sys/class/backlight/panel0-backlight/brightness";

char const*const BUTTON_FILE
        = "/sys/class/leds/button-backlight/brightness";

char const*const PERSISTENCE_FILE
        = "/sys/class/graphics/fb0/msm_fb_persist_mode";


echo 0 >  /sys/class/leds/blue0/brightness
```


- 关键方法

```java
static struct hw_module_methods_t lights_module_methods = {
    .open =  open_lights,
};

write_str

写节点亮度：
set_rgb_led_brightness
snprintf(file, sizeof(file), "/sys/class/leds/%s/brightness", led_names[led]);

写节点闪烁
static int set_rgb_led_timer_trigger(enum rgb_led led, int onMS, int offMS)
    snprintf(file_on, sizeof(file_on), "/sys/class/leds/%s/trigger", led_names[led]);
    snprintf(file_off, sizeof(file_off), "/sys/class/leds/%s/delay_off", led_names[led]);
    snprintf(file_on, sizeof(file_on), "/sys/class/leds/%s/delay_on", led_names[led]);
```


---

# kernel

```java
kernel\drivers\leds
kernel/msm-4.9/drivers/leds
kernel/msm-4.19/drivers/leds/leds-aw2016.c
```

light驱动节点：
```java
/sys/class/leds/
```

节点文件：
蓝，绿，红，三个节点：

![note_3](note_3.png)

red0目录下的节点：

![note_red0](note_red0.png)




查看一个实现文件：
kernel\msm-4.19\drivers\iio\light\Makefile

```makefile
obj-$(CONFIG_ZOPT2201)      += zopt2201.o
```

kernel\msm-4.19\drivers\iio\light\Kconfig
```makefile
config ZOPT2201
    tristate "ZOPT2201 ALS and UV B sensor"
    depends on I2C
    help
     Say Y here if you want to build a driver for the IDT
     ZOPT2201 ambient light and UV B sensor.

     To compile this driver as a module, choose M here: the
     module will be called zopt2201.
```


kernel\msm-4.19\drivers\iio\light\zopt2201.c

```c
static struct i2c_driver zopt2201_driver = {
    .driver = {
        .name   = ZOPT2201_DRV_NAME,
    },
    .probe  = zopt2201_probe, //注意这个文件
    .id_table = zopt2201_id,
};

static int zopt2201_probe(struct i2c_client *client,
              const struct i2c_device_id *id)
{
    ......
}
```




---

# FAQ

## 相关的命令

手动调整背光亮度的命令：

```shell
adb root && adb remount
adb shell
echo 255 > /sys/class/leds/wled/brightness
```

屏幕亮度查看命令：

```shell
adb root
cat /sys/class/leds/wled/brightness
```

---

## leds brightness(led亮度)

LED灯--充电时，充电指示灯太刺眼，影响体验，减少LED灯的亮度：
kernel/msm-4.9/drivers/leds/leds-qpnp.c
```c
    /* START<fix><demand 188 ><for leds brightness ><201901014>> */
    switch(led->id){
    case QPNP_ID_RGB_RED:
    case QPNP_ID_RGB_GREEN:
    case QPNP_ID_RGB_BLUE:
    led->cdev.brightness = value >> 3;
    break;
    }
    /* end<fix><demand 188 ><for leds brightness ><201901014>> */
```



---




---



---




---

# 参考资料

1.VibratorManagerService 灯光控制
https://blog.csdn.net/u012932409/article/details/108345928

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p22-系统开发之浅谈VibratorManagerService)

---

# 结束语

<img src="../Images/end_001.png">
