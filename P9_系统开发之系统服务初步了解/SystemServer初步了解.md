# SystemServer初步了解

<img src="../flower/flower_one.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

[返回 P8_系统开发之自定义系统服务](https://github.com/hfreeman2008/android_core_framework/blob/main/P8_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1.md)

---

# SystemServer介绍

frameworks\base\services\java\com\android\server\SystemServer.java

SystemServer类主要是system_server的主入口

![system_server_启动流程](./image/system_server_启动流程.png)

---

# 查看Service列表

查看系统服务列表：
```bash
adb shell service list

Found 1 services:
0 phone: [com.android.internal.telephony.ITelephony]
```

检查Service是否存在：
```bash
adb shell service check phone
Service phone: found
```


使用Service：
```bash
adb shell service call phone 2 s16 "10086"
```

---

# 如何获取正在运行的服务列表

如何获取正在运行的服务列表
```bash
// List all services
adb shell dumpsys activity services
```


---

# run()

如果persist.sys.timezone没有设置,那我们设置其为GMT

```java
//
// Default the timezone property to GMT if not set.
//
String timezoneProperty =  SystemProperties.get("persist.sys.timezone");
if (timezoneProperty == null || timezoneProperty.isEmpty()) {
    Slog.w(TAG, "Timezone not set; setting to GMT.");
    SystemProperties.set("persist.sys.timezone", "GMT");
}
```

如果persist.sys.language为空,则读取当地的语言,再设置:

```java
// If the system has "persist.sys.language" and friends set, replace them with
// "persist.sys.locale". Note that the default locale at this point is calculated
// using the "-Duser.locale" command line flag. That flag is usually populated by
// AndroidRuntime using the same set of system properties, but only the system_server
// and system apps are allowed to set them.
//
// NOTE: Most changes made here will need an equivalent change to
// core/jni/AndroidRuntime.cpp
if (!SystemProperties.get("persist.sys.language").isEmpty()) {
    final String languageTag = Locale.getDefault().toLanguageTag();

    SystemProperties.set("persist.sys.locale", languageTag);
    SystemProperties.set("persist.sys.language", "");
    SystemProperties.set("persist.sys.country", "");
    SystemProperties.set("persist.sys.localevar", "");
}
```

导入android_servers库

```java
// Initialize native services.
System.loadLibrary("android_servers");
```

---

# 启动各种服务


```java
startBootstrapServices();
startCoreServices();
startOtherServices();
```

---

## startBootstrapServices-启动开机的引导服务


```java
SystemServerInitThreadPool.get().submit(SystemConfig::getInstance, TAG_SYSTEM_CONFIG);

Installer installer = mSystemServiceManager.startService(Installer.class);

mSystemServiceManager.startService(DeviceIdentifiersPolicyService.class);

mActivityManagerService = mSystemServiceManager.startService(
        ActivityManagerService.Lifecycle.class).getService();
mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
mActivityManagerService.setInstaller(installer);

mPowerManagerService = mSystemServiceManager.startService(PowerManagerService.class);

mActivityManagerService.initPowerManagement();

........
mSystemServiceManager.startService(RecoverySystemService.class);

mSystemServiceManager.startService(LightsService.class);

mDisplayManagerService = mSystemServiceManager.startService(DisplayManagerService.class);


if (RegionalizationEnvironment.isSupported()) {
    Slog.i(TAG, "Regionalization Service");
    RegionalizationService regionalizationService = new RegionalizationService();
    ServiceManager.addService("regionalization", regionalizationService);
}


mPackageManagerService = PackageManagerService.main(mSystemContext, installer,
        mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore);
mFirstBoot = mPackageManagerService.isFirstBoot();
mPackageManager = mSystemContext.getPackageManager();


mSystemServiceManager.startService(UserManagerService.LifeCycle.class);


mActivityManagerService.setSystemProcess();

mSystemServiceManager.startService(new OverlayManagerService(mSystemContext, installer));
```


## startCoreServices-启动核心服务


```java
mSystemServiceManager.startService(DropBoxManagerService.class);

mSystemServiceManager.startService(BatteryService.class);

mSystemServiceManager.startService(UsageStatsService.class);
mActivityManagerService.setUsageStatsManager(
        LocalServices.getService(UsageStatsManagerInternal.class));

mWebViewUpdateService = mSystemServiceManager.startService(WebViewUpdateService.class);
```



```bash

```


```bash

```



---

```bash

```

```bash

```

---




# 结束语




---


[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#systemserver初步了解)

---

[返回 P8_系统开发之自定义系统服务](https://github.com/hfreeman2008/android_core_framework/blob/main/P8_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1.md)



---


<img src="../Images/end_001.png">