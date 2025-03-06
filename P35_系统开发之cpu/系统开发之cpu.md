# P35: 系统开发之cpu

<img src="../flower/flower_p25.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

[上一篇文章 P34_系统开发之Display](https://github.com/hfreeman2008/android_core_framework/blob/main/P34_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay.md)






---

# kernel cpu

```bash
kernel\msm-4.19/drivers/cpufreq
kernel\msm-4.19\drivers\cpuidle
```

---

# 功耗的计算

P = f * V * V 

(功耗与频率成正比，与电压的平方成正比)


---

# adb shell dumpsys cpuinfo

```bash
Load: 1.92 / 1.59 / 0.97
CPU usage from 1698338ms to 798324ms ago:
  48% 2844/com.jamdeo.tv.vod: 42% user + 6.6% kernel / faults: 1126309 minor 2533 major
  12% 1991/svcrefsig: 11% user + 0.6% kernel
  7.1% 1648/surfaceflinger: 4% user + 3% kernel / faults: 10697 minor 35 major
  ......
  0.5% 540/ion_system_heap: 0% user + 0.5% kernel
 +0% 25126/kworker/1:3: 0% user + 0% kernel
27% TOTAL: 19% user + 6.9% kernel + 0.3% iowait + 0.3% softirq
```

第一行显示的是cpuload （负载平均值）信息：Load: 1.92 / 1.59 / 0.97 这三个数字表示逐渐变长的时间段（平均一分钟，五分钟和十五分钟）的平均值，而较低的数字则更好。数字越大表示有问题或机器过载。

需要注意的是，这里的Load需要除以核心数，比如我这里的系统核心数为8核，所以最终每一个单核CPU的Load为0.24 / 0.20 / 0.12，如果Load超过1，则表示出现了问题。

---

# adb shell top

---

# adb shell vmstat 

---

# adb shell cat  /proc/cpuinfo

```bash
processor    : 0
Processor    : ARMv7 Processor rev 4 (v7l)
model name    : ARMv7 Processor rev 4 (v7l)
Features    : half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm aes pmull sha1 sha2 crc32 
CPU architecture: 7

......

Hardware    : AC8257V/WAB
Revision    : 0000
Serial        : 0000000000000000
```



---
# adb shell ls -lha /sys/devices/system/cpu/

```bash
core_ctl_isolated  cpu1/              cpu3/              cpu5/              cpu7/              cpufreq/           hotplug/           kernel_max         offline            possible           present            vulnerabilities/
cpu0/              cpu2/              cpu4/              cpu6/              cpu_boost/         cpuidle/           isolated           modalias           online             power/             uevent
```

---

# 查看有多少个CPU

```bash
adb shell cat sys/devices/system/cpu/online
0-3
```

---


---

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```



```bash

```


```bash

```


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p35-系统开发之cpu)

---


[上一篇文章 P34_系统开发之Display](https://github.com/hfreeman2008/android_core_framework/blob/main/P34_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay.md)



---

# 结束语

<img src="../Images/end_001.png">