# sensor种类

<img src="../flower/flower_p28.png">

---


[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---


# Accelerometer 加速度传感器
加速度传感器又叫G-sensor，返回x、y、z三轴的加速度数值。

该数值包含地心引力的影响，单位是m/s^2。

将手机平放在桌面上，x轴默认为0，y轴默认0，z轴默认9.81。 将手机朝下放在桌面上，z轴为-9.81。 将手机向左倾斜，x轴为正值。 将手机向右倾斜，x轴为负值。 将手机向上倾斜，y轴为负值。 将手机向下倾斜，y轴为正值。

# Magnetometer 磁力传感器

磁力传感器简称为M-sensor，返回x、y、z三轴的环境磁场数据。

该数值的单位是微特斯拉（micro-Tesla），用uT表示。 单位也可以是高斯（Gauss），1Tesla=10000Gauss。 硬件上一般没有独立的磁力传感器，磁力数据由电子罗盘传感器提供（E-compass）。
电子罗盘传感器同时提供下文的方向传感器数据。

# Orientation 方向传感器

方向传感器简称为O-sensor，返回三轴的角度数据，方向数据的单位是角度。

为了得到精确的角度数据，E-compass(电子罗盘传感器)需要获取G-sensor(加速度传感器)的数据， 经过计算生产O-sensor数据，否则只能获取水平方向的角度。

方向传感器提供三个数据，分别为azimuth、pitch和roll。

azimuth：方位，返回水平时磁北极和Y轴的夹角，范围为0°至360°。 0°=北，90°=东，180°=南，270°=西。

pitch：x轴和水平面的夹角，范围为-180°至180°。 当z轴向y轴转动时，角度为正值。

roll：y轴和水平面的夹角，由于历史原因，范围为-90°至90°。 当x轴向z轴移动时，角度为正值。

电子罗盘在获取正确的数据前需要进行校准，通常可用8字校准法。 8字校准法要求用户使用需要校准的设备在空中做8字晃动， 原则上尽量多的让设备法线方向指向空间的所有8个象限。

# Gravity 重力传感器

重力传感器简称GV-sensor，输出重力数据。

在地球上，重力数值为9.8，单位是m/s^2。 坐标系统与加速度传感器相同。 当设备复位时，重力传感器的输出与加速度传感器相同。

# Gyroscope 陀螺仪传感器

陀螺仪传感器叫做Gyro-sensor，返回x、y、z三轴的角速度数据。

角速度的单位是radians/second。

根据Nexus S手机实测： 水平逆时针旋转，Z轴为正。 水平逆时针旋转，z轴为负。 向左旋转，y轴为负。 向右旋转，y轴为正。 向上旋转，x轴为负。 向下旋转，x轴为正。

# Ambient Light Sensor (环境)光线感应传感器
光线感应传感器检测实时的光线强度，光强单位是lux(勒克司度)，其物理意义是照射到单位面积上的光通量。

光线感应传感器主要用于Android系统的LCD自动亮度功能。

可以根据采样到的光强数值实时调整LCD的亮度。

# Barometer Sensor (气)压力传感器
压力传感器返回当前的压强，单位是百帕斯卡hectopascal（hPa）。

# Temperature Sensor 温度传感器
温度传感器返回当前的温度。

# Proximity Sensor (近)距离传感器
又称接近传感器，检测物体与手机的距离，单位是cm(厘米)。

一些接近传感器只能返回远和近两个状态， 因此，接近传感器将最大距离返回远状态，小于最大距离返回近状态。 接近传感器可用于接听电话时自动关闭LCD屏幕以节省电量。

一些芯片集成了接近传感器和光线传感器两者功能。

# Linear Acceleration 线性加速度传感器
线性加速度传感器简称LA-sensor。

线性加速度传感器是加速度传感器减去重力影响获取的数据。 单位是m/s^2，坐标系统与加速度传感器相同。

加速度传感器、重力传感器和线性加速度传感器的计算公式如下： 加速度 = 重力 + 线性加速度

# Rotation Vector 旋转矢量传感器
旋转矢量传感器简称RV-sensor。

旋转矢量代表设备的方向，是一个将坐标轴和角度混合计算得到的数据。

RV-sensor输出三个数据： xsin(theta/2) ysin(theta/2) z*sin(theta/2) sin(theta/2)是RV的数量级。

RV的方向与轴旋转的方向相同。 RV的三个数值，与cos(theta/2)组成一个四元组。 RV的数据没有单位，使用的坐标系与加速度相同。

举例：

```cpp
sensors_event_t.data[0] = x*sin(theta/2)
sensors_event_t.data[1] = ysin(theta/2)
sensors_event_t.data[2] = z*sin(theta/2)
sensors_event_t.data[3] = cos(theta/2)
```

---

# 其他传感器

- Step Detector 步数探测器
- Step Counter 计步器
- Significant Motion Detector 运动检测器
- Game Rotation Vector 游戏旋转矢量
- Geomagnetic Rotation Vector 地磁旋转矢量
- Basic Gestures 基本手势
- Motion Accel 运动加速度

---


[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#sensor种类)

---

# 结束语

<img src="../Images/end_001.png">