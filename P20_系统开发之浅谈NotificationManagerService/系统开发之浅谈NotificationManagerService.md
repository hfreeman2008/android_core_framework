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

# NotificationManagerService 类图

<img src="NotificationManagerService_class.png">


---

# 日志开关
```java
NotificationManagerService.java
public static final boolean DBG = true;//Log.isLoggable(TAG, Log.DEBUG);

NotificationRecord.java
static final boolean DBG = true;//Log.isLoggable(TAG, Log.DEBUG);
```

---

# dump信息
adb shell dumpsys notification

```java
DUMP OF SERVICE notification:
Current Notification Manager state:
  Notification List:
    NotificationRecord(0x0dfbaffe: pkg=android user=UserHandle{-1} id=26 tag=null importance=4 key=-1|android|26|null|1000: Notification(channel=DEVELOPER_IMPORTANT shortcut=null contentView=null vibrate=null sound=null tick defaults=0x0 flags=0x2 color=0x00000000 vis=PUBLIC))
      uid=1000 userId=-1
      opPkg=android
      icon=Icon(typ=RESOURCE pkg=android id=0x0108081f)
      flags=0x2
      originalFlags=0x2
      pri=0
      key=-1|android|26|null|1000
      seen=true
      groupKey=-1|android|26|null|1000
      notification=
            fullscreenIntent=null
            contentIntent=PendingIntent{3d835f: PendingIntentRecord{970f496 android startActivity}}
            deleteIntent=null
            number=0
            groupAlertBehavior=0
            when=0
            tickerText=...
            contentView=null
            bigContentView=null
            headsUpContentView=null
            color=0x00000000
            timeout=unknown
            extras={
                android.title=String [length=11]
                android.reduced.images=Boolean (true)
                android.text=String [length=13]
                android.appInfo=ApplicationInfo (ApplicationInfo{2b211ac android})
                android.tv.EXTENSIONS=Bundle (Bundle[{channel_id=usbdevicemanager.adb.tv, suppressShowOverApps=false, flags=1}])
            }
      publicNotification=
            None
      stats=SingleNotificationStats{posttimeElapsedMs=59501689, posttimeToFirstClickMs=-1, posttimeToDismissMs=-1, airtimeCount=1, airtimeMs=5639, currentAirtimeStartElapsedMs=-1, airtimeExpandedMs=5636, posttimeToFirstVisibleExpansionMs=818, currentAirtimeExpandedStartElapsedMs=-1, requestedImportance=3, naturalImportance=4, isNoisy=true}
      mContactAffinity=0.0
      mRecentlyIntrusive=false
      mPackagePriority=0
      mPackageVisibility=-1000
      mSystemImportance=UNSPECIFIED
      mAsstImportance=UNSPECIFIED
      mImportance=HIGH
      mImportanceExplanation=app
      mIsAppImportanceLocked=false
      mIntercept=false
      mHidden==false
      mGlobalSortKey=crtcl=0x0002:intrsv=1:grnk=0x0000:gsmry=1:nsk:rnk=0x0000
      mRankingTimeMs=1706767655811
      mCreationTimeMs=1706767655811
      mVisibleSinceMs=1706767656832
      mUpdateTimeMs=1706767655811
      mInterruptionTimeMs=1706767656833
      mSuppressedVisualEffects= 0
      mSound= content://settings/system/notification_sound
      mVibration= null
      mAttributes= AudioAttributes: usage=USAGE_NOTIFICATION content=CONTENT_TYPE_SONIFICATION flags=0x800 tags= bundle=null
      mLight= null
      mShowBadge=false
      mColorized=false
      mAllowBubble=false
      isBubble=false
      mIsInterruptive=true
      effectiveNotificationChannel=NotificationChannel{mId='DEVELOPER_IMPORTANT', mName=重要开发者消息, mDescription=, mImportance=4, mBypassDnd=false, mLockscreenVisibility=-1000, mSound=content://settings/system/notification_sound, mLights=false, mLightColor=0, mVibration=null, mUserLockedFields=0, mFgServiceShown=false, mVibrationEnabled=false, mShowBadge=true, mDeleted=false, mDeletedTimeMs=-1, mGroup='null', mAudioAttributes=AudioAttributes: usage=USAGE_NOTIFICATION content=CONTENT_TYPE_SONIFICATION flags=0x800 tags= bundle=null, mBlockableSystem=false, mAllowBubbles=-1, mImportanceLockedDefaultApp=false, mOriginalImp=4, mParent=null, mConversationId=null, mDemoted=false, mImportantConvo=false}
      mAdjustments=[]
      shortcut=null found valid? false

......
  
  mUseAttentionLight=false
  mHasLight=false
  mNotificationPulseEnabled=true
  mSoundNotificationKey=null
  mVibrateNotificationKey=null
  mDisableNotificationEffects=true
  mCallState=CALL_STATE_IDLE
  mSystemReady=true
  mMaxPackageEnqueueRate=5.0
  hideSilentStatusBar=false
  mArchive=Archive (0 notifications)

  Snoozed notifications:

 Pending snoozed notifications

  Ranking Config:
    mSignalExtractors.length = 11
      NotificationChannelExtractor
      NotificationAdjustmentExtractor
      BubbleExtractor
      ValidateNotificationPeople
      PriorityExtractor
      ZenModeExtractor
      ImportanceExtractor
      NotificationIntrusivenessExtractor
      VisibilityExtractor
      BadgeExtractor
      CriticalNotificationExtractor


```

---

# 日志


```java

```

---



```java

```


```java

```

```java

```







---


# 结束语

<img src="../Images/end_001.png">