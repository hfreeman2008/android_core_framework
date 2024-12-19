# P29: 系统开发之tp

<img src="../flower/flower_p29.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

# touchscreen屏幕
kernel\msm-4.9\drivers\input\touchscreen

---
# 屏幕架构
Frameworks

touch drive

i2c core

touch(interrut,gpio,VDD,VCC,RST,data sheet)

---

# tp和lcm介绍
1. LCM指的是液晶显示屏，即Liquid Crystal Module，它是手机屏幕的主要部分，用于显示图像和信息。

2. TP指的是触摸屏，即Touch Panel，它是手机屏幕的一部分，用于接收用户的触摸操作。

3. AA区域，全称为Active Area，指的是屏幕的可视区域。这个区域包含了屏幕上可以进行交互的显示部分，用户可以看到并对其进行操作。


---

# TP的硬件接口

引脚 | 名称及作用
-|-|
VDD | TP供电
RESET | 复位引脚
EINT | 中断引脚
SCL SDA | I2C接口


​ TP的工作方式比较简单：
- 上电后通过RESET脚控制TP芯片复位；
- 通过I2C接口给TP设置参数或读取TP数据；
- TP有触摸操作时通过EINT脚通知主控；

---

# 一个Demo
kernel\msm-4.9\drivers\input\touchscreen

我们项目的物料为：gt1x，我们进入此目录，我们先查看probe：

grep -nr probe

```java
kernel/msm-4.9$ grep -rni "probe"  drivers/input/touchscreen/gt1x/
drivers/input/touchscreen/gt1x/docs/RevisionLog.txt:10:         - Add fallback flow to probe function.
drivers/input/touchscreen/gt1x/gt1x.c:761: * gt1x_ts_probe -   I2c probe.
drivers/input/touchscreen/gt1x/gt1x.c:766:static int gt1x_ts_probe(struct i2c_client *client, const struct i2c_device_id *id)
drivers/input/touchscreen/gt1x/gt1x.c:903:      GTP_ERROR("GTP probe failed:%d", ret);
drivers/input/touchscreen/gt1x/gt1x.c:996:      .probe = gt1x_ts_probe,
```

我们查看：vi  gt1x.c +996

![gt1x_01](./image/gt1x_01.png)


```java

```


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



# 参考资料





---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p29-系统开发之tp)

---

# 结束语

<img src="../Images/end_001.png">