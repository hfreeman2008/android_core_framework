# P16_系统开发之浅谈PowerManagerService

<img src="../flower/flower_p16.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

# PowerManagerService 类的作用：

主要是处理与电源有关的逻辑,包括手机亮度管理,手机锁屏,节能锁屏,手机低电量模式,手机睡眠,手机双击亮屏.

---


# 获取PowerManagerService的方式：

```java
方式1
PowerManager powerManager = (PowerManager)context.getSystemService(Context.POWER_SERVICE);

方式2
IPowerManager pm = IPowerManager.Stub.asInterface(
        ServiceManager.getService(Context.POWER_SERVICE));

方式3 (system server进程使用)
PowerManagerInternal mLocalPowerManager = getLocalService(PowerManagerInternal.class);
```



# PowerManagerService 调用流程

---

<img src="PowerManagerService_whole.png">

图一 PowerManagerService调用流程

PowerManagerService调用流程和其他的服务完全是一样的，这部分具体讲解就不说了。

---

# 启动PowerManagerService服务：



SystemServer.java

```java
t.traceBegin("StartPowerManager");
mPowerManagerService = mSystemServiceManager.startService(PowerManagerService.class);
t.traceEnd();

......

t.traceBegin("MakePowerManagerServiceReady");
try {
    // TODO: use boot phase
    mPowerManagerService.systemReady(mActivityManagerService.getAppOpsService());
} catch (Throwable e) {
    reportWtf("making Power Manager Service ready", e);
}
t.traceEnd();
```

---


# 注册 PowerManagerService 服务：

SystemServiceRegistry.java
```java
registerService(Context.POWER_SERVICE, PowerManager.class,
        new CachedServiceFetcher<PowerManager>() {
    @Override
    public PowerManager createService(ContextImpl ctx) throws ServiceNotFoundException {
        IBinder powerBinder = ServiceManager.getServiceOrThrow(Context.POWER_SERVICE);
        IPowerManager powerService = IPowerManager.Stub.asInterface(powerBinder);
        IBinder thermalBinder = ServiceManager.getServiceOrThrow(Context.THERMAL_SERVICE);
        IThermalService thermalService = IThermalService.Stub.asInterface(thermalBinder);
        return new PowerManager(ctx.getOuterContext(), powerService, thermalService,
                ctx.mMainThread.getHandler());
    }});
```

---

# PowerManagerService 类图

<img src="PowerManagerService_class.png">
图三 PowerManagerService类图




---

# handler消息

有一个handler:
```java
Handler mHandler;
```

消息列表：
```java
// Message: Sent when a user activity timeout occurs to update the power state.
private static final int MSG_USER_ACTIVITY_TIMEOUT = 1;
// Message: Sent when the device enters or exits a dreaming or dozing state.
private static final int MSG_SANDMAN = 2;
// Message: Sent when the screen brightness boost expires.
private static final int MSG_SCREEN_BRIGHTNESS_BOOST_TIMEOUT = 3;
// Message: Polling to look for long held wake locks.
private static final int MSG_CHECK_FOR_LONG_WAKELOCKS = 4;
// Message: Sent when an attentive timeout occurs to update the power state.
private static final int MSG_ATTENTIVE_TIMEOUT = 5;
```

---

# dump信息

命令：
```java
adb shell dumpsys power
```
相关方法：
```java
dumpInternal(PrintWriter pw)
dumpProto(FileDescriptor fd) 
```

---

# 日志开关：

```java
private static final boolean DEBUG = true;//false;
private static final boolean DEBUG_SPEW = DEBUG && true;
```

---


# 读取配置参数--readConfigurationLocked：

```java
mDecoupleHalAutoSuspendModeFromDisplayConfig = resources.getBoolean(
        com.android.internal.R.bool.config_powerDecoupleAutoSuspendModeFromDisplay);
mDecoupleHalInteractiveModeFromDisplayConfig = resources.getBoolean(
        com.android.internal.R.bool.config_powerDecoupleInteractiveModeFromDisplay);
mWakeUpWhenPluggedOrUnpluggedConfig = resources.getBoolean(
        com.android.internal.R.bool.config_unplugTurnsOnScreen);  ----是否插拔亮屏
mWakeUpWhenPluggedOrUnpluggedInTheaterModeConfig = resources.getBoolean(
        com.android.internal.R.bool.config_allowTheaterModeWakeFromUnplug);
mSuspendWhenScreenOffDueToProximityConfig = resources.getBoolean(
        com.android.internal.R.bool.config_suspendWhenScreenOffDueToProximity);
mAttentiveTimeoutConfig = resources.getInteger(
        com.android.internal.R.integer.config_attentiveTimeout);
mAttentiveWarningDurationConfig = resources.getInteger(
        com.android.internal.R.integer.config_attentiveWarningDuration);  
mDreamsSupportedConfig = resources.getBoolean(
        com.android.internal.R.bool.config_dreamsSupported);  ----dreams是否支持
mDreamsEnabledByDefaultConfig = resources.getBoolean(
        com.android.internal.R.bool.config_dreamsEnabledByDefault);
mDreamsActivatedOnSleepByDefaultConfig = resources.getBoolean(
        com.android.internal.R.bool.config_dreamsActivatedOnSleepByDefault);
mDreamsActivatedOnDockByDefaultConfig = resources.getBoolean(
        com.android.internal.R.bool.config_dreamsActivatedOnDockByDefault);
mDreamsEnabledOnBatteryConfig = resources.getBoolean(
        com.android.internal.R.bool.config_dreamsEnabledOnBattery);
mDreamsBatteryLevelMinimumWhenPoweredConfig = resources.getInteger(
        com.android.internal.R.integer.config_dreamsBatteryLevelMinimumWhenPowered);
mDreamsBatteryLevelMinimumWhenNotPoweredConfig = resources.getInteger(
        com.android.internal.R.integer.config_dreamsBatteryLevelMinimumWhenNotPowered);
mDreamsBatteryLevelDrainCutoffConfig = resources.getInteger(
        com.android.internal.R.integer.config_dreamsBatteryLevelDrainCutoff);
mDozeAfterScreenOff = resources.getBoolean(
        com.android.internal.R.bool.config_dozeAfterScreenOffByDefault);
mMinimumScreenOffTimeoutConfig = resources.getInteger(
        com.android.internal.R.integer.config_minimumScreenOffTimeout); ----息屏最小时间
mMaximumScreenDimDurationConfig = resources.getInteger(
        com.android.internal.R.integer.config_maximumScreenDimDuration);
mMaximumScreenDimRatioConfig = resources.getFraction(
        com.android.internal.R.fraction.config_maximumScreenDimRatio, 1, 1);
mSupportsDoubleTapWakeConfig = resources.getBoolean(
        com.android.internal.R.bool.config_supportDoubleTapWake);  ----是否支持双击亮屏
```

---

# 内容监听:

```java
mSettingsObserver = new SettingsObserver(mHandler);

// Register for settings changes.
resolver.registerContentObserver(Settings.Secure.getUriFor(
        Settings.Secure.SCREENSAVER_ENABLED),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.Secure.getUriFor(
        Settings.Secure.SCREENSAVER_ACTIVATE_ON_SLEEP),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.Secure.getUriFor(
        Settings.Secure.SCREENSAVER_ACTIVATE_ON_DOCK),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.System.getUriFor(
        Settings.System.SCREEN_OFF_TIMEOUT),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.Secure.getUriFor(
        Settings.Secure.SLEEP_TIMEOUT),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.Secure.getUriFor(
        Settings.Secure.ATTENTIVE_TIMEOUT),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.Global.getUriFor(
        Settings.Global.STAY_ON_WHILE_PLUGGED_IN),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.System.getUriFor(
        Settings.System.SCREEN_BRIGHTNESS_MODE),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.System.getUriFor(
        Settings.System.SCREEN_AUTO_BRIGHTNESS_ADJ),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.Global.getUriFor(
        Settings.Global.THEATER_MODE_ON),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.Secure.getUriFor(
        Settings.Secure.DOZE_ALWAYS_ON),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.Secure.getUriFor(
        Settings.Secure.DOUBLE_TAP_TO_WAKE),
        false, mSettingsObserver, UserHandle.USER_ALL);
resolver.registerContentObserver(Settings.Global.getUriFor(
        Settings.Global.DEVICE_DEMO_MODE),
        false, mSettingsObserver, UserHandle.USER_SYSTEM);
```



```java
private final class SettingsObserver extends ContentObserver {
    @Override
    public void onChange(boolean selfChange, Uri uri) {
        synchronized (mLock) {
            handleSettingsChangedLocked();
```


---

# incrementBootCount---设备启动次数

```java
private void incrementBootCount() {
    synchronized (mLock) {
        int count;
        try {
            count = Settings.Global.getInt(
                    getContext().getContentResolver(), Settings.Global.BOOT_COUNT);
        } catch (SettingNotFoundException e) {
            count = 0;
        }
        Settings.Global.putInt(
                getContext().getContentResolver(), Settings.Global.BOOT_COUNT, count + 1);
    }
}
```

```java
adb shell settings get global boot_count
```


---

# publishBinderService--Context.POWER_SERVICE

```java
private final class BinderService extends IPowerManager.Stub {
......
}


private final BinderService mBinderService;

mBinderService = new BinderService();

BinderService getBinderServiceInstance() {
    return mBinderService;
}

public void onStart() {
    publishBinderService(Context.POWER_SERVICE, mBinderService, /* allowIsolated= */ false,
            DUMP_FLAG_PRIORITY_DEFAULT | DUMP_FLAG_PRIORITY_CRITICAL);

```

读取服务的方法：
```java
PowerManager powerManager = (PowerManager)context.getSystemService(Context.POWER_SERVICE);
```


---

# LocalService--PowerManagerInternal


```java
private final class LocalService extends PowerManagerInternal {
    ......    
}


private final LocalService mLocalService;

mLocalService = new LocalService();

LocalService getLocalServiceInstance() {
    return mLocalService;
}

public void onStart() {
    publishLocalService(PowerManagerInternal.class, mLocalService);
}

```

system_server读取服务：
```java
private PowerManagerInternal mLocalPowerManager;
mLocalPowerManager = LocalServices.getService(PowerManagerInternal.class);
```

---

# 相关的系统属性

```java
adb shell getprop sys.boot.reason      设备启动的原因

private static final String REASON_SHUTDOWN = "shutdown";
private static final String REASON_REBOOT = "reboot";
private static final String REASON_USERREQUESTED = "shutdown,userrequested";
private static final String REASON_THERMAL_SHUTDOWN = "shutdown,thermal";
private static final String REASON_LOW_BATTERY = "shutdown,battery";
private static final String REASON_BATTERY_THERMAL_STATE = "shutdown,thermal,battery";
```



---

# 低电量关机或重启

```java
shutdownOrRebootInternal
lowLevelShutdown
lowLevelReboot
```

---

# native方法--nativeSetInteractive为例:

定义nativeSetInteractive:
```java
private static native void nativeSetInteractive(boolean enable);
```

调用:
```java
public PowerManagerService(Context context) {
    ...........
    mNativeWrapper.nativeSetInteractive(true);
}

public void nativeSetInteractive(boolean enable) {
    PowerManagerService.nativeSetInteractive(enable);
}
```

frameworks\base\services\core\jni\com_android_server_power_PowerManagerService.cpp
```java
static void nativeSetInteractive(JNIEnv* /* env */, jclass /* clazz */, jboolean enable) {
    std::unique_lock<std::mutex> lock(gPowerHalMutex);
    switch (connectPowerHalLocked()) {
        case HalVersion::NONE:
            return;
        case HalVersion::HIDL_1_0:
            FALLTHROUGH_INTENDED;
        case HalVersion::HIDL_1_1: {
            android::base::Timer t;
            sp<IPowerV1_0> handle = gPowerHalHidlV1_0_;
            lock.unlock();
            auto ret = handle->setInteractive(enable);
            processPowerHalReturn(ret.isOk(), "setInteractive");
            if (t.duration() > 20ms) {
                ALOGD("Excessive delay in setInteractive(%s) while turning screen %s",
                      enable ? "true" : "false", enable ? "on" : "off");
            }
            return;
        }
        case HalVersion::AIDL: {
            sp<IPowerAidl> handle = gPowerHalAidl_;
            lock.unlock();
            setPowerModeWithHandle(handle, Mode::INTERACTIVE, enable);
            return;
        }
        default: {
            ALOGE("Unknown power HAL state");
            return;
        }
    }
}



static const JNINativeMethod gPowerManagerServiceMethods[] = {
        /* name, signature, funcPtr */
        {"nativeInit", "()V", (void*)nativeInit},
        {"nativeAcquireSuspendBlocker", "(Ljava/lang/String;)V",
         (void*)nativeAcquireSuspendBlocker},
        {"nativeForceSuspend", "()Z", (void*)nativeForceSuspend},
        {"nativeReleaseSuspendBlocker", "(Ljava/lang/String;)V",
         (void*)nativeReleaseSuspendBlocker},
        {"nativeSetInteractive", "(Z)V", (void*)nativeSetInteractive},
```

注册方法

```java
int register_android_server_PowerManagerService(JNIEnv* env) {
    int res = jniRegisterNativeMethods(env, "com/android/server/power/PowerManagerService",
            gPowerManagerServiceMethods, NELEM(gPowerManagerServiceMethods));
    (void) res;  // Faked use when LOG_NDEBUG.
    LOG_FATAL_IF(res < 0, "Unable to register native methods.");

    // Callbacks

    jclass clazz;
    FIND_CLASS(clazz, "com/android/server/power/PowerManagerService");

    GET_METHOD_ID(gPowerManagerServiceClassInfo.userActivityFromNative, clazz,
            "userActivityFromNative", "(JII)V");

    // Initialize
    for (int i = 0; i <= USER_ACTIVITY_EVENT_LAST; i++) {
        gLastEventTime[i] = LLONG_MIN;
    }
    gPowerManagerServiceObj = NULL;
    return 0;
}
```

---

# 按电源键快速亮灭屏

```java
private boolean wakeUpNoUpdateLocked(long eventTime, @WakeReason int reason, String details,
int reasonUid, String opPackageName, int opUid) {
    if (DEBUG_SPEW) {
        Slog.d(TAG, "wakeUpNoUpdateLocked: eventTime=" + eventTime + ", uid=" + reasonUid);
    }

    if (eventTime < mLastSleepTime || mWakefulness == WAKEFULNESS_AWAKE
    || !mBootCompleted || !mSystemReady || mForceSuspendActive) {
        return false;
    }

    Trace.asyncTraceBegin(Trace.TRACE_TAG_POWER, TRACE_SCREEN_ON, 0);

    Trace.traceBegin(Trace.TRACE_TAG_POWER, "wakeUp");
    try {
        Slog.i(TAG, "Waking up from "
        + PowerManagerInternal.wakefulnessToString(mWakefulness)
        + " (uid=" + reasonUid
        + ", reason=" + PowerManager.wakeReasonToString(reason)
        + ", details=" + details
        + ")...");
        //optimize screen on/off time
        if("android.policy:POWER".equals(details)) {
            setPerfLock();
        }
        //optimize screen on/off time

        //optimize screen on/off time
        private void setPerfLock() {
            int aBoostTimeOut = 300;
            int aBoostParamVal[] = {0x40C00000, 0x1, 0x40804000, 0xFFF, 0x40804100, 0xFFF, 0x40800000, 0xFFF, 0x40800100, 0xFFF};
            mPerf = new BoostFramework();
            if(mPerf != null) {
                mPerf.perfLockAcquire(aBoostTimeOut, aBoostParamVal);
            }
        }
        //optimize screen on/off time
```


---

# 电源管理架构

PowerManagerService在Framework层本质为策略控制方案，其作用为：

- 向上提供给应用程序接口，例如音频场景中保持系统唤醒、消息通知中唤醒手机屏幕场景；
- 向下决策HAL层以及Kernel层来控制设备待机状态，控制显示屏、背光灯、距离传感器、光线传感器等硬件设备的状态；

<img src="../Images/power_whole.png">


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p16_系统开发之浅谈powermanagerservice)

---


# 结束语

<img src="../Images/end_001.png">
