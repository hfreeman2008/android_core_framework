
# P27: 系统开发之anr

<img src="../flower/flower_p27.png">

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

# ANR

ANR(application not response)，应用无响应。

整个ANR机制是横跨几个层：
- App层：
应用主线程的处理逻辑；

- Framework层：
主要有AMS、BroadcastQueue、ActiveServices、InputmanagerService、InputMonitor、InputChannel、ProcessCpuTracker等；

- Native层：
InputDispatcher.cpp；


---


# 哪些场景会造成ANR：


- Service Timeout:

比如前台服务在20s内未执行完成，后台服务200s未执行完成；

对于前台服务，则超时为 SERVICE_TIMEOUT = 20s；

对于后台服务，则超时为 SERVICE_BACKGROUND_TIMEOUT = 200s

logcat日志关键字：Timeout executing service

- BroadcastQueue Timeout：

比如前台广播在10s内未执行完成，后台广播在60s内未执行完成

对于前台广播，则超时为 BROADCAST_FG_TIMEOUT = 10s；

对于后台广播，则超时为 BROADCAST_BG_TIMEOUT = 60s;

logcat日志关键字：Timeout of broadcast BroadcastRecord

- ContentProvider Timeout：

内容提供者,在publish过超时10s;

ContentProvider超时为 CONTENT_PROVIDER_PUBLISH_TIMEOUT = 10s;

logcat日志关键字：timeout publishing content providers

- InputDispatching Timeout: 

输入事件分发超时5s，包括按键和触摸事件。

logcat日志关键字：Input event dispatching timed out


---

# service超时导致anr

Service Timeout是位于”ActivityManager”线程中的AMS.MainHandler收到 SERVICE_TIMEOUT_MSG 消息时触发。

```java
/frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java
static final int SERVICE_TIMEOUT_MSG = 12;
```


对于Service:
- 对于前台服务，则超时为 SERVICE_START_FOREGROUND_TIMEOUT = 10s；
- 一般的服务，超时为SERVICE_TIMEOUT = 20 s;
- 对于后台服务，则超时为 SERVICE_BACKGROUND_TIMEOUT = 200s

```java
frameworks/base/services/core/java/com/android/server/am/ActiveServices.java
// How long we wait for a service to finish executing.
static final int SERVICE_TIMEOUT = 20*1000;

// How long we wait for a service to finish executing.
static final int SERVICE_BACKGROUND_TIMEOUT = SERVICE_TIMEOUT * 10;

// How long the startForegroundService() grace period is to get around to
// calling startForeground() before we ANR + stop it.
static final int SERVICE_START_FOREGROUND_TIMEOUT = 10*1000;
```



---

## 埋炸弹

ActiveServices#scheduleServiceTimeoutLocked

发送引爆消息SERVICE_TIMEOUT_MSG：

frameworks\base\services\core\java\com\android\server\am\ActiveServices.java 

```java
/**
 * Note the name of this method should not be confused with the started services concept.
 * The "start" here means bring up the instance in the client, and this method is called
 * from bindService() as well.
 */
private final void realStartServiceLocked(ServiceRecord r,
        ProcessRecord app, boolean execInFg) throws RemoteException {
    .......
    r.setProcess(app);
    r.restartTime = r.lastActivity = SystemClock.uptimeMillis();
    final boolean newService = app.startService(r);
    //发送delay消息(SERVICE_TIMEOUT_MSG)
    bumpServiceExecutingLocked(r, execInFg, "create");
    ......
    //最终执行服务的onCreate()方法
    app.thread.scheduleCreateService(r, r.serviceInfo,
        mAm.compatibilityInfoForPackage(r.serviceInfo.applicationInfo),
        app.getReportedProcState());
    ......
}
```

```java
private final void bumpServiceExecutingLocked(ServiceRecord r, boolean fg, String why) {
    ... 
    scheduleServiceTimeoutLocked(r.app);
}
```

```java
void scheduleServiceTimeoutLocked(ProcessRecord proc) {
    if (proc.executingServices.size() == 0 || proc.thread == null) {
        return;
    }
    Message msg = mAm.mHandler.obtainMessage(
            ActivityManagerService.SERVICE_TIMEOUT_MSG);
    msg.obj = proc;
    //当超时后仍没有remove该SERVICE_TIMEOUT_MSG消息，则执行service Timeout流程
    mAm.mHandler.sendMessageDelayed(msg,
            proc.execServicesFg ? SERVICE_TIMEOUT : SERVICE_BACKGROUND_TIMEOUT);
}
```


## 拆炸弹

在system_server进程AS.realStartServiceLocked()调用的过程会埋下一颗炸弹, 超时没有启动完成则会爆炸. 那么什么时候会拆除这颗炸弹的引线呢? 经过Binder等层层调用进入目标进程的主线程handleCreateService()的过程.

(1)ActivityThread.handleCreateService

frameworks\base\core\java\android\app\ActivityThread.java

```java
private void handleCreateService(CreateServiceData data) {
        ...
        java.lang.ClassLoader cl = packageInfo.getClassLoader();
        Service service = (Service) cl.loadClass(data.info.name).newInstance();
        ...

        try {
            //创建ContextImpl对象
            ContextImpl context = ContextImpl.createAppContext(this, packageInfo);
            context.setOuterContext(service);
            //创建Application对象
            Application app = packageInfo.makeApplication(false, mInstrumentation);
            service.attach(context, this, data.info.name, data.token, app,
                    ActivityManagerNative.getDefault());
            //调用服务onCreate()方法 
            service.onCreate();
            //拆除炸弹引线
            ActivityManagerNative.getDefault().serviceDoneExecuting(
                    data.token, SERVICE_DONE_EXECUTING_ANON, 0, 0);
        } 
    }
```

(2)ActiveServices.serviceDoneExecutingLocked

该方法的主要工作是当service启动完成，则移除服务超时消息SERVICE_TIMEOUT_MSG:

frameworks\base\services\core\java\com\android\server\am\ActiveServices.java

```java
private void serviceDoneExecutingLocked(ServiceRecord r, boolean inDestroying,
            boolean finishing) {
    ......
    r.executeNesting--;
    if (r.executeNesting <= 0) {
    ......
    r.app.executingServices.remove(r);
    if (r.app.executingServices.size() == 0) {
        //拆炸弹:当前服务所在进程中没有正在执行的service,移除SERVICE_TIMEOUT_MSG消息
        mAm.mHandler.removeMessages(ActivityManagerService.SERVICE_TIMEOUT_MSG, r.app);
```




---

```java

```


```java

```




```sh

```



```sh

```





```java


```


---



![系统属性架构设计](系统属性架构设计.png)



---



---

# 参考资料

1.Android ANR分析

https://blog.csdn.net/yxz329130952/article/details/50087731


2.理解Android ANR的触发原理

http://gityuan.com/2016/07/02/android-anr/


3.解读Java进程的Trace文件

http://gityuan.com/2016/11/26/art-trace/

4.Native进程之Trace原理

http://gityuan.com/2016/11/27/native-traces/


5.Android打印Trace堆栈

http://gityuan.com/2017/07/09/android_debug/



---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p27-系统开发之anr)

---

# 结束语

<img src="../Images/end_001.png">