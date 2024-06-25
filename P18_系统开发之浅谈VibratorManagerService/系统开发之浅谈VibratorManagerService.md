# P18_系统开发之浅谈VibratorManagerService

<img src="../flower/flower_p18.png">


---

# VibratorManagerService 类的作用：

vibrator管理震动，这个服务比较简单，我们可以参考这个服务来自己做一个服务

---

# 获取VibratorManagerService的方式

```java
方式1
VibratorManager vibratorManager = mContext.getSystemService(VibratorManager.class);

方式2
Vibrator mVibrator = (Vibrator) context.getSystemService(Context.VIBRATOR_SERVICE);

方式3
IVibratorManagerService mService = IVibratorManagerService.Stub.asInterface(
        ServiceManager.getService(Context.VIBRATOR_MANAGER_SERVICE));
```

---

# VibratorManagerService调用流程






```java

```

```java

```

```java

```



---


# 结束语

<img src="../Images/end_001.png">
