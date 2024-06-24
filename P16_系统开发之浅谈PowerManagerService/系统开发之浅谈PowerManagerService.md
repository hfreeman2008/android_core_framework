# P16_系统开发之浅谈PowerManagerService

<img src="../flower/flower_u_white_001.png">


---

# PowerManagerService 类的作用：

主要是处理与电源有关的逻辑,包括手机亮度管理,手机锁屏,节能锁屏,手机低电量模式,手机睡眠,手机双击亮屏.

---


# 获取PowerManagerService的方式：

```java
方式1
PowerManager powerManager = (PowerManager)context.getSystemService(Context.POWER_SERVICE);

方式2
IPowerManager pm = IPowerManager.Stub.asInterface(
        ServiceManager.getService(Context.POWER_SERVICE));

方式3 (system server进程使用)
PowerManagerInternal mLocalPowerManager = getLocalService(PowerManagerInternal.class);
```



# PowerManagerService 调用流程

---

<img src="PowerManagerService_whole.png">




```

```java

```

```java

```


---



# 结束语

<img src="../Images/end_001.png">
