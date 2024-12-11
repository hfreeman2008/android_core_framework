# FAQ14385 user/userdebug模式下如何打开 ANR predump的Debug开关

# DESCRIPTION

在解决ANR问题的过程中，可能会碰到Log不足或者backtrace不准确的情况。
那么可以通过ANR的predump机制帮我们打印出在ANR还未发生之前，即timeout一半的时候的backtrace，或者当时的进程的historymessage信息，来帮助我们解决ANR的问题
  
# SOLUTION
 ANR predump 的方法，根据需要选择其一：

- 方法一：动态执行adb shell 命令
```java
adb shell dumpsys activity log anr 2 （命令重启后无效）
```

- 方法二：修改代码，适合需要重启手机或者开机阶段的时候的ANR
/frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java
```java
 public void systemReady(final Runnable goingCallback) {
……

 ANRManager.AnrOption = 2;
 /*
 if (IS_USER_BUILD && 0 == anrStatus) {
 Settings.System.putInt(mContext.getContentResolver(), Settings.System.ANR_DEBUGGING_MECHANISM, mANRManager.DISABLE_PARTIAL_ANR_MECHANISM);
 ANRManager.AnrOption = mANRManager.DISABLE_PARTIAL_ANR_MECHANISM;
 Settings.System.putInt(mContext.getContentResolver(), Settings.System.ANR_DEBUGGING_MECHANISM_STATUS, 1);
 }
 /// Enable/disable ANR mechanism from adb command @}
*/

……
```