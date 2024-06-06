# 专题四: 系统分析的屠龙刀之log日志
android问题的分析，大家使用最多的，也可以说是最重要的就是日志，日志是系统问题分析的屠龙刀，是一点都不夸张。

<img src=".\Images\log_sword.png">

# 系统工程师log日志分析要达到什么水平

那log分析，要达到什么程度才算是可以达到系统工程师的水平呢？

我的回答是：

- 1.可以从log复盘出指定时间段的机器的操作和运行情况；

- 2.可以通过关键字定位到未知时间特定的异常或错误；

那对于这样一个每个人都会，每个人都用的log日志，我们这篇文章，准备能玩出什么花来了。

且来看下面分解。


# log日志常见分类

## system
--------- beginning of system

打印system日志的接口：

```java
import android.util.Slog;
 Slog.v(TAG, "system log ");
```

### ams相关
启动应用:
```java
ActivityTaskManager: START u0 {cmp=com.dream.dreamlogger/.DreagActivity} from uid 1000
ActivityManager: Start proc 6499:com.dream.dreamlogger/1000 for pre-top-activity {com.dream.dreamlogger/com.dream.dreamlogger.DreagActivity}
```


启动服务：
```java
ActivityManager: Start proc 6619:.connect.ConnectorService/1000 for service {com.android.usbaccessory/com.android.usbaccessory.connect.ConnectorService}
```


### wms相关

```java
V WindowManager: Orientation start waiting for draw, mDrawState=DRAW_PENDING in Window{1e537fa u0 com.android.settings/com.android.settings.FallbackHome}, surfaceController Surface(name=com.android.settings/com.android.settings.FallbackHome)/@0x9822dab
```




## events
--------- beginning of events



## main
--------- beginning of main

打印main日志的接口：

```java
 android.util.Log.i(TAG, "log info is ===========");
```

通过这个，我们就知道main日志大部分是app应用自已打印输出的，没有什么规律。


## crash
--------- beginning of crash
这个crash日志，是平常我们常见的问题处理之一，需要重点关注。

- 第一种:native crash

```java
F libc    : Fatal signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0x0 in tid 3374 (com.android.nfc), pid 3374 (com.android.nfc)
F DEBUG   : Build fingerprint: 
F DEBUG   : Revision: '0'
F DEBUG   : ABI: 'arm64'
F DEBUG   : Timestamp: 
F DEBUG   : pid: 3374, tid: 3374, name: com.android.nfc  >>> com.android.nfc <<<
F DEBUG   : uid: 1027
F DEBUG   : signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0x0
F DEBUG   : Cause: null pointer dereference
F DEBUG   :     x0  0000000000000000  x1  0000007fe73eb974  x2  0000000000000000  x3  b400007769de4380
F DEBUG   :     x4  0000007fe73ec020  x5  0000007595cc0769  x6  b2feff7568e0cdff  x7  ff7f7f7f7fffff7f
F DEBUG   : backtrace:
F DEBUG   :       #00 pc 000000000005c15c  /apex/com.android.runtime/lib64/bionic/libc.so (sem_getvalue) (BuildId: f4881cdb04823cc0d8c0fa3f95c4db2e)
F DEBUG   :       #03 pc 000000000000a65c  /system/app/NfcNci/oat/arm64/NfcNci.odex (art_jni_trampoline+124) (BuildId: 13e5d153b841131176688eda1cc33cec2778b1e6)
F DEBUG   :       #04 pc 0000000000133564  /apex/com.android.art/lib64/libart.so (art_quick_invoke_stub+548) (BuildId: 0183cc6150704cdc371a87b659800e56)
F DEBUG   :       #08 pc 000000000067df14  /apex/com.android.art/lib64/libart.so (MterpInvokeInterface+1032) (BuildId: 0183cc6150704cdc371a87b659800e56)
F DEBUG   :       #09 pc 000000000012da14  /apex/com.android.art/lib64/libart.so (mterp_op_invoke_interface+20) (BuildId: 0183cc6150704cdc371a87b659800e56)
F DEBUG   :       #10 pc 000000000017c858  /system/app/NfcNci/NfcNci.apk (offset 0x1000) (com.android.nfc.NfcService$NfcServiceHandler.handleMessage+1092)
```

- 第二种：app 应用crash
```java
E AndroidRuntime: Process: com.dream.recorder, PID: 3691
E AndroidRuntime: java.lang.IllegalArgumentException: supportsCameraApi:2365: Unknown camera ID 0
E AndroidRuntime:  at android.hardware.camera2.CameraManager.throwAsPublicException(CameraManager.java:1013)
E AndroidRuntime:  at android.hardware.camera2.CameraManager.getCameraCharacteristics(CameraManager.java:461)
E AndroidRuntime:  at android.hardware.camera2.CameraManager.openCameraDeviceUserAsync(CameraManager.java:497)
E AndroidRuntime:  at android.hardware.camera2.CameraManager.openCameraForUid(CameraManager.java:737)
E AndroidRuntime:  at android.hardware.camera2.CameraManager.openCamera(CameraManager.java:665)
E AndroidRuntime:         at com.dream.recorder.media.camera.impl.LnlyjCameraImpl.openInner(LnlyjCameraImpl.java:252)
E AndroidRuntime:         at com.dream.recorder.media.camera.impl.LnlyjCameraImpl.access$800(LnlyjCameraImpl.java:32)
E AndroidRuntime:         at com.dream.recorder.media.camera.impl.LnlyjCameraImpl$WorkHandler.open(LnlyjCameraImpl.java:573)
E AndroidRuntime:         at com.dream.recorder.media.camera.impl.LnlyjCameraImpl$WorkHandler.handleMessage(LnlyjCameraImpl.java:538)
E AndroidRuntime:         at android.os.Handler.dispatchMessage(Handler.java:106)
E AndroidRuntime:         at android.os.Looper.loop(Looper.java:223)
E AndroidRuntime:         at android.os.HandlerThread.run(HandlerThread.java:67)
E AndroidRuntime: Caused by: android.os.ServiceSpecificException: supportsCameraApi:2365: Unknown camera ID 0 (code 3)
E AndroidRuntime:         at android.os.Parcel.createExceptionOrNull(Parcel.java:2387)
E AndroidRuntime:         at android.os.Parcel.createException(Parcel.java:2357)
E AndroidRuntime:         at android.hardware.ICameraService$Stub$Proxy.supportsCameraApi(ICameraService.java:906)
E AndroidRuntime:         at android.hardware.camera2.CameraManager.supportsCameraApiLocked(CameraManager.java:1066)
E AndroidRuntime:         at android.hardware.camera2.CameraManager.supportsCamera2ApiLocked(CameraManager.java:1042)
E AndroidRuntime:         at android.hardware.camera2.CameraManager.getCameraCharacteristics(CameraManager.java:434)
```

## radio
--------- beginning of radio
此部分日志主要为和网络相关，留白，因为本人没有什么有效的经验分享。

## kernel

查看kernel日志命令：

```java
adb shell cat /proc/kmsg > kernel.log
adb shell dmesg > kernel_001.log
```

此部分留白，因为本人没有做过驱动，没有什么有效的经验分享。


