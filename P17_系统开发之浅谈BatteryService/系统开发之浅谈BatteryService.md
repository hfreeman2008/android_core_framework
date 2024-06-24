# P17_系统开发之浅谈BatteryService

<img src="../flower/flower_p17.png">


---

# BatteryService 类的作用：

该服务继承SystemService，主要用于管理 电池的充电状态，充电百分比，监视当前电量的状态,并根据当前的电量状态进行相应的逻辑操作.

如:处理电源的led灯,是否是低电量模式,电池的温度过高时关机,电池的电量为0时关机,把电池的状态同步发送广播通知给系统, 

---

# 获取BatteryService的方式：

```java
方式1
BatteryManager batteryManager =(BatteryManager) context.getSystemService(Context.BATTERY_SERVICE);

方式2
BatteryManager mBatteryManager = mContext.getSystemService(BatteryManager.class);

方式3
IBatteryStats mBatteryStats = IBatteryStats.Stub.asInterface(ServiceManager.getService(BatteryStats.SERVICE_NAME));

方式4 (system server进程使用)
BatteryManagerInternal batteryManagerInternal = LocalServices.getService(BatteryManagerInternal.class);
```

---

# BatteryService 调用流程

<img src="BatteryService_whole.png">
图一 BatteryManager调用流程

PowerManagerService调用流程和其他的服务完全是一样的，这部分具体讲解就不说了。

---

# 启动BatteryService服务：
SystemServer.java


```java
t.traceBegin("StartBatteryService");
// Tracks the battery level.  Requires LightService.
mSystemServiceManager.startService(BatteryService.class);
t.traceEnd();
```

---

# 注册BatteryService服务：
SystemServiceRegistry.java

```java
registerService(Context.BATTERY_SERVICE, BatteryManager.class,
        new CachedServiceFetcher<BatteryManager>() {
    @Override
    public BatteryManager createService(ContextImpl ctx) throws ServiceNotFoundException {
        IBatteryStats stats = IBatteryStats.Stub.asInterface(
                ServiceManager.getServiceOrThrow(BatteryStats.SERVICE_NAME));
        IBatteryPropertiesRegistrar registrar = IBatteryPropertiesRegistrar.Stub
                .asInterface(ServiceManager.getServiceOrThrow("batteryproperties"));
        return new BatteryManager(ctx, stats, registrar);
    }});
```

---

# BatteryService 类图

<img src="BatteryService_class.png">
图三 BatteryService类图

---

# handler消息

有一个handler:
```java
private final Handler mHandler;
```

---

# dump信息

查看电池信息

```java
adb shell dumpsys battery
adb shell dumpsys batterystats
```

```java
Current Battery Service state:
  AC powered: false
  USB powered: true
  Wireless powered: false
  Max charging current: 200000
  Max charging voltage: 5000000
  Charge counter: 2963700
  status: 2
  health: 2
  present: true
  level: 90
  scale: 100
  voltage: 4167
  temperature: 354
  technology: Li-ion
```


---

# 日志开关：


```java
private static final boolean DEBUG = false;
```

---

# publishBinderService--battery



```java
private final class BinderService extends Binder

public void onStart() {
    mBinderService = new BinderService();
    publishBinderService("battery", mBinderService);
}

BinderService mBinderService;
```

读取服务的方法：

```java
BatteryManager batteryManager =(BatteryManager) context.getSystemService(Context.BATTERY_SERVICE);
```

---

# publishBinderService--batteryproperties

```java
BatteryPropertiesRegistrar mBatteryPropertiesRegistrar;

private final class BatteryPropertiesRegistrar extends IBatteryPropertiesRegistrar.Stub {
......    
}

public void onStart() {
    mBatteryPropertiesRegistrar = new BatteryPropertiesRegistrar();
    publishBinderService("batteryproperties", mBatteryPropertiesRegistrar);
}
```

读取服务的方法：

```java
IBatteryPropertiesRegistrar registrar = IBatteryPropertiesRegistrar.Stub.asInterface(
        ServiceManager.getService("batteryproperties"));
```

---

# LocalService--BatteryManagerInternal


```java
private final class LocalService extends BatteryManagerInternal {
}

public void onStart() {
    publishLocalService(BatteryManagerInternal.class, new LocalService());
}
```

system_server读取服务：
```java
BatteryManagerInternal batteryManagerInternal = LocalServices.getService(BatteryManagerInternal.class);
```

---

# 电源连接类型
```java
private int mPlugType;

mPlugType = BatteryManager.BATTERY_PLUGGED_AC;
mPlugType = BatteryManager.BATTERY_PLUGGED_USB;
mPlugType = BatteryManager.BATTERY_PLUGGED_WIRELESS;
mPlugType = BATTERY_PLUGGED_NONE;
```

---

# processValuesLocked日志

```java
BatteryService: Processing new values: info={.chargerAcOnline = false, .chargerUsbOnline = true, .chargerWirelessOnline = false, .maxChargingCurrent = 500000, .maxChargingVoltage = 5000000, .batteryStatus = CHARGING, .batteryHealth = GOOD, .batteryPresent = true, .batteryLevel = 100, .batteryVoltage = 4399, .batteryTemperature = 280, .batteryCurrent = 138, .batteryCycleCount = 6, .batteryFullCharge = 3666000, .batteryChargeCounter = 3482700, .batteryTechnology = Li-ion}, mBatteryLevelCritical=false, mPlugType=2, mOemChargerType=-1, mChargingCurrent=125122, mBoardTemperature=348, mUsbChargingVoltage =0
```

```java
processValuesLocked方法
if (DEBUG) {
    Slog.d(TAG, "Processing new values: "
    + "info=" + mHealthInfo
    + ", mBatteryLevelCritical=" + mBatteryLevelCritical
    + ", mPlugType=" + mPlugType);
}
```

---

# 是否是低电量模式

```java
private boolean mBatteryLevelLow;

if (!mBatteryLevelLow) {
    // Should we now switch in to low battery mode?
    //如果没有连接电池,并且当前的电量少于低电量值(20)
    if (mPlugType == BATTERY_PLUGGED_NONE
            && mBatteryProps.batteryLevel <= mLowBatteryWarningLevel) {
        mBatteryLevelLow = true;
    }
```

低电量提醒：
```java
mLowBatteryWarningLevel = mContext.getResources().getInteger(
        com.android.internal.R.integer.config_lowBatteryWarningLevel);
        
mLowBatteryWarningLevel = Settings.Global.getInt(resolver,
        Settings.Global.LOW_POWER_MODE_TRIGGER_LEVEL, defWarnLevel);
```

frameworks\base\core\res\res\values\config.xml
```xml
<!-- Display low battery warning when battery level dips to this value -->
<integer name="config_lowBatteryWarningLevel">20</integer>
```



```java
adb shell settings get global low_power_mode_trigger_level
```

---

# 关闭低电量提醒


```java
private int mLowBatteryCloseWarningLevel;

mLowBatteryCloseWarningLevel = mLowBatteryWarningLevel + mContext.getResources().getInteger(
        com.android.internal.R.integer.config_lowBatteryCloseWarningBump);
```

```java
<!-- Close low battery warning when battery level reaches the lowBatteryWarningLevel
     plus this -->
<integer name="config_lowBatteryCloseWarningBump">5</integer>
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

# 电源架构

<img src="../Images/power_whole.png">


---


# 结束语

<img src="../Images/end_001.png">