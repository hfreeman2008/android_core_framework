
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