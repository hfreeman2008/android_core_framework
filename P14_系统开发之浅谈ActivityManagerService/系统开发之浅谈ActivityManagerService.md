
# P14_系统开发之浅谈ActivityManagerService

<img src="../flower/flower_001.png">

---


# ActivityManagerService类的作用：
四大组件：activity管理，广播管理，SERVICES，CONTENT PROVIDERS，TASK MANAGEMENT,PERMISSIONS，PROCESS INFO，BACKUP AND RESTORE，INSTRUMENTATION，LIFETIME MANAGEMENT，监视anr,cpu,oom,memo空间，电量。

```java
// =========================================================
CONTENT PROVIDERS
SERVICES
BROADCASTS
TASK MANAGEMENT
LIFETIME MANAGEMENT
INSTRUMENTATION
PROCESS INFO
PERMISSIONS
GLOBAL MANAGEMENT
BACKUP AND RESTORE
// =========================================================
```

---
# 获取ams的方式：

```java
方式1
ActivityManager mAm;
mAm = context.getSystemService(ActivityManager.class);

方式2
ActivityManager activityManager =(ActivityManager) mContext.getSystemService(Context.ACTIVITY_SERVICE);
        
方式3
IActivityManager mAm;
mAm = ActivityManager.getService();

方式4 (system server进程使用)
ActivityManagerInternal mAm;
mAm = LocalServices.getService(ActivityManagerInternal.class);
```
---
# ActivityManagerService调用流程

<img src="ams_whole.png">

图一 ActivityManagerService调用流程

参考图一，我们以getRunningAppProcesses()接口为例，看一下其调用流程

(1)在app应用中调用ActivityManagerService.getRunningAppProcesses()接口：

```java
ActivityManager am = (ActivityManager) getContext().getSystemService(Context.ACTIVITY_SERVICE);
List<RunningAppProcessInfo> apps = am.getRunningAppProcesses();
```

(2)其对应ActivityManager.getRunningAppProcesses：

```java
public List<RunningAppProcessInfo> getRunningAppProcesses() {
     ......
     return getService().getRunningAppProcesses();
}
```
(3)其对应IActivityManager.aidl中定义接口：

```java
List<ActivityManager.RunningAppProcessInfo> getRunningAppProcesses();
```
(4)ActivityManagerService服务中getRunningAppProcesses：

```java
public List<ActivityManager.RunningAppProcessInfo> getRunningAppProcesses() {
    enforceNotIsolatedCaller("getRunningAppProcesses");

    final int callingUid = Binder.getCallingUid();
    final int clientTargetSdk = mPackageManagerInt.getUidTargetSdkVersion(callingUid);

    final boolean allUsers = ActivityManager.checkUidPermission(INTERACT_ACROSS_USERS_FULL,
            callingUid) == PackageManager.PERMISSION_GRANTED;
    final int userId = UserHandle.getUserId(callingUid);
    final boolean allUids = mAtmInternal.isGetTasksAllowed(
            "getRunningAppProcesses", Binder.getCallingPid(), callingUid);

    synchronized (mProcLock) {
        // Iterate across all processes
        return mProcessList.getRunningAppProcessesLOSP(allUsers, userId, allUids,
                callingUid, clientTargetSdk);
    }
}
```
(5)进一步查看ProcessList类中的getRunningAppProcessesLOSP

```java
List<ActivityManager.RunningAppProcessInfo> getRunningAppProcessesLOSP(boolean allUsers,
        int userId, boolean allUids, int callingUid, int clientTargetSdk) {
    // Lazy instantiation of list
    List<ActivityManager.RunningAppProcessInfo> runList = null;
    ......
    //Slog.v(TAG, "Proc " + app.processName + ": imp=" + currApp.importance
    //        + " lru=" + currApp.lru);
    if (runList == null) {
        runList = new ArrayList<>();
    }
    runList.add(currApp);
    }
    return runList;
}
```
(6)在SystemServer.startBootstrapServices启动ActivityTaskManagerService服务：
```java
// Activity manager runs the show.
t.traceBegin("StartActivityManager");
// TODO: Might need to move after migration to WM.
ActivityTaskManagerService atm = mSystemServiceManager.startService(
        ActivityTaskManagerService.Lifecycle.class).getService();
mActivityManagerService = ActivityManagerService.Lifecycle.startService(
        mSystemServiceManager, atm);
mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
mActivityManagerService.setInstaller(installer);
mWindowManagerGlobalLock = atm.getGlobalLock();
t.traceEnd();
```

(7)注册WindowManagerService服务：
SystemServiceRegistry.java
```java
registerService(Context.ACTIVITY_SERVICE, ActivityManager.class,
        new CachedServiceFetcher<ActivityManager>() {
    @Override
    public ActivityManager createService(ContextImpl ctx) {
        return new ActivityManager(ctx.getOuterContext(), ctx.mMainThread.getHandler());
    }});
```
---

# ActivityManagerService类图

<img src="ams_class.png">

图二 ActivityManagerService类图

图二，我们可以看到ActivityManagerService就是和四大组件的管理（activity,activity的容器，wms，广播，content provider,service）,以及电池电量，app应用管理，oom内存管理等各个方面的一个综合管理体。

其代码将近2万行，变量，方法众多，我们全部研究是不现实的，也没有必要，但是我们可以针对性的了解，方便我们做对应的开发。
---

# 探究mActivityTaskManager

如我们想研究mActivityTaskManager,探究mActivityTaskManager是如何调用的，我们可以直接搜索mActivityTaskManager：

```java
public ActivityTaskManagerService mActivityTaskManager;

private void start() {
    .....
    mActivityTaskManager.onActivityManagerInternalAdded();
}

```
查看ActivityTaskManagerService.onActivityManagerInternalAdded：

```java
public void onActivityManagerInternalAdded() {
    synchronized (mGlobalLock) {
        mAmInternal = LocalServices.getService(ActivityManagerInternal.class);
        mUgmInternal = LocalServices.getService(UriGrantsManagerInternal.class);
    }
}
```
可以看出，这就是一个初始化的逻辑调用。
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

```java

```

---

---

# 结束语

<img src="../Images/end_001.png">