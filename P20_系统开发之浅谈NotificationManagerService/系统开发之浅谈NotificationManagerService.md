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

```
Notification noti = new Notification.Builder(mContext)
         .setContentTitle(&quot;New mail from &quot; + sender.toString())
         .setContentText(subject)
         .setSmallIcon(R.drawable.new_mail)
        .setLargeIcon(aBitmap)
        .build();


---

# NotificationManagerService 调用流程



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