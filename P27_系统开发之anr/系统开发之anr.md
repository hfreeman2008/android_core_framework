
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

## 引爆炸弹
在system_server进程中有一个Handler线程, 名叫”ActivityManager”.当倒计时结束便会向该Handler线程发送 一条信息SERVICE_TIMEOUT_MSG,

ActivityManagerService#MainHandler

接收引爆消息SERVICE_TIMEOUT_MSG：

frameworks\base\services\core\java\com\android\server\am\ActivityManagerService.java

```java
final class MainHandler extends Handler {
   ......
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
        case SERVICE_TIMEOUT_MSG: {
            mServices.serviceTimeout((ProcessRecord)msg.obj);
        } break;
```

ActiveServices#serviceTimeout

frameworks\base\services\core\java\com\android\server\am\ActiveServices.java

```java
void serviceTimeout(ProcessRecord proc) {
    String anrMessage = null;
    synchronized(mAm) {
        ......
        if (proc.executingServices.size() == 0 || proc.thread == null) {
            return;
        }
        final long now = SystemClock.uptimeMillis();
        final long maxTime =  now -
                (proc.execServicesFg ? SERVICE_TIMEOUT : SERVICE_BACKGROUND_TIMEOUT);
        ServiceRecord timeout = null;
        long nextTime = 0;
        for (int i=proc.executingServices.size()-1; i>=0; i--) {
            ServiceRecord sr = proc.executingServices.valueAt(i);
            if (sr.executingStart < maxTime) {
                timeout = sr;
                break;
            }
            if (sr.executingStart > nextTime) {
                nextTime = sr.executingStart;
            }
        }
        if (timeout != null && mAm.mProcessList.mLruProcesses.contains(proc)) {
            Slog.w(TAG, "Timeout executing service: " + timeout);
            StringWriter sw = new StringWriter();
            PrintWriter pw = new FastPrintWriter(sw, false, 1024);
            pw.println(timeout);
            timeout.dump(pw, "    ");
            pw.close();
            mLastAnrDump = sw.toString();
            mAm.mHandler.removeCallbacks(mLastAnrDumpClearer);
            mAm.mHandler.postDelayed(mLastAnrDumpClearer, LAST_ANR_LIFETIME_DURATION_MSECS);
            anrMessage = "executing service " + timeout.shortInstanceName;
        } else {
            Message msg = mAm.mHandler.obtainMessage(
                    ActivityManagerService.SERVICE_TIMEOUT_MSG);
            msg.obj = proc;
            //再次发SERVICE_TIMEOUT_MSG
            mAm.mHandler.sendMessageAtTime(msg, proc.execServicesFg
                    ? (nextTime+SERVICE_TIMEOUT) : (nextTime + SERVICE_BACKGROUND_TIMEOUT));
        }
    }
    if (anrMessage != null) {
        //显示anr对话框
        mAm.mAnrHelper.appNotResponding(proc, anrMessage);
    }
}
```

## Service启动流程：

![Service启动流程](Service启动流程.png)


---

# BroadcastReceiver超时导致anr

BroadcastReceiver Timeout是位于”ActivityManager”线程中的BroadcastQueue.BroadcastHandler收到 BROADCAST_TIMEOUT_MSG 消息时触发。

frameworks/base/services/core/java/com/android/server/am/BroadcastQueue.java

```java
static final int BROADCAST_TIMEOUT_MSG = ActivityManagerService.FIRST_BROADCAST_QUEUE_MSG + 1;
```

对于广播队列有两个: foreground队列和background队列:
- 对于前台广播，则超时为 BROADCAST_FG_TIMEOUT = 10s；
- 对于后台广播，则超时为 BROADCAST_BG_TIMEOUT = 60s

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
static final int BROADCAST_FG_TIMEOUT = 10*1000;
static final int BROADCAST_BG_TIMEOUT = 60*1000;
```

## 埋炸弹
frameworks\base\services\core\java\com\android\server\am\BroadcastQueue.java

BroadcastQueue#processNextBroadcast

```java
final void processNextBroadcast(boolean fromMsg) {
    synchronized(mService) {
        ...
        //part 2: 处理当前有序广播
        do {
            r = mOrderedBroadcasts.get(0);
            //获取所有该广播所有的接收者
            int numReceivers = (r.receivers != null) ? r.receivers.size() : 0;
            if (mService.mProcessesReady && r.dispatchTime > 0) {
                long now = SystemClock.uptimeMillis();
                if ((numReceivers > 0) &&
                        (now > r.dispatchTime + (2*mTimeoutPeriod*numReceivers))) {
                    //当广播处理时间超时，则强制结束这条广播
                    broadcastTimeoutLocked(false);
                    ...
                }
            }
            if (r.receivers == null || r.nextReceiver >= numReceivers
                    || r.resultAbort || forceReceive) {
                if (r.resultTo != null) {
                    //处理广播消息消息
                    performReceiveLocked(r.callerApp, r.resultTo,
                        new Intent(r.intent), r.resultCode,
                        r.resultData, r.resultExtras, false, false, r.userId);
                    r.resultTo = null;
                }
                //拆炸弹
                cancelBroadcastTimeoutLocked();
            }
        } while (r == null);
        ...

        //part 3: 获取下条有序广播
        r.receiverTime = SystemClock.uptimeMillis();
        if (!mPendingBroadcastTimeoutMessage) {
            long timeoutTime = r.receiverTime + mTimeoutPeriod;
            //埋炸弹
            setBroadcastTimeoutLocked(timeoutTime);
        }
        ...
    }
}
```
BroadcastQueue#setBroadcastTimeoutLocked


```java
final void setBroadcastTimeoutLocked(long timeoutTime) {
    if (! mPendingBroadcastTimeoutMessage) {
        //发送消息BROADCAST_TIMEOUT_MSG，埋下炸弹
        Message msg = mHandler.obtainMessage(BROADCAST_TIMEOUT_MSG, this);
        mHandler.sendMessageAtTime(msg, timeoutTime);
        mPendingBroadcastTimeoutMessage = true;
    }
}
```



## 拆炸弹
frameworks\base\services\core\java\com\android\server\am\BroadcastQueue.java
BroadcastQueue#processNextBroadcast

```java
final void processNextBroadcast(boolean fromMsg) {
    synchronized (mService) {
        processNextBroadcastLocked(fromMsg, false);
    }
}

final void processNextBroadcastLocked(boolean fromMsg, boolean skipOomAdj) {  
    ......
    if (DEBUG_BROADCAST) Slog.v(TAG_BROADCAST, "Cancelling BROADCAST_TIMEOUT_MSG");
    cancelBroadcastTimeoutLocked();
    ......
}
```
BroadcastQueue#cancelBroadcastTimeoutLocked

```java
final void cancelBroadcastTimeoutLocked() {
    if (mPendingBroadcastTimeoutMessage) {
        //移除消息BROADCAST_TIMEOUT_MSG
        mHandler.removeMessages(BROADCAST_TIMEOUT_MSG, this);
        mPendingBroadcastTimeoutMessage = false;
    }
}
```

## 引爆炸弹
frameworks\base\services\core\java\com\android\server\am\BroadcastQueue.java

BroadcastQueue#BroadcastHandler

```java
private final class BroadcastHandler extends Handler {
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case BROADCAST_TIMEOUT_MSG: {
                synchronized (mService) {
                    //响应消息BROADCAST_TIMEOUT_MSG
                    broadcastTimeoutLocked(true);
                }
            } break;
        }
    }
}
```
BroadcastQueue#broadcastTimeoutLocked

```java
final void broadcastTimeoutLocked(boolean fromMsg) {
    if (fromMsg) {
        mPendingBroadcastTimeoutMessage = false;
    }
    if (mDispatcher.isEmpty() || mDispatcher.getActiveBroadcastLocked() == null) {
        return;
    }
    long now = SystemClock.uptimeMillis();
    BroadcastRecord r = mDispatcher.getActiveBroadcastLocked();
    if (fromMsg) {
        if (!mService.mProcessesReady) {
            // Only process broadcast timeouts if the system is ready; some early
            // broadcasts do heavy work setting up system facilities
            return;//当系统还没有准备就绪时，广播处理流程中不存在广播超时
        }
        // If the broadcast is generally exempt from timeout tracking, we're done
        if (r.timeoutExempt) {
            if (DEBUG_BROADCAST) {
                Slog.i(TAG_BROADCAST, "Broadcast timeout but it's exempt: "
                        + r.intent.getAction());
            }
            return;
        }
        long timeoutTime = r.receiverTime + mConstants.TIMEOUT;
        if (timeoutTime > now) {
            //如果当前正在执行的receiver没有超时，则重新设置广播超时
            setBroadcastTimeoutLocked(timeoutTime);
            return;
        }
    }
    if (r.state == BroadcastRecord.WAITING_SERVICES) {
        //广播已经处理完成，但需要等待已启动service执行完成。当等待足够时间，
        //则处理下一条广播。
        r.curComponent = null;
        r.state = BroadcastRecord.IDLE;
        processNextBroadcast(false);
        return;
    }
    final boolean debugging = (r.curApp != null && r.curApp.isDebugging());
    r.receiverTime = now;
    if (!debugging) {
        //当前BroadcastRecord的anr次数执行加1操作
        r.anrCount++;
    }
    ProcessRecord app = null;
    String anrMessage = null;
    Object curReceiver;
    if (r.nextReceiver > 0) {
        curReceiver = r.receivers.get(r.nextReceiver-1);
        r.delivery[r.nextReceiver-1] = BroadcastRecord.DELIVERY_TIMEOUT;
    } else {
        curReceiver = r.curReceiver;
    }
    logBroadcastReceiverDiscardLocked(r);
    //查询App进程
    if (curReceiver != null && curReceiver instanceof BroadcastFilter) {
        BroadcastFilter bf = (BroadcastFilter)curReceiver;
        if (bf.receiverList.pid != 0
                && bf.receiverList.pid != ActivityManagerService.MY_PID) {
            synchronized (mService.mPidsSelfLocked) {
                app = mService.mPidsSelfLocked.get(
                        bf.receiverList.pid);
            }
        }
    } else {
        app = r.curApp;
    }
    if (app != null) {
        anrMessage = "Broadcast of " + r.intent.toString();
    }
    if (mPendingBroadcast == r) {
        mPendingBroadcast = null;
    }
    // Move on to the next receiver.
    //继续移动到下一个广播接收者
    finishReceiverLocked(r, r.resultCode, r.resultData,
            r.resultExtras, r.resultAbort, false);
    scheduleBroadcastsLocked();
    if (!debugging && anrMessage != null) {
        //显示anr对话框
        mService.mAnrHelper.appNotResponding(app, anrMessage);
    }
}
```

---

# ContentProvider超时导致anr

ContentProvider Timeout是位于”ActivityManager”线程中的AMS.MainHandler收到 CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG 消息时触发。

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
static final int CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG = 57;
```
ContentProvider 超时为 CONTENT_PROVIDER_PUBLISH_TIMEOUT = 10s. 这个跟前面的Service和BroadcastQueue完全不同, 由Provider进程启动过程相关.

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
static final int CONTENT_PROVIDER_PUBLISH_TIMEOUT = 10*1000;
```

## 埋炸弹
frameworks\base\services\core\java\com\android\server\am\ActivityManagerService.java

ActivityManagerService#attachApplicationLocked

```java
private boolean attachApplicationLocked(@NonNull IApplicationThread thread,
            int pid, int callingUid, long startSeq) {
    ProcessRecord app;
    long startTime = SystemClock.uptimeMillis();
    long bindApplicationTimeMillis;
    if (pid != MY_PID && pid >= 0) {
        synchronized (mPidsSelfLocked) {
            app = mPidsSelfLocked.get(pid);
        }
    ......
    if (providers != null && checkAppInLaunchingProvidersLocked(app)) {
        //发送消息CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG，埋下炸弹
        Message msg = mHandler.obtainMessage(CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG);
        msg.obj = app;
        mHandler.sendMessageDelayed(msg,
                ContentResolver.CONTENT_PROVIDER_PUBLISH_TIMEOUT_MILLIS);
    }
    ......
}
```

## 拆炸弹
当provider成功publish之后,便会拆除该炸弹

frameworks\base\services\core\java\com\android\server\am\ActivityManagerService.java
ActivityManagerService#publishContentProviders

```java
public final void publishContentProviders(IApplicationThread caller,
            List<ContentProviderHolder> providers) {
    ......
    final int N = providers.size();
    for (int i = 0; i < N; i++) {
        ContentProviderHolder src = providers.get(i);
        if (src == null || src.info == null || src.provider == null) {
            continue;
        }
        ContentProviderRecord dst = r.pubProviders.get(src.info.name);
        if (dst != null) {
            ComponentName comp = new ComponentName(dst.info.packageName, dst.info.name);
            mProviderMap.putProviderByClass(comp, dst);
            String names[] = dst.info.authority.split(";");
            for (int j = 0; j < names.length; j++) {
                mProviderMap.putProviderByName(names[j], dst);
            }
            int launchingCount = mLaunchingProviders.size();
            int j;
            boolean wasInLaunchingProviders = false;
            for (j = 0; j < launchingCount; j++) {
                if (mLaunchingProviders.get(j) == dst) {
                    //将该provider移除mLaunchingProviders队列
                    mLaunchingProviders.remove(j);
                    wasInLaunchingProviders = true;
                    j--;
                    launchingCount--;
                }
            }
    ......
    //移除消息CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG
    if (wasInLaunchingProviders) {
        mHandler.removeMessages(CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG, r);
    }
    ......
    synchronized (dst) {
        dst.provider = src.provider;
        dst.setProcess(r);
        /唤醒客户端的wait等待方法
        dst.notifyAll();
    }
}
```

## 引爆炸弹
在system_server进程中有一个Handler线程, 名叫”ActivityManager”.当倒计时结束便会向该Handler线程发送 一条信息CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG,

frameworks\base\services\core\java\com\android\server\am\ActivityManagerService.java

ActivityManagerService#MainHandler

```java
final class MainHandler extends Handler {
    ......
    //收到消息CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG
    case CONTENT_PROVIDER_PUBLISH_TIMEOUT_MSG: {
        ProcessRecord app = (ProcessRecord)msg.obj;
        synchronized (ActivityManagerService.this) {
            processContentProviderPublishTimedOutLocked(app);
        }
    } break;
}
```


```java
private final void processContentProviderPublishTimedOutLocked(ProcessRecord app) {
    cleanupAppInLaunchingProvidersLocked(app, true);
    mProcessList.removeProcessLocked(app, false, true,
            ApplicationExitInfo.REASON_INITIALIZATION_FAILURE,
            ApplicationExitInfo.SUBREASON_UNKNOWN,
            "timeout publishing content providers");
}
```



```java
    boolean cleanupAppInLaunchingProvidersLocked(ProcessRecord app, boolean alwaysBad) {
        // Look through the content providers we are waiting to have launched,
        // and if any run in this process then either schedule a restart of
        // the process or kill the client waiting for it if this process has
        // gone bad.
        boolean restart = false;
        for (int i = mLaunchingProviders.size() - 1; i >= 0; i--) {
            ContentProviderRecord cpr = mLaunchingProviders.get(i);
            if (cpr.launchingApp == app) {
                if (++cpr.mRestartCount > ContentProviderRecord.MAX_RETRY_COUNT) {
                    // It's being launched but we've reached maximum attempts, mark it as bad
                    alwaysBad = true;
                }
                if (!alwaysBad && !app.bad && cpr.hasConnectionOrHandle()) {
                    restart = true;
                } else {
                    //移除死亡的provider
                    removeDyingProviderLocked(app, cpr, true);
                }
            }
        }
        return restart;
    }
```

---

# key事件分发超时导致anr


frameworks\base\services\core\java\com\android\server\wm\ActivityTaskManagerService.java

```java
    // How long we wait until we timeout on key dispatching.
    public static final int KEY_DISPATCHING_TIMEOUT_MS = 5 * 1000;
    // How long we wait until we timeout on key dispatching during instrumentation.
    static final int INSTRUMENTATION_KEY_DISPATCHING_TIMEOUT_MS = 60 * 1000;
```

```java
frameworks/native/services/inputflinger/InputDispatcher.cpp
constexpr nsecs_t DEFAULT_INPUT_DISPATCHING_TIMEOUT = 5000 * 1000000LL; // 5 sec

frameworks/base/services/core/java/com/android/server/wm/WindowManagerService.java
static final long DEFAULT_INPUT_DISPATCHING_TIMEOUT_NANOS = 5000 * 1000000L;
```

## 输入事件流程：

![输入事件流程1](输入事件流程1.png)


![输入事件流程2](输入事件流程2.png)

## ANR处理流程

ANR时间区间：

当前这次的事件dispatch过程中执行findFocusedWindowTargetsLocked()方法到下一次执行resetANRTimeoutsLocked()的时间区间. 


以下5个时机会reset. 都位于InputDispatcher.cpp文件
- resetAndDropEverythingLocked
- releasePendingEventLocked
- setFocusedApplication
- dispatchOnceInnerLocked
- setInputDispatchMode

简单来说, 主要是以下4个场景,会有机会执行resetANRTimeoutsLocked:
- 解冻屏幕, 系统开/关机的时刻点 (thawInputDispatchingLw, setEventDispatchingLw)
- wms聚焦app的改变 (WMS.setFocusedApp, WMS.removeAppToken)
- 设置input filter的过程 (IMS.setInputFilter)
- 再次分发事件的过程(dispatchOnceInnerLocked)

当InputDispatcher线程 findFocusedWindowTargetsLocked()过程调用到handleTargetsNotReadyLocked，且满足超时5s的情况则会调用onANRLocked().


## 输入事件的发送和接收主要流程：

![输入事件的发送和接收主要流程](输入事件的发送和接收主要流程.png)


## 按键事件超时监测整体流程：

![按键事件超时监测整体流程](按键事件超时监测整体流程.png)


---

```java


```


---





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