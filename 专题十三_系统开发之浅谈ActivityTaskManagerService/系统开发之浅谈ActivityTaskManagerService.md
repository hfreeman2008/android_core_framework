# 专题十三_系统开发之浅谈ActivityTaskManagerService

<img src="flows_atms_001.png">


# ActivityTaskManagerService 类的作用：
ActivityTaskManagerService 类的作用就是管理activity，和activity的容器（task,stack,displays）

```java
/**
 * System service for managing activities and their containers (task, displays,... ).
 *
 * {@hide}
 */
```


# 获取atms的方式：


```java
方式1
ActivityTaskManager mActivityTaskManager;
mActivityTaskManager = mContext.getSystemService(ActivityTaskManager.class);

方式2
IActivityTaskManager mActivityTaskManager;
mActivityTaskManager = ActivityTaskManager.getService();

方式3
IBinder b = ServiceManager.getService(Context.ACTIVITY_TASK_SERVICE);
IActivityTaskManager mActivityTaskManager = IActivityTaskManager.Stub.asInterface(b);


方式4(system server进程使用)
ActivityTaskManagerInternal mActivityTaskManagerService;
mActivityTaskManagerService = LocalServices.getService(ActivityTaskManagerInternal.class);
```


# ActivityTaskManagerService调用流程

<img src="atms_whole.png">

图一 ActivityTaskManagerService调用流程

参考图一 ActivityTaskManagerService调用流程，我们以removeAllVisibleRecentTasks()接口为例，看一下其调用流程

(1)在app应用中调用ActivityTaskManagerService.removeAllVisibleRecentTasks()接口：
```java
ActivityTaskManager atm = mContext.getSystemService(ActivityTaskManager.class);
atm.removeAllVisibleRecentTasks();
```

(2)其对应ActivityTaskManager.removeAllVisibleRecentTasks：

```java
public void removeAllVisibleRecentTasks() {
        getService().removeAllVisibleRecentTasks();
}

public static IActivityTaskManager getService() {
    return IActivityTaskManagerSingleton.get();
}

private static final Singleton<IActivityTaskManager> IActivityTaskManagerSingleton =
        new Singleton<IActivityTaskManager>() {
            @Override
            protected IActivityTaskManager create() {
                //其对应ActivityTaskManagerService服务
                final IBinder b = ServiceManager.getService(Context.ACTIVITY_TASK_SERVICE);
                return IActivityTaskManager.Stub.asInterface(b);
            }
        };
```

(3)其对应IActivityTaskManager.aidl中定义接口：

```java
void removeAllVisibleRecentTasks();
```

(4)ActivityTaskManagerService服务中removeAllVisibleRecentTasks：

```java
public void removeAllVisibleRecentTasks() {
    mAmInternal.enforceCallingPermission(REMOVE_TASKS, "removeAllVisibleRecentTasks()");
    synchronized (mGlobalLock) {
        final long ident = Binder.clearCallingIdentity();
        try {
            getRecentTasks().removeAllVisibleTasks(mAmInternal.getCurrentUserId());
        } finally {
            Binder.restoreCallingIdentity(ident);
        }
    }
}

RecentTasks getRecentTasks() {
    return mRecentTasks;
}
```
(5)进一步查看RecentTasks类中的removeAllVisibleTasks：

```java
void removeAllVisibleTasks(int userId) {
    Set<Integer> profileIds = getProfileIds(userId);
    for (int i = mTasks.size() - 1; i >= 0; --i) {
        final Task task = mTasks.get(i);
        if (!profileIds.contains(task.mUserId)) continue;
        if (isVisibleRecentTask(task)) {
            mTasks.remove(i);
            notifyTaskRemoved(task, true /* wasTrimmed */, true /* killProcess */);
        }
    }
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

(7)注册ActivityTaskManagerService服务：

SystemServiceRegistry.java

```java
registerService(Context.ACTIVITY_TASK_SERVICE, ActivityTaskManager.class,
    new CachedServiceFetcher<ActivityTaskManager>() {
@Override
public ActivityTaskManager createService(ContextImpl ctx) {
    return new ActivityTaskManager(
            ctx.getOuterContext(), ctx.mMainThread.getHandler());
}});
```


# ActivityTaskManagerService类图

<img src="atms_class.png">

图二 ActivityTaskManagerService类图


图二，我们可以看到ActivityTaskManagerService主要是管理activity,activity 容器，wms，应用管理，状态栏，锁屏，返回手势，用户管理等。


# activity和各个contain的关系

<img src="../专题十二_系统开发之浅谈WindowManagerService/ams_wms_001.png">


这部分参考：

[android开发浅谈之ActivityDisplay/ActivityStack/TaskRecord/ActivityRecord](https://blog.csdn.net/hfreeman2008/article/details/113309272)

https://blog.csdn.net/hfreeman2008/article/details/113309272

[android开发浅谈之WindowToken,AppWindowToken,Task,TaskStack和DisplayContent](https://blog.csdn.net/hfreeman2008/article/details/113589077)

https://blog.csdn.net/hfreeman2008/article/details/113589077


此部分是以前的关系图，android最新的版本是已经调整了，后面需要同步更新此部分数据。


# android 13 ams和wms的关系

## 1. 界面结构布局

<img src="atms_whole_13.png">



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

# 结束语

<img src="../Images/end_001.png">
