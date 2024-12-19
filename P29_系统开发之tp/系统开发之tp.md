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

我们再看看方法gt1x_ts_probe：

![gt1x_ts_probe](./image/gt1x_ts_probe.png)

我们再搜索compatible：

grep -rni "compatible" ./

./gt1x.c:776:           {.compatible = "goodix,gt1x",},


查看：

![compatible_01](./image/compatible_01.png)

进入目录：kernel/msm-4.9/arch/arm64/boot/dts/qcom，查找“goodix,gt1x”：

grep -nr goodix,gt1x

```java
kernel/msm-4.9$ grep -rni "goodix,gt1x"  arch/arm64/boot/dts/qcom/
arch/arm64/boot/dts/qcom/sdm710-i7s-qrd.dtsi:257:                compatible = "goodix,gt1x";
```

对就查看此配制文件：

 vi sdm710-i7s-qrd.dtsi +257

 ![goodix_dtsi](./image/goodix_dtsi.png)


gt1x.c 驱动加载:

```java
module_init(gt1x_ts_init);
```
调用gt1x_ts_init函数，作用是注册一个i2c驱动:

```java
/**   
 * gt1x_ts_init - Driver Install function.
 * Return   0---succeed.
 */
static int __init gt1x_ts_init(void)
{
	//GTP_INFO("GTP driver installing...");
	return i2c_add_driver(&gt1x_ts_driver);
}
```
再查看变量gt1x_ts_driver:

```java
static struct i2c_driver gt1x_ts_driver = {
	.probe = gt1x_ts_probe,
	.remove = gt1x_ts_remove,
	.id_table = gt1x_ts_id,
	.driver = {
		   .name = GTP_I2C_NAME,
		   .owner = THIS_MODULE,
#ifdef CONFIG_OF
		   .of_match_table = gt1x_match_table,
#endif
		   },
};
```

这个gt1x_ts_driver.driver.name = GTP_I2C_NAME，去们可以查看:

```java
cat /proc/bus/input/devices
```

probe函数内容gt1x_ts_probe:

```java
/**
 * gt1x_ts_probe -   I2c probe.
 * @client: i2c device struct.
 * @id: device id.
 * Return  0: succeed, <0: failed.
 */
static int gt1x_ts_probe(struct i2c_client *client, const struct i2c_device_id *id)
{
	int ret = -1;
	struct goodix_ts_data *ts;

	/* do NOT remove these logs */
	GTP_INFO("GTP Driver Version: %s,slave addr:%02xh",
			GTP_DRIVER_VERSION, client->addr);

	gt1x_i2c_client = client;
	spin_lock_init(&irq_lock);
```
调用到linux-3.0.86/drivers/input/input.c中的一些方法（实质上注册input设备，生成设备节点等什么都是这个东西干的）

---

# touch panel 驱动定位

通过：

```java
adb shell cat /proc/bus/input/devices
```


```java
I: Bus=0018 Vendor=dead Product=beef Version=28bb
N: Name="goodix-ts"
P: Phys=input/ts
S: Sysfs=/devices/virtual/input/input3
U: Uniq=
H: Handlers=kgsl event3 cpufreq
B: PROP=2
B: EV=b
B: KEY=400 c00000000000000 0 0 0 0
B: ABS=661800000000000
```

cat /dev/input/event0(根据实际情况分析具体是那个节点),最终确定tp的驱动：

```java
N: Name="goodix-ts"
```

然后去内核中确定源码是：

```java
LINUX\android\kernel\msm-4.9\drivers\input\touchscreen\goodix.c
```

确定dtsi：
```java
grep -rni  --include={*.dts,*.dtsi} "goodix-ts"  ./
```
vendor\qcom\proprietary\devicetree-4.19\qcom\sc780-dts\bengal-qrd.dtsi 中一个tp touch的定义：
```java
&qupv3_se2_i2c {
	status = "okay";
	qcom,i2c-touch-active="chipsemi,chsc_cap_touch";

	smtouch@2E {
		compatible = "chipsemi,chsc_cap_touch";
		reg = <0x2E>;
		interrupt-parent = <&tlmm>;
		interrupts = <80 0x2008>;
		//vdd-supply = <&pm8916_l17>;
		vio-supply = <&L9A>;
		chipsemi,int-gpio = <&tlmm 80 0x2008>;
		chipsemi,rst-gpio = <&tlmm 71 0x00>;
		chipsemi,vdd-en-gpio = <&tlmm 97 0x00>;
		pinctrl-names = "pmx_ts_active","pmx_ts_suspend","pmx_ts_release","pmx_ts_int_active";
		pinctrl-0 = <&ts_reset_active>;
		pinctrl-1 = <&ts_int_suspend &ts_reset_suspend>;
		pinctrl-2 = <&ts_release>;
		pinctrl-3 = <&ts_int_active>;

		panel = <&dsi_sc780_st7785m_qvga_video>;
	};
};
```

配置开关一般在这种文件中：

```java
kernel/msm-4.19/arch/arm64/configs/vendor/sc780_defconfig
kernel/msm-4.19/arch/arm64/configs/vendor/sc780-perf_defconfig
```

---

# Qualcomm 平台触摸屏驱动移植

https://blog.csdn.net/weijory/article/details/72733155

调试相关经验:

一般TP驱动开发，屏产都会给驱动代码或者PATCH，这时主要合代码进去。

一般找代码内现有的一个TP驱动，按它的添加。主要：

1. 把驱动文件放入kernel\drivers\input\touchscreen\，
2. 修改kconfig和Makefile，加入需要根据宏才能编进去，那么需要在deconfig配置文件中设置为Y.
3. 在DTSI中加入该TP的配置。
4. 编译boot，在out/target/product/msmXXX/obj/KERNEL_OBJ/driver/input/touchscreen/下，看是否有.o文件没有，有则编译成功。
5. 把新的boot文件刷入板子，查看内核log，cat proc/kmsg，看是否有该TP驱动的打印信息。
6. 根据打印信息，判断出错的问题。

一般问题，中断注册不上，资源分配不成功，I2C设备通信失败。

一些经验，I2C总线不通，可能是因为I2C供电的电源没有供电，或者该总线上挂的设备太多影响的，前期调最好I2C总线上，只挂一个设备。

若probe成功，可在中断或者工作线程里面加一些打印log。
在adb shell进入终端，输入getevent，手按TP，查看是否有数据打出，对于该TP的输入设备。

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

---


---



# 参考资料





---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p29-系统开发之tp)

---

# 结束语

<img src="../Images/end_001.png">