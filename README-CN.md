
## [README of English][readme]

---

# android core and key information of framework


此系统文章主要是针对android framework的一些核心和关键信息做一些笔记，同时也是分享一些自己的心得和技术总结。


当然，这也可以理解为一个android 系统工程师的一些必备技术技能。


<img src="./flower/flower_red_003.png">


---

## P1：基于android 13的GMS认证
[基于android 13的GMS认证.md](./P1_基于android_13的GMS认证/基于android_13的GMS认证.md)

---

## P2: 开机速度优化
[开机速度优化.md](./P2_开机速度优化/开机速度优化.md)

---

## P3: AndroidStudio不同渠道打包
[AndroidStudio不同渠道打包.md](./P3_AndroidStudio不同渠道打包/AndroidStudio不同渠道打包.md)

---

## P4: 系统分析的屠龙刀之log日志
[系统分析的屠龙刀之log日志.md](./P4_系统分析的屠龙刀之log日志/系统分析的屠龙刀之log日志.md)

---

## P5: 系统分析的屠龙刀之dumpsys信息
[系统分析的屠龙刀之dumpsys信息.md](./P5_系统分析的屠龙刀之dumpsys信息/系统分析的屠龙刀之dumpsys信息.md)

---

## P6: 系统分析的屠龙刀之traceview
[系统分析的屠龙刀之traceview.md](./P6_系统分析的屠龙刀之traceview/系统分析的屠龙刀之traceview.md)

---

## P7: 系统分析的屠龙刀之Android Profile
[系统分析的屠龙刀之Profile.md](./P7_系统分析的屠龙刀之Profile/系统分析的屠龙刀之Profile.md)

---

## P8: 系统开发之自定义系统服务
[系统开发之自定义系统服务.md](./P8_系统开发之自定义系统服务/系统开发之自定义系统服务.md)

---

## P9: 系统开发之系统服务初步了解
[系统开发之系统服务初步了解.md](./P9_系统开发之系统服务初步了解/系统开发之系统服务初步了解.md)

---

## P10: 系统开发之浅谈TimeZoneDetectorService
[系统开发之浅谈TimeZoneDetectorService.md](./P10_系统开发之浅谈TimeZoneDetectorService/系统开发之浅谈TimeZoneDetectorService.md)

---

## P11: 系统开发之浅谈TimeDetectorService
[系统开发之浅谈TimeDetectorService.md](./P11_系统开发之浅谈TimeDetectorService/系统开发之浅谈TimeDetectorService.md)

---

## P12: 系统开发之浅谈WindowManagerService
[系统开发之浅谈WindowManagerService.md](./P12_系统开发之浅谈WindowManagerService/系统开发之浅谈WindowManagerService.md)

---

## P13: 系统开发之浅谈ActivityTaskManagerService
[系统开发之浅谈ActivityTaskManagerService.md](./P13_系统开发之浅谈ActivityTaskManagerService/系统开发之浅谈ActivityTaskManagerService.md)

---

## P14: 系统开发之浅谈ActivityManagerService
[系统开发之浅谈ActivityManagerService.md](./P14_系统开发之浅谈ActivityManagerService/系统开发之浅谈ActivityManagerService.md)

---

## P15: 系统开发之浅谈PackageManagerService
[系统开发之浅谈PackageManagerService.md](./P15_系统开发之浅谈PackageManagerService/系统开发之浅谈PackageManagerService.md)

---

## P16: 系统开发之浅谈PowerManagerService
[系统开发之浅谈PowerManagerService.md](./P16_系统开发之浅谈PowerManagerService/系统开发之浅谈PowerManagerService.md)

---

## P17: 系统开发之浅谈BatteryService
[系统开发之浅谈BatteryService.md](./P17_系统开发之浅谈BatteryService/系统开发之浅谈BatteryService.md)

---

## P18: 系统开发之浅谈VibratorManagerService
[系统开发之浅谈VibratorManagerService.md](./P18_系统开发之浅谈VibratorManagerService/系统开发之浅谈VibratorManagerService.md)

---

## P19: 系统开发之浅谈AudioService
[系统开发之浅谈AudioService.md](./P19_系统开发之浅谈AudioService/系统开发之浅谈AudioService.md)

---

## P20: 系统开发之浅谈NotificationManagerService
[系统开发之浅谈NotificationManagerService.md](./P20_系统开发之浅谈NotificationManagerService/系统开发之浅谈NotificationManagerService.md)

---

备注：这一系统文章参考源码以android 11和13为主，还有部分是其他的android 版本。

---
# Plan
内存，cpu,......


---
# Donations

**所谓法不轻传，道不贱卖；**

**千两黄金不卖道；**

如果你看完这个系列的文章，觉得对你有用，或者能助你在android开发上走的更远，那么你可以扫下方二维码随意打赏我，就当是请我喝杯咖啡或是啤酒，我将不胜感激 :-)


<div align=center>
<img src=".\Images\donate.jpg" width=200 height=300>
<div align=left>


---
# Contact

- [csdn博客](https://blog.csdn.net/hfreeman2008)

- [github博客](https://github.com/hfreeman2008)

**欢迎大家加我做朋友**
<div align=center>
<img src=".\Images\weixin_hxm_001.png">
<div align=left>

---

# About me

2011年深圳大学毕业后，一直从事android开发，大部分时间是手机开发。主要的开发经验在app,framework java层，对于sh脚本，android 编译，配置也比较熟悉。


大学时的我(2011)：
<div align=center>
<img src=".\Images\2008年大学.png" width=500 height=600>
<div align=left>

工作十年+后的我(2024)：
<div align=center>
<img src=".\Images\2024_hexiaoming.png" width=500 height=600>
<div align=left>


android开发是个杀猪刀，入行要考虑清楚。



---


[readme]: https://github.com/hfreeman2008/android_core_framework/blob/main/README.md
[readme-cn]: https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md

