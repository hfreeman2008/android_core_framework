# P38: 系统开发之permission PermissionManager

<img src="../flower/flower_p29.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

[返回 P38_系统开发之permission](https://github.com/hfreeman2008/android_core_framework/blob/main/P38_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bpermission/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bpermission.md)


---


# PermissionManager主要功能

‌1. 运行时权限管理‌

提供动态权限状态检查、申请及跟踪能力，兼容 Android 6.0（API 23）及以上版本的运行时权限模型。

管理标准权限（如摄像头、定位）和自定义权限，支持细粒度权限控制。

权限声明检查：可以查看应用在 AndroidManifest.xml 文件中声明的权限


‌2. 权限组状态监控‌

监控权限组（如 STORAGE、LOCATION）的整体授权状态，当组内任一权限被用户修改时触发回调。

‌3. 权限使用跟踪‌

记录权限使用历史（如权限请求次数、拒绝/授予时间），用于生成权限使用报告或合规性审计。

‌4. 特殊权限管理‌

处理系统级特殊权限（如 SCHEDULE_EXACT_ALARM、MANAGE_EXTERNAL_STORAGE），需通过特定流程申请。

---

# 相关文件

frameworks\base\core\java\android\permission\PermissionManager.java
frameworks\base\services\core\java\com\android\server\pm\permission\PermissionManagerService.java
frameworks\base\core\java\android\permission\PermissionManagerInternal.java
frameworks\base\services\core\java\com\android\server\pm\permission\PermissionManagerServiceInternal.java




---

# 关键接口与方法
‌
## 实例获取‌

```java
PermissionManager permissionManager = getSystemService(PermissionManager.class);
```

## checkPermission-权限状态检查‌
‌
checkPermission()‌   检查单个权限的当前状态：

```java
int result = permissionManager.checkPermission(Manifest.permission.CAMERA, packageName, userId);
// 返回值为 PERMISSION_GRANTED 或 PERMISSION_DENIED
```
注意‌：需 INTERACT_ACROSS_USERS 权限才能跨用户检查。

```java
int permissionStatus = permissionManager.checkPermission(
    Manifest.permission.CAMERA, Process.myPid(), Process.myUid());
if (permissionStatus == PackageManager.PERMISSION_GRANTED) {
    // 权限已授予
} else {
    // 权限未授予
}

```

## checkSelfPermission-检查权限

```java
if (PermissionManager.checkSelfPermission(Manifest.permission.READ_CONTACTS) == PackageManager.PERMISSION_GRANTED) {
    // 已获得权限，执行相关操作
} else {
    // 未获得权限，请求权限
}
```

## getPermissionFlags()‌-获取权限的详细标记

获取权限的详细标记（如用户是否选择“不再询问”）：
```java
int flags = permissionManager.getPermissionFlags(Manifest.permission.ACCESS_FINE_LOCATION, packageName, userId);
```

## requestPermissions-权限请求与修改‌
requestPermissions()‌  发起权限请求（与 Activity.requestPermissions() 类似，但通过系统服务处理）：
```java
permissionManager.requestPermissions(new String[]{Manifest.permission.RECORD_AUDIO}, requestCode, executor, callback);
```

参数‌：executor 指定回调线程，callback 实现 OnPermissionsResult 接口。

```java
String[] permissions = {Manifest.permission.CAMERA, Manifest.permission.LOCATION};
PermissionManager.requestPermissions(permissions, REQUEST_CODE_PERMISSIONS);
```


```java
if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
    != PackageManager.PERMISSION_GRANTED) {
    // 请求权限
    ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.CAMERA}, 1);
}

```


## onRequestPermissionsResult-处理权限请求结果

```java
参数
requestCode：请求码，与请求权限时传入的值对应。
permissions：请求的权限数组。
grantResults ：每个权限对应的授予权限结果，
PackageManager.PERMISSION_GRANTED 表示权限已授予，
PackageManager.PERMISSION_DENIED 表示权限被拒绝。
```


```java
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    if (requestCode == REQUEST_CODE_PERMISSIONS) {
        if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            // 用户授予权限，执行相关操作
        } else {
            // 用户拒绝权限，处理拒绝情况
        }
    }
}
```



## revokePermission()‌-撤销已授予的权限
撤销已授予的权限（仅系统或特权应用可用）：

```java
permissionManager.revokePermission(
    Manifest.permission.READ_SMS, 
    packageName, 
    userId);
```

## getPermissionGroupInfo-权限组管理‌
getPermissionGroupInfo()‌  获取权限组元数据：
```java
PermissionGroupInfo groupInfo = permissionManager.getPermissionGroupInfo(
    Manifest.permission_group.LOCATION, 0);
```


## getAllPermissionGroups()‌-列出所有已注册的权限组
列出所有已注册的权限组：

```java
List<PermissionGroupInfo> groups = permissionManager.getAllPermissionGroups(0);
```



## addOnPermissionsChangeListener-监听权限变化‌
addOnPermissionsChangeListener()‌  注册全局权限状态变化监听器：


```java
permissionManager.addOnPermissionsChangeListener(new PermissionManager.OnPermissionsChangedListener() {
    @Override
    public void onPermissionsChanged(int uid) {
        // 当指定UID应用的权限发生变化时触发
    }
});
```

## removeOnPermissionsChangeListener()‌-移除已注册的监听器
移除已注册的监听器，避免内存泄漏。


## openAppSettings-页面权限管理
用于打开应用的设置页面，方便用户在设置中手动调整权限。

```java
PermissionManager.openAppSettings();
```

---


# 动态申请权限并处理结果


```java
PermissionManager pm = getSystemService(PermissionManager.class);
pm.requestPermissions(
    new String[]{Manifest.permission.ACCESS_BACKGROUND_LOCATION},
    REQUEST_CODE,
    ContextCompat.getMainExecutor(this),
    new PermissionManager.OnPermissionsResult() {
        @Override
        public void onPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
            if (requestCode == REQUEST_CODE) {
                if (grantResults == PERMISSION_GRANTED) {
                    // 权限已授予
                } else {
                    // 处理拒绝逻辑
                }
            }
        }
    }
);
```

```java
// 检查位置权限是否已授权
if (!PermissionManager.getInstance().checkPermission(Manifest.permission.ACCESS_FINE_LOCATION)) {
    // 如果没有授权，则请求权限
    PermissionManager.getInstance().requestPermissions(
        new String[]{Manifest.permission.ACCESS_FINE_LOCATION},
        new PermissionManager.OnPermissionCallback() {
            @Override
            public void onPermissionGranted() {
                // 权限被授予后的操作
            }

            @Override
            public void onPermissionDenied() {
                // 权限被拒绝后的操作
            }

            @Override
            public void onPermissionRationaleShouldBeShown() {
                // 权限被拒绝且需要向用户解释为什么需要这个权限
            }
        }
    );
} else {
    // 权限已被授予，直接进行操作
}
```


---

# /data/system/packages.xml

权限存储：
PermissionManagerService 不负责权限信息的存储，而是由 PackageManagerService 负责。权限信息会存储在 /data/system/package.xml 文件中，设备重启时会读取该文件以恢复权限信息。


![权限存储](./image/权限存储.png)



---

# /system/etc/permissions/

系统启动阶段‌：PMS 从 /system/etc/permissions 目录加载预定义权限，并合并应用声明的自定义权限‌


## 应用添加到权限白名单

报权限相关的无法开机问题：

```java
E AndroidRuntime: *** FATAL EXCEPTION IN SYSTEM PROCESS: main
E AndroidRuntime: java.lang.IllegalStateException: Signature|privileged permissions not in privapp-permissions allowlist: {
com.topwise.myappstore (/system/priv-app/AppMarket): android.permission.RECOVERY, com.topwise.myappstore (/system/priv-app/AppMarket): android.permission.REAL_GET_TASKS, com.topwise.myappstore (/system/priv-app/AppMarket): android.permission.UPDATE_APP_OPS_STATS, com.topwise.myappstore (/system/priv-app/AppMarket): android.permission.INSTALL_PACKAGES, com.topwise.myappstore (/system/priv-app/AppMarket): android.permission.INTERACT_ACROSS_USERS, com.topwise.myappstore (/system/priv-app/AppMarket): android.permission.MANAGE_USERS, com.topwise.myappstore (/system/priv-app/AppMarket): android.permission.MOUNT_UNMOUNT_FILESYSTEMS, com.topwise.myappstore (/system/priv-app/AppMarket): android.permission.CLEAR_APP_CACHE}
E AndroidRuntime:     at com.android.server.pm.permission.PermissionManagerServiceImpl.onSystemReady(PermissionManagerServiceImpl.java:4545)
E AndroidRuntime:     at com.android.server.pm.permission.PermissionManagerService$PermissionManagerServiceInternalImpl.onSystemReady(PermissionManagerService.java:756)
E AndroidRuntime:     at com.android.server.pm.PackageManagerService.systemReady(PackageManagerService.java:4134)
E AndroidRuntime:     at com.android.server.SystemServer.startOtherServices(SystemServer.java:2885)
E AndroidRuntime:     at com.android.server.SystemServer.run(SystemServer.java:962)
E AndroidRuntime:     at com.android.server.SystemServer.main(SystemServer.java:665)
E AndroidRuntime:     at java.lang.reflect.Method.invoke(Native Method)
E AndroidRuntime:     at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:569)
E AndroidRuntime:     at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1002)
```

添加权限：

在手机中，手动在此文件中添加：

/system/etc/permissions/privapp-permissions-platform.xml


(1)应用权限添加到白名单中

base/data/etc/platform.xml

(2)应用权限添加在源码中添加：

frameworks/base/data/etc/privapp-permissions-platform.xml


```xml
<privapp-permissions package="com.android.documentsui">
<permission name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
<permission name="android.permission.WRITE_MEDIA_STORAGE"/>
<permission name="android.permission.INTERACT_ACROSS_USERS"/>
</privapp-permissions>
```

## /system/etc/permissions/privapp-permissions-platform.xml



https://blog.csdn.net/u012514113/article/details/125470210

### 第一种方案 ： 
修改 framework/base/data/etc/privapp-permissions-platform.xml 把自己的权限加入其中

### 第二种方案： 
还是通过Android.mk 文件实现，稍微修改一下路径就可以了：

Android.mk


```makefile
LOCAL_PATH:= $(call my-dir)

include $(CLEAR_VARS)
LOCAL_MODULE := TopDiagnosis
LOCAL_MODULE_TAGS := optional 
LOCAL_SRC_FILES := TopDiagnosis.apk
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_CERTIFICATE := platform
LOCAL_PRIVILEGED_MODULE := true
+######编译priv-app 权限到apk中###########
+LOCAL_MODULE_PATH := $(TARGET_OUT)/priv-app
+LOCAL_REQUIRED_MODULES := com.topwise.topdiagnosis.xml

include $(BUILD_PREBUILT)

+######预编译priv-app 权限，输出路径为system/etc/permissions###########
+# Permissions pre-grant
+include $(CLEAR_VARS)
+LOCAL_MODULE := com.topwise.topdiagnosis.xml
+LOCAL_MODULE_CLASS := ETC
+LOCAL_MODULE_PATH := $(TARGET_OUT_ETC)/permissions
+LOCAL_SRC_FILES := $(LOCAL_MODULE)
+include $(BUILD_PREBUILT)

+include $(call all-makefiles-under,$(LOCAL_PATH))
```

com.topwise.topdiagnosis.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<permissions>
    <privapp-permissions package="com.topwise.topdiagnosis">
        <permission name="android.permission.MODIFY_PHONE_STATE" />
        <permission name="android.permission.READ_PRIVILEGED_PHONE_STATE" />
    </privapp-permissions>
</permissions>
```

---

# 版本适配‌

Android 10+（API 29）‌：PermissionManager 正式引入。

Android 13+（API 33）‌：新增 POST_NOTIFICATIONS 等权限的精细管理接口。


---

# DefaultPermissionGrantPolicy

## grantDefaultPermissions--初始化

PermissionManagerService::systemReady

```java
private void systemReady() {
......
    // If we upgraded grant all default permissions before kicking off.
    for (int userId : grantPermissionsUserIds) {
        mDefaultPermissionGrantPolicy.grantDefaultPermissions(userId);
    }

```

```java
public void grantDefaultPermissions(int userId) {
    DelayingPackageManagerCache pm = new DelayingPackageManagerCache();

    grantPermissionsToSysComponentsAndPrivApps(pm, userId);
    //给默认应用赋权限
    grantDefaultSystemHandlerPermissions(pm, userId);
    grantDefaultPermissionExceptions(pm, userId);
}
```

## 给应用默认赋权限  给应用默认权限
grantDefaultSystemHandlerPermissions

```java
//Start by for virtualbluetoothservice
try{
    PackageInfo virtualBtInfo = pm.getSystemPackageInfo("com.topwise.bluetooh.virtualbluetoohprint");
    if (virtualBtInfo != null && doesPackageSupportRuntimePermissions(virtualBtInfo)){
        Log.i("dd","virtualBtInfo:" + virtualBtInfo.packageName + " to grant permission");
        grantRuntimePermissions(pm, virtualBtInfo,
            BLUETOOTH_PERMISSIONS,
            true, // systemFixed
            userId);
    }
} catch (Exception ee) {
}
//End by  for virtualbluetoothservice

//Start by  for appstore
try{
    PackageInfo appStoreInfo = pm.getSystemPackageInfo("com.topwise.myappstore");
    if (appStoreInfo != null && doesPackageSupportRuntimePermissions(appStoreInfo)){
        grantPermissionsToSystemPackage(pm, "com.topwise.myappstore",
            userId, ALWAYS_LOCATION_PERMISSIONS, STORAGE_PERMISSIONS);
    }
} catch (Exception ee) {
}
//End by  for appstore
```


## 添加应用需要运行时权限

```java
// AppStore
PackageInfo appStore = pm.getSystemPackageInfo("com.topwise.myappstore");
if (appStore != null && doesPackageSupportRuntimePermissions(appStore)){
    grantPermissionsToSystemPackage(pm,appStore.packageName,
            userId, ALWAYS_LOCATION_PERMISSIONS, STORAGE_PERMISSIONS);
}

// Custom app
PackageInfo customApp = pm.getSystemPackageInfo("com.smarteasy.refillo");
if (customApp != null && doesPackageSupportRuntimePermissions(customApp)){
    grantPermissionsToSystemPackage(pm,customApp.packageName,
            userId, ALWAYS_LOCATION_PERMISSIONS, STORAGE_PERMISSIONS,CAMERA_PERMISSIONS,PHONE_PERMISSIONS);
}
```


```java
+    private static final Set<String> XCHAT_PERMISSIONS = new ArraySet<>();
+    static {
+        XCHAT_PERMISSIONS.add("android.permission.INTERNET");
+        XCHAT_PERMISSIONS.add("android.permission.ACCESS_WIFI_STATE");
+        XCHAT_PERMISSIONS.add("android.permission.CHANGE_WIFI_STATE");
+        XCHAT_PERMISSIONS.add("android.permission.GET_TASKS");
+        XCHAT_PERMISSIONS.add("android.permission.VIBRATE");
+        XCHAT_PERMISSIONS.add("android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS");
+    }

  +          //xchat
  +          PackageParser.Package xchatPckg = getPackageLPr( "com.xthinks.xchat");
  +          if (xchatPckg != null
  +                  && doesPackageSupportRuntimePermissions(xchatPckg)) {
  +              Log.w(TAG, "grantRuntimePermissionsLPw for xchat");
  +              grantRuntimePermissionsLPw(xchatPckg, STORAGE_PERMISSIONS, true, userId);
  +              grantRuntimePermissionsLPw(xchatPckg, XCHAT_PERMISSIONS, true, userId);
  +          }
```

---

# Manifest.permission类

frameworks\base\api\current.txt

```java
  public static final class Manifest.permission {
    ctor public Manifest.permission();
    field public static final String ACCEPT_HANDOVER = "android.permission.ACCEPT_HANDOVER";
    field public static final String ACCESS_BACKGROUND_LOCATION = "android.permission.ACCESS_BACKGROUND_LOCATION";
    field public static final String ACCESS_CHECKIN_PROPERTIES = "android.permission.ACCESS_CHECKIN_PROPERTIES";
    field public static final String ACCESS_COARSE_LOCATION = "android.permission.ACCESS_COARSE_LOCATION";
    field public static final String ACCESS_FINE_LOCATION = "android.permission.ACCESS_FINE_LOCATION";
    field public static final String ACCESS_LOCATION_EXTRA_COMMANDS = "android.permission.ACCESS_LOCATION_EXTRA_COMMANDS";
    field public static final String ACCESS_MEDIA_LOCATION = "android.permission.ACCESS_MEDIA_LOCATION";
    field public static final String ACCESS_NETWORK_STATE = "android.permission.ACCESS_NETWORK_STATE";
    field public static final String ACCESS_NOTIFICATION_POLICY = "android.permission.ACCESS_NOTIFICATION_POLICY";
    field public static final String ACCESS_WIFI_STATE = "android.permission.ACCESS_WIFI_STATE";
    field public static final String ACCOUNT_MANAGER = "android.permission.ACCOUNT_MANAGER";
    field public static final String ACTIVITY_RECOGNITION = "android.permission.ACTIVITY_RECOGNITION";
    field public static final String ADD_VOICEMAIL = "com.android.voicemail.permission.ADD_VOICEMAIL";
    field public static final String ANSWER_PHONE_CALLS = "android.permission.ANSWER_PHONE_CALLS";
    field public static final String BATTERY_STATS = "android.permission.BATTERY_STATS";
    field public static final String BIND_ACCESSIBILITY_SERVICE = "android.permission.BIND_ACCESSIBILITY_SERVICE";
    field public static final String BIND_APPWIDGET = "android.permission.BIND_APPWIDGET";
    field public static final String BIND_AUTOFILL_SERVICE = "android.permission.BIND_AUTOFILL_SERVICE";
    field public static final String BIND_CALL_REDIRECTION_SERVICE = "android.permission.BIND_CALL_REDIRECTION_SERVICE";
```




---


# PermissionManager 与 AppOpsManager 核心对比

## 一、职责与设计目标差异‌
| ‌维度   | ‌PermissionManager‌‌ | ‌AppOpsManager‌‌     |
|--------|------|----------|
| 核心定位‌   | 管理应用声明的标准权限（如CAMERA、LOCATION），处理动态权限授予与状态跟踪   | 控制应用对敏感操作的实际执行权限（如后台定位、剪贴板读取），提供运行时拦截能力‌     |
| 权限模型‌   | 基于安装时声明（AndroidManifest）和运行时动态申请   | 基于动态策略控制，可覆盖传统权限设置（如授予权限但限制操作）‌     |
| 用户可见性‌   | 权限状态通过系统弹窗或设置界面直接暴露给用户   | 操作模式（如静默拒绝）通常由系统工具或企业策略配置，用户无感知     |





## 二、权限控制粒度对比‌
| ‌类别‌‌   | PermissionManager‌‌ | AppOpsManager‌‌     |
|--------|------|----------|
| 控制对象‌   | 标准权限组（如STORAGE、CONTACTS）   | 具体操作（OP_STR标识，如OP_READ_SMS、OP_CAMERA）‌     |
| 覆盖范围‌   | 仅管理声明在AndroidManifest中的权限   | 可控制未声明权限的敏感操作（如剪贴板读取、后台定位）‌     |
| 策略优先级‌   | 权限授予状态决定应用能否执行相关操作   | 即使权限被授予，仍可通过setMode()拦截操作     |




## 三、接口设计与使用场景‌

| ‌功能‌   | ‌PermissionManager‌‌ | AppOpsManager     |
|--------|------|----------|
| ‌‌核心接口‌   | checkPermission()：权限状态检查 requestPermissions()：动态申请权限   | checkOp():检测权限  noteOp()：记录单次操作 setMode()：动态调整操作模式 startOp()：标记长时操作     |
| 权限状态   | PackageManager.PERMISSION_GRANTED（已授予） PackageManager.PERMISSION_DENIED（未授予）   | AppOpsManager.MODE_ALLOWED：允许应用执行该操作。AppOpsManager.MODE_IGNORED：忽略应用对该操作的请求。AppOpsManager.MODE_ERRORED：发生错误。AppOpsManager.MODE_DEFAULT：使用默认的权限设置。     |
| ‌典型使用场景   | ‌应用内动态权限申请，权限合规性检查   | ‌后台敏感操作限制（如禁止应用读取剪贴板） ，企业设备管控（禁用非必要权限）‌|
| 权限依赖   | ‌普通应用可直接调用接口   | 需系统级应用签名或android:sharedUserId="android.uid.system"     |
| 用户交互   | 需要用户授予权限   | 无需用户交互，操作权限由系统或应用控制     |




## ‌四、数据存储与兼容性‌
| ‌维度‌‌   | PermissionManager‌‌ | AppOpsManager‌‌     |
|--------|------|----------|
| 配置存储‌   | 权限状态存储在/data/system/packages.xml   | 操作模式记录在/data/system/appops.xml‌     |
| 版本兼容性‌   | Android 10+ 正式引入自    | Android 4.3 引入，部分操作码需高版本适配（如 Android 13+ 新增OP_READ_MEDIA_IMAGES）‌     |
| 审计能力‌   | 提供权限授予历史记录   | 记录操作调用次数、时间等详细日志（如摄像头启动次数）‌     |




## 总结：核心区别与适用场景‌
PermissionManager‌
- ‌重点‌：管理应用声明的标准权限，遵循 Android 官方权限模型，提供用户可见的交互流程 
- ‌适用场景‌：常规动态权限申请、权限状态检查、权限组管理 


AppOpsManager‌
- ‌重点‌：控制应用实际操作的执行权限，支持细粒度策略覆盖，适用于系统级或企业级深度管控 
- ‌适用场景‌：后台敏感操作拦截、无声明权限的隐私保护、操作日志审计 ‌

## ‌选择建议‌
常规开发‌：优先使用 PermissionManager 处理动态权限流程，确保符合 Google Play 政策 

系统工具或企业应用‌：结合 AppOpsManager 实现更严格的权限策略（如禁止后台录音），需注意系统权限要求








---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p38-系统开发之permission-PermissionManager)

---


[返回 P38_系统开发之permission](https://github.com/hfreeman2008/android_core_framework/blob/main/P38_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bpermission/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bpermission.md)



---

# 结束语

<img src="../Images/end_001.png">