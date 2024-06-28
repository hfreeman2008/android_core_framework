# P20: 系统开发之浅谈NotificationManagerService

<img src="../flower/flower_p20.png">

---

# NotificationManagerService 作用

主要是管理通知(notification)，包含通知的增加，删除，更新，播放通知声音，创建一些白名单应用的通知栏，消息等级管理，震动，led灯的管理，通知的策略管理。

---

# 获取通知管理服务：

```java
方式1
NotificationManager notificationManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);

方式2
NotificationManager nm = context.getSystemService(NotificationManager.class);

方式3
INotificationManager iNotificationManager = NotificationManager.getService();

方式4
IBinder b = ServiceManager.getService("notification");
INotificationManager sService = INotificationManager.Stub.asInterface(b);

方式5 (system server进程使用)
NotificationManagerInternal nm = LocalServices.getService(NotificationManagerInternal.class);
```

---

# 常用发送通知方式

```java
NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
NotificationCompat.Builder builder = new NotificationCompat.Builder(this, CHANNEL_ID)
.setSmallIcon(R.drawable.ic_notification)
.setContentTitle("通知标题")
.setContentText("通知内容")
.setPriority(NotificationCompat.PRIORITY_DEFAULT);
notificationManager.notify(NOTIFICATION_ID, builder.build());
```

frameworks\base\core\java\android\app\Notification.java
创建一个notification,Notification.Builder就是我们常用的方式：

```java
Notification noti = new Notification.Builder(mContext)
         .setContentTitle(&quot;New mail from &quot; + sender.toString())
         .setContentText(subject)
         .setSmallIcon(R.drawable.new_mail)
        .setLargeIcon(aBitmap)
        .build();
```

---

# NotificationManagerService 调用流程

<img src="NotificationManagerService_whole.png">


以getActiveNotifications为例：
(1)app中调用getActiveNotifications：
```java
NotificationManager notificationManager = getNotificationManager(context);
StatusBarNotification[] activeNotifications = notificationManager.getActiveNotifications();
```

(2)NotificationManager.java中调用getActiveNotifications：

```java
public StatusBarNotification[] getActiveNotifications() {
    final INotificationManager service = getService();
    final String pkg = mContext.getPackageName();
    try {
        final ParceledListSlice<StatusBarNotification> parceledList
                = service.getAppActiveNotifications(pkg, mContext.getUserId());
        if (parceledList != null) {
            final List<StatusBarNotification> list = parceledList.getList();
            return list.toArray(new StatusBarNotification[list.size()]);
        }
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
    return new StatusBarNotification[0];
}
```

(3)INotificationManager.aidl中定义getActiveNotifications：
```java
StatusBarNotification[] getActiveNotifications(String callingPkg);
```

(4)NotificationManagerService.java中实现getActiveNotifications：
```java
/**
 * @deprecated Use {@link #getActiveNotificationsWithAttribution(String, String)} instead.
 */
@Deprecated
@Override
public StatusBarNotification[] getActiveNotifications(String callingPkg) {
    return getActiveNotificationsWithAttribution(callingPkg, null);
}

/**
 * System-only API for getting a list of current (i.e. not cleared) notifications.
 *
 * Requires ACCESS_NOTIFICATIONS which is signature|system.
 * @returns A list of all the notifications, in natural order.
 */
@Override
public StatusBarNotification[] getActiveNotificationsWithAttribution(String callingPkg,
        String callingAttributionTag) {
    // enforce() will ensure the calling uid has the correct permission
    getContext().enforceCallingOrSelfPermission(
            android.Manifest.permission.ACCESS_NOTIFICATIONS,
            "NotificationManagerService.getActiveNotifications");

    ArrayList<StatusBarNotification> tmp = new ArrayList<>();
    int uid = Binder.getCallingUid();

    ArrayList<Integer> currentUsers = new ArrayList<>();
    currentUsers.add(UserHandle.USER_ALL);
    Binder.withCleanCallingIdentity(() -> {
        for (int user : mUm.getProfileIds(ActivityManager.getCurrentUser(), false)) {
            currentUsers.add(user);
        }
    });

    // noteOp will check to make sure the callingPkg matches the uid
    if (mAppOps.noteOpNoThrow(AppOpsManager.OP_ACCESS_NOTIFICATIONS, uid, callingPkg,
            callingAttributionTag, null)
            == MODE_ALLOWED) {
        synchronized (mNotificationLock) {
            final int N = mNotificationList.size();
            for (int i = 0; i < N; i++) {
                final StatusBarNotification sbn = mNotificationList.get(i).getSbn();
                if (currentUsers.contains(sbn.getUserId())) {
                    tmp.add(sbn);
                }
            }
        }
    }
    return tmp.toArray(new StatusBarNotification[tmp.size()]);
}
```

(5)启动通知管理服务：
SystemServer.startOtherServices
```java
t.traceBegin("StartNotificationManager");
mSystemServiceManager.startService(NotificationManagerService.class);
SystemNotificationChannels.removeDeprecated(context);
SystemNotificationChannels.createAll(context);
notification = INotificationManager.Stub.asInterface(
        ServiceManager.getService(Context.NOTIFICATION_SERVICE));
t.traceEnd();
```

---

```java

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







---


# 结束语

<img src="../Images/end_001.png">