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

# adb shell cat /proc/stat

```bash
cpu  198178 24452 470960 20101746 8719 195793 53310 0 0 0
cpu0 47388 5745 116163 15783063 8033 54365 13406 0 0 0
cpu1 48861 4913 119994 610943 272 48337 13485 0 0 0
cpu2 49287 6357 118486 610053 211 46892 13100 0 0 0
cpu3 48930 6067 111291 611538 176 45926 13265 0 0 0
cpu4 753 190 860 621650 2 62 10 0 0 0
cpu5 833 276 1130 621685 9 58 12 0 0 0
cpu6 1068 103 884 621256 6 47 8 0 0 0
cpu7 1055 798 2149 621555 7 102 20 0 0 0

intr 33036846 0 3013362 0 0 3074087 0 5724772 0 11 11 0 0 0 0 0 4487 0 0 0 0 1805133 0 0 169321 11486 19048 45 0 0 0
7 29 6 0 6 1 2740 2728 0 0 0 0 0 0 0 1 0 241552 1 255 18 0 0 0 0 0 0 0 22 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1218 0 0 0 0 0 0 0 0 0 11 0 126119 111006 0 4672 1651 2283 0 0 0 141 0 2 2 0 2 0 2 0 2 0 10 0 3971 0 0 0 0 0 0 0 0 0 0 0 4 577462 0 0 0 0 0 0 0 0 0 0 0 0 0 7 0 0 0 0 1 0 1 0 1 0 1 0 1 0 1 0 0 9 0 0 0 0 435955 0 0 0 0 0 0 34 0 0 0 0 0 0 0 249 0
0 189 0 0 0 0 0 0 0 0 0 0 0 0 0 119 0 0 0 0 0 0 0 0 0 3 2 0 4 0 5 39 0 0 5 0 0 5 0 0 13422 1649 0 30 120 0 4 0 0 0 0
0 0 0 0 1469 1025 0 0 0 1 289 18 2 240 11617 2 4 0 0 0 0 0 0 0 0 0 0 0 0 40747 0 0

ctxt 56302055
btime 1735626477
processes 21792
procs_running 1
procs_blocked 0
softirq 13832526 49 7818885 16145 57 49 0 93137 3593090 0 2311114
```

---

# /proc/stat解析

https://blog.csdn.net/houzhizhen/article/details/79474427

1.1 CPU时间
```bash
cpu指标 含义
user 用户态时间
nice 用户态时间(低优先级，nice>0)
system 内核态时间
idle 空闲时间
iowait I/O等待时间
irq 硬中断
softirq 软中断
```


2.2 说明

```bash
intr：系统启动以来的所有interrupts的次数情况
ctxt: 系统上下文切换次数
btime：启动时长(单位:秒)，从Epoch(即1970零时)开始到系统启动所经过的时长，每次启动会改变。
processes：系统启动后所创建过的进程数量。当短时间该值特别大，系统可能出现异常
procs_running：处于Runnable状态的进程个数
procs_blocked：处于等待I/O完成的进程个数
```

---

# Android-Systrace-CPU

https://www.androidperformance.com/2019/12/21/Android-Systrace-CPU


配置 cpuset

使用 cpuset 子系统可以限制某一类的任务跑在特定的 CPU 或者 CPU 组里面，比如下面，Android 中会划分一些默认的 CPU 组，厂商可以针对不同的 CPU 架构进行定制，目前默认划分
```bash
system-background 一些低优先级的任务会被划分到这里，只能跑到小核心里面
foreground 前台进程
top-app 目前正在前台和用户交互的进程
background 后台进程
foreground/boost 前台 boost 进程，通常是用来联动的，现在已经没有用到了，之前的时候是应用启动的时候，会把所有 foreground 里面的进程都迁移到这个进程组里面
```

每个 CPU 架构对应的 cpuset 的配置都不一样，每个厂商也会有不同的策略在里面，比如下面就是一个 Google 官方默认的配置，各位也可以查看对应的节点来查看自己的 CPUset 组的配置

此配置信息在：

init.target.rc

```bash
# For cpusets initialize for Silver Only first and then Silver + Gold
# Silver Only configuration cannot work with 0-7
on boot
chown system system /sys/kernel/hbtp/display_pwr
start rmt_storage
start rfs_access
write /dev/cpuset/top-app/cpus 0-3
write /dev/cpuset/audio-app/cpus 1-2
write /dev/cpuset/foreground/cpus 0-3
write /dev/cpuset/foreground/boost/cpus 0-3
write /dev/cpuset/background/cpus 0-3
write /dev/cpuset/system-background/cpus 0-3
write /dev/cpuset/top-app/cpus 0-7
write /dev/cpuset/foreground/cpus 0-7
write /dev/cpuset/foreground/boost/cpus 0-7
write /dev/cpuset/background/cpus 0-7
write /dev/cpuset/system-background/cpus 0-7
```

//官方默认配置
```bash
write /dev/cpuset/top-app/cpus 0-7
write /dev/cpuset/foreground/cpus 0-7
write /dev/cpuset/foreground/boost/cpus 4-7
write /dev/cpuset/background/cpus 0-7
write /dev/cpuset/system-background/cpus 0-3
```


```bash
adb shell cat /dev/cpuset/top-app/cpus
adb shell cat /dev/cpuset/foreground/cpus
adb shell cat /dev/cpuset/foreground/boost/cpus
adb shell cat /dev/cpuset/background/cpus
adb shell cat /dev/cpuset/system-background/cpus
```

// 查看

```bash
adb shell cat /dev/cpuset/top-app/cpus
0-7
```

对应的，可以在每个 cpuset 组的 tasks 节点下面看有哪些进程和线程是跑在这个组里面的
```bash
adb shell cat /dev/cpuset/top-app/tasks
```

部分进程也可以在启动的时候就配置好跑到哪个进程里面，下面是 lmkd 的启动配置，writepid /dev/CPUset/system-background/tasks 这一句把自己安排到了 system-background 这个组里面
```bash
service lmkd /system/bin/lmkd
class core
user lmkd
group lmkd system readproc
capabilities DAC_OVERRIDE KILL IPC_LOCK SYS_NICE SYS_RESOURCE BLOCK_SUSPEND
critical
socket lmkd seqpacket 0660 system system
writepid /dev/CPUset/system-background/tasks
```

大部分 App 进程是根据状态动态去变化的,在 Process 这个类中有详细的定义

/frameworks/base/core/java/android/os/Process.java
```bash
public static final int THREAD_GROUP_DEFAULT = -1;
public static final int THREAD_GROUP_BG_NONINTERACTIVE = 0;
private static final int THREAD_GROUP_FOREGROUND = 1;
public static final int THREAD_GROUP_SYSTEM = 2;
public static final int THREAD_GROUP_AUDIO_APP = 3;
public static final int THREAD_GROUP_AUDIO_SYS = 4;
public static final int THREAD_GROUP_TOP_APP = 5;
public static final int THREAD_GROUP_RT_APP = 6;
public static final int THREAD_GROUP_RESTRICTED = 7;
```
在 OomAdjuster 中会动态根据进程的状态修改其对应的 CPUset 组， 详细可以自行查看 OomAdjuster 中 computeOomAdjLocked、updateOomAdjLocked、applyOomAdjLocked 的执行逻辑(Android 10)

配置 affinity

使用 affinity 也可以设置任务跑在哪个核心上，其系统调用的 taskset， taskset 用来查看和设定“CPU 亲和力”，其实就是查看或者配置进程和 CPU 的绑定关系，让某进程在指定的 CPU 核上运行，即是“绑核”。

taskset 的用法

```bash
显示进程运行的CPU：
adb shell taskset -p pid
```

注意，此命令返回的是十六进制的，转换成二进制后，每一位对应一个逻辑 CPU，低位是 0 号CPU，依次类推。如果每个位置上是1，表示该进程绑定了该 CPU。例如，0101 就表示进程绑定在了 0 号和 3 号逻辑 CPU 上了

例如：

```bash
adb shell "ps -AT | grep -i messa"
u0_a66 12988 12988 711 5666492 159396 ep_poll 0 S droid.messaging

adb shell taskset -p 12988
pid 12988's current affinity mask: 6f
```
6f为：转为二进制： 110 1111 在012356号CPU上


绑核设定
```bash
taskset -pc 3 pid 表示将进程pid绑定到第3个核上
taskset -c 3 command 表示执行 command 命令，并将 command 启动的进程绑定到第3个核上。
```

Android 中也可以使用这个系统调用，把任务绑定到某个核心上运行。部分较老的内核里面不支持 CPUset，就会用 taskset 来设置

调度算法

在 Linux 的调度算法中修改调度逻辑，也可以让指定的 task 跑在指定的核上面，部分厂家的核调度优化就是使用的这种方法，这里就不具体来讲了


锁频

正常情况下，CPU 的调度算法都可以满足日常的使用，但是在 Android 中的部分场景里面，单纯依靠调度器，可能会无法满足这个场景对性能的要求。

比如说应用启动场景，如果让调度器去拉频率迁核，可能就会有一定的延迟，比如任务先在小核跑，发现小核频率不够，那就把小核频率往上拉，拉上去之后发现可能还是不够，

经过几次一直拉到最高发现还是不够，然后把这个任务迁移到中核，频率也是一次一次拉，拉到最高发现还是不够，最好迁移到大核去做。这样一套下来，时间过去不少不说，启动速度也不是最快的

基于这种情况的考虑，系统中一般都会在这种特殊场景直接暴力拉核，将硬件资源直接拉到最高去运行，比如 CPU、GPU、IO、BUS 等；

另外也会在某些场景把某些资源限制使用，比如发热太严重的时候，需要限制 CPU 的最高频率，来达到降温的目的；有时候基于功耗的考虑，也会限制一些资源在某些场景的使用


目前 Android 系统一般会在下面几个场景直接进行锁频（不同厂家也会自己定制）
```bash
应用启动
应用安装
转屏
窗口动画
List Fling
Game
```

以高通平台为例，在 CPU Info 中我们也可以看到锁频的情况

---
# bugreport中的cpu文件

cpuinfo
```bash
processor    : 0
model name    : ARMv7 Processor rev 4 (v7l)
BogoMIPS    : 24.00
Features    : half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm aes pmull sha1 sha2 crc32 
CPU implementer    : 0x41
CPU architecture: 7
CPU variant    : 0x0
CPU part    : 0xd03
CPU revision    : 4
......
processor    : 3
model name    : ARMv7 Processor rev 4 (v7l)
BogoMIPS    : 24.00
Features    : half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm aes pmull sha1 sha2 crc32 
CPU implementer    : 0x41
CPU architecture: 7
CPU variant    : 0x0
CPU part    : 0xd03
CPU revision    : 4

Hardware    : Novatek Cortex-A53
Revision    : 0000
Serial        : 0000000000000000
```

---

# cpuFreq的策略

查看当前支持的governor（手机型号可能略有不同）     

```bash
adb shell cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors
sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors                
conservative ondemand userspace powersave performance schedutil
```



```bash
performance表示不降频，以最高频率运行；
ondemand表示使用内核提供的功能，可以动态调节频率，平时以低速方式运行，当系统负载提高时按需自动提高频率；
conservative表示传统保守的，和ondemand类似，区别在于动态调节频率时采用渐进的方式；
powersvae表示省电模式，通常是在最低频率下运行，即scaling_min_freq；
userspace表示用户模式，在此模式下允许其他用户程序调节CPU频率。让根用户通过sys节点scaling_setspeed设置频率；
schedutil  userdebug 默认
```

scaling_governor：通过echo命令，能够改变当前处理器的governor类型
```bash
echo userspace > sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
echo 70000 > sys/devices/system/cpu/cpu0/cpufreq/scaling_setspeed
```


---

# CPU 状态

CPU info 中还有标识 CPU 状态的标记，如下图所示，CPU 状态有 0 ，1，2，3 这四种

下面是摘抄的其他平台的支持 C0-C4 的处理器的状态和功耗状态，Android 中不同的平台表现不一致，大家可以做一下参考

- C0 状态（激活）
这是 CPU 最大工作状态，在此状态下可以接收指令和处理数据，所有现代处理器必须支持这一功耗状态

- C1 状态（挂起）
可以通过执行汇编指令“ HLT （挂起）”进入这一状态，唤醒时间超快！（快到只需 10 纳秒！）
可以节省 70% 的 CPU 功耗，所有现代处理器都必须支持这一功耗状态

- C2 状态（停止允许）
处理器时钟频率和 I/O 缓冲被停止
换言之，处理器执行引擎和 I/0 缓冲已经没有时钟频率
在 C2 状态下也可以节约 70% 的 CPU 和平台能耗
从 C2 切换到 C0 状态需要 100 纳秒以上

- C3 状态（深度睡眠）
总线频率和 PLL 均被锁定，在多核心系统下，缓存无效
在单核心系统下，内存被关闭，但缓存仍有效可以节省 70% 的 CPU 功耗，但平台功耗比 C2 状态下大一些
唤醒时间需要 50 微妙


| 状态 | 功耗(nW) | 延迟(us) |
|------|----------|----------|
| C0   | -1       | 0        |
| C1   | 1000     | 1        |
| C2   | 500      | 1        |
| C3   | 100      | 57       |


---

# [FAQ04205] [Power] 怎样察看当前系统某个核的频率
察看当前系统某个核的频率的方法： 

查看cpu0当前的频率的方法：

```bash
adb shell cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq
```
如果当前cpu1也在运行的话，察看其当前频率的方法：
```bash
adb shell cat /sys/devices/system/cpu/cpu1/cpufreq/scaling_cur_freq
```

---

# cpu性能优先

kba-181112174319_1_game_performance_issue_debugging_introduction.pdf


```bash
#cpufreq performance 
adb shell "echo performance > /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor"
```

---

# Scheduler

Task placement goal: save power as much as possible, minimize hurting performance 
- Big task -> eligible to perf cpu : task demand higher than sched_upmigrate
- Small task -> eligible to power cpu: task demand lower than sched_downmigrate 
- Task packing: put new eligible running task in to run queue of cpu to avoid wake up cpus 
- Load balance: keep load on each cpu is “balanced” 
- Affinity: task is pinned to certain cpu 
- Sched_boost: force move task to perf cpu to improve performance

---

# 进入游戏模式

put into perf mode, 59fps 

```bash
adb shell stop perfd 
adb shell stop thermal-engine 
adb shell "echo Y > /sys/module/lpm_levels/parameters/sleep_disabled" 
adb shell "echo 0 > /sys/module/msm_thermal/parameters/enabled“ 
adb shell "echo performance >/sys/class/devfreq/X/governor“ 
adb shell "echo performance > sys/devices/system/cpu/cpuX/cpufreq/scaling_governor"
```


---

# Game design issue hw capability

![Game_design_issue](./image/Game_design_issue.png)

---

# cpu工作频率（CPU设置）--应用宝中搜索

com.zhanhong.cpusettings


```bash
adb shell pm path com.zhanhong.cpusettings
[ro.vendor.qti.core_ctl_max_cpu]: [6]
[ro.vendor.qti.core_ctl_min_cpu]: [4]
```

---

# [FAQ17683] 如何调整CPU corenum, freq, policy
[DESCRIPTION]

设置平台CPUfreq 与以及core 

[SOLUTION]

cpufreq控制结点位于 /sys/devices/system/cpu/cpu0/cpufreq/
```bash
C:\Users\mtk71247>adb shell
root@NOBLEX:/ # cd sys/devices/system/cpu/cpu0/cpufreq
cd sys/devices/system/cpu/cpu0/cpufreq
root@NOBLEX:/sys/devices/system/cpu/cpu0/cpufreq # ls
```

```bash
ls
cpuinfo_cur_freq： 当前cpu正在运行的工作频率
cpuinfo_max_freq：该文件指定了处理器能够运行的最高工作频率 （单位: 千赫兹）
cpuinfo_min_freq ：该文件指定了处理器能够运行的最低工作频率 （单位: 千赫兹）
cpuinfo_transition_latency：该文件定义了处理器在两个不同频率之间切换时所需要的时间  （单位： 纳秒）
scaling_available_frequencies：所有支持的主频率列表  （单位: 千赫兹）
scaling_available_governors：该文件显示当前内核中支持的所有cpufreq governor类型
scaling_cur_freq：被governor和cpufreq核决定的当前CPU工作频率。该频率是内核认为该CPU当前运行的主频率
scaling_driver：该文件显示该CPU正在使用何种cpufreq driver
scaling_governor：通过echo命令，能够改变当前处理器的governor类型
scaling_max_freq：显示当前policy的上下限  （单位: 千赫兹）需要注意的是，当改变cpu policy时，需要首先设置scaling_max_freq, 然后才是scaling_min_freq
scaling_setspeed：如果用户选择了“userspace” governor, 那么可以设置cpu工作主频率到某一个指定值。只需要这个值在scaling_min_freq 和 scaling_max_freq之间即可。
```



1、查看当前CPU支持的频率档位
```bash
root@NOBLEX:/sys # cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies
sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies              
1300000 1235000 1170000 1040000 819000 598000 442000 299000
```


2、查看当前支持的governor（手机型号可能略有不同）  
```bash
root@NOBLEX:/sys # cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors
sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors                
ondemand userspace powersave interactive performance

performance表示不降频，
ondemand表示使用内核提供的功能，可以动态调节频率，
powersvae表示省电模式，通常是在最低频率下运行，
userspace表示用户模式，在此模式下允许其他用户程序调节CPU频率。
```   


3、查看当前选择的governor
```bash
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
interactive
```


4、查看系统支持多少核数
```bash
root@NOBLEX:/ # cat sys/devices/system/cpu/present
cat sys/devices/system/cpu/present
0-3
```

5、全开所有cpu ，在实际设置时，还需要（有root权限才可以设置）
```bash
adb shell "echo 0 > /proc/hps/enabled" (关闭cpu hotplug)
adb shell "echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor" (固定最高频)
echo 1 > /sys/devices/system/cpu/cpuX/online
X表示(0~3，不同平台CPU core 数是不一样的)
```


例：6735平台
```bash
root@NOBLEX:/ # echo 1 >sys/devices/system/cpu/cpu1/online
echo 1 >sys/devices/system/cpu/cpu1/online
root@NOBLEX:/ # echo 1 >sys/devices/system/cpu/cpu2/online
echo 1 >sys/devices/system/cpu/cpu2/online
root@NOBLEX:/ # echo 1 >sys/devices/system/cpu/cpu3/online
echo 1 >sys/devices/system/cpu/cpu3/online
```


6、设置频率(可以先cat 出来当前的频率有哪些)
```bash
C:\Users\mtk71247>adb shell "cat /proc/cpufreq/cpufreq_ptpod_freq_volt"
[0] = { .cpufreq_khz = 1300000, .cpufreq_volt = 113750, .cpufreq_volt_org = 1250
00, },
[1] = { .cpufreq_khz = 1235000, .cpufreq_volt = 110000, .cpufreq_volt_org = 1231
25, },
[2] = { .cpufreq_khz = 1170000, .cpufreq_volt = 106250, .cpufreq_volt_org = 1206
25, },
[3] = { .cpufreq_khz = 1040000, .cpufreq_volt = 98750,  .cpufreq_volt_org = 1150
00, },
[4] = { .cpufreq_khz = 819000,  .cpufreq_volt = 95000,  .cpufreq_volt_org = 1100
00, },
[5] = { .cpufreq_khz = 598000,  .cpufreq_volt = 95000,  .cpufreq_volt_org = 1050
00, },
[6] = { .cpufreq_khz = 442000,  .cpufreq_volt = 95000,  .cpufreq_volt_org = 1000
00, },
[7] = { .cpufreq_khz = 299000,  .cpufreq_volt = 95000,  .cpufreq_volt_org = 9500
0, },
```


设置后再cat 看一下当前的设置是否成功
```bash
C:\Users\mtk71247>adb shell "cat proc/cpufreq/cpufreq_oppidx"
[MT_CPU_DVFS_LITTLE/0]
cpufreq_oppidx = 0
        OP(1300000, 113750),
        OP(1235000, 110000),
        OP(1170000, 106250),
        OP(1040000, 98750),
        OP(819000, 95000),
        OP(598000, 95000),
        OP(442000, 95000),
        OP(299000, 95000),
```


---

# 设置固定CPU最大值：

```bash
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_max_freq
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_min_freq

i7s:/ # cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_min_freq
576000
576000
576000
576000
576000
576000
652800
652800

echo 1708800 > /sys/devices/system/cpu/cpu0/cpufreq/scaling_min_freq ;
echo 1708800 > /sys/devices/system/cpu/cpu1/cpufreq/scaling_min_freq ;
echo 1708800 > /sys/devices/system/cpu/cpu2/cpufreq/scaling_min_freq ;
echo 1708800 > /sys/devices/system/cpu/cpu3/cpufreq/scaling_min_freq ;
echo 1708800 > /sys/devices/system/cpu/cpu4/cpufreq/scaling_min_freq ;
echo 1708800 > /sys/devices/system/cpu/cpu5/cpufreq/scaling_min_freq ;
echo 2208000 > /sys/devices/system/cpu/cpu6/cpufreq/scaling_min_freq ;
echo  2208000 > /sys/devices/system/cpu/cpu7/cpufreq/scaling_min_freq ;

cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_min_freq
cat /sys/devices/system/cpu/cpu*/cpufreq/cpuinfo_cur_freq
```

高通：
```bash
固定最高频
adb shell "echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor"

adb shell "echo performance > /sys/devices/system/cpu/cpufreq/policy0/scaling_governor"
adb shell "echo performance > /sys/devices/system/cpu/cpufreq/policy4/scaling_governor"
```

system/core/rootdir/init.rc
```bash
on early-init
write /sys/devices/system/cpu/cpufreq/policy0/scaling_governor performance

.......
on property:sys.boot_completed=1
    write schedutil /sys/devices/system/cpu/cpufreq/policy0/scaling_governor
```

MTK：

固定频率 mtk的接口是/proc/ppm/policy/ut_fix_freq_idx：
```bash
write /proc/ppm/policy/ut_fix_freq_idx  "0 0 0" 
// 6797是3个cpu簇，000 标识3簇都是固定最高频率
```
如果只是写 performance 到 scaling_governor，那平台的动态cpu调度策略还会起作用【会动态降频】

---

# [FAQ04204] [Power] 怎样将手机中的频率固定在某一个level

将手机中的频率固定在单核某个频率xxx的方法： 

```bash
echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
echo 0 > /sys/devices/system/cpu/cpu1/online
echo 0 > /proc/mtk_hotplug/enable
echo xxx > /sys/power/cpufreq_limited_freq
```

将手机中的频率固定在双核某个频率xxx的方法：

```bash
echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
echo 0 > /sys/devices/system/cpu/cpu1/online
echo performance > /sys/devices/system/cpu/cpu1/cpufreq/scaling_governor
echo 0 > /proc/mtk_hotplug/enable
echo xxx > /sys/power/cpufreq_limited_freq
```
 以上方法可以保证不管是亮屏还是灭屏，都可以固定在您设定的频率下运行。

---

# 查看各温度的类型(包括cpu)

```bash
cat /sys/class/thermal/thermal_zone*/type 
```

查看各sensor的温度:
```bash
cat /sys/class/thermal/thermal_zone*/temp
```

[DESCRIPTION]

  获取thermal sensor 实时温度温度的方法

[SOLUTION]

1、透过ADB，及SW 接口获取 
    user层可以通过这个路径获得CPU的温度  
```bash
cat /sys/class/thermal/thermal_zone*/temp
   每个thermal_zone代表的sensor，可以 cat /sys/class/thermal/thermal_zone*/type

其中* 是指的0，1，2，3，4，5........
driver层您可以调用如下sw 接口：   
mediatek\kernel\drivers\thermal\mtk_thermal_monitor.c
mtk_thermal_get_temp(ID)，其中ID的定义可以通过结构体MTK_THERMAL_SENSOR_ID获取
```
2、工程模式下

  *#*#3646633#*#*

others-thermal-thermal sensors 可以获得各个sensor实时 温度

https://online.mediatek.com/FAQ#/HW/FAQ05485

ANS：连上adb，进入adb shell之后，可通过以下指令获取。

```bash
CPU real temp
cat  /sys/class/thermal/thermal_zone0/temp
Battery real temp
cat  /sys/class/thermal/thermal_zone1/temp
PA real temp
cat  /sys/class/thermal/thermal_zone2/temp
ABB real temp
cat  /sys/class/thermal/thermal_zone3/temp
PMIC real temp
cat  /sys/class/thermal/thermal_zone4/temp
WMT real temp
cat  /sys/class/thermal/thermal_zone5/temp
```




```bash
i7s:/sys/class/thermal/thermal_zone24 # cat type
gpu0-usr------------GPU温度
i7s:/sys/class/thermal/thermal_zone25 # cat type
gpu1-usr------------GPU温度

i7s:/sys/class/thermal/thermal_zone43 # cat type
battery------------电池温度
```

---

# [FAQ15144] 如何手动去设置CPU核数，关闭thermal验证performance问题

[DESCRIPTION]

 因CPU或者DVFS设置不同，或Thermal过高而导致性能下降，如何通过adb命令进行设置 

[SOLUTION]

1.对于是否由于CPU或者DVFS不同引起的问题，可以通过下面的命令验证：

- 首先取得root权限: 请参考FAQ11862 user版本如何打开root权限  

- 手动设定CPU core数量：    
```bash
        setup：(务必先下setup部分，才能下定频定核相关命令)
         不同平台，设置会有差别，下面针对MT6795 相关类似平台
          adb shell "echo 0 > /proc/hps/enabled"    (关闭cpu hotplug)
          adb shell "echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor" (固定最高频)
          开启CPU1～CPU7（CPU0 always on）
          adb shell "echo 1 > /sys/devices/system/cpu/cpu1/online"
          adb shell "echo 1 > /sys/devices/system/cpu/cpu2/online"
            ......
          关闭CPU1～CPU7（CPU0 always on）
          adb shell "echo 0 > /sys/devices/system/cpu/cpu1/online"
        adb shell "echo 0 > /sys/devices/system/cpu/cpu2/online"
            ......
    Note: echo 1 打开，echo 0 关掉
```

- 恢复最初 cpu core设置
```bash
    adb shell "echo 1 > /proc/hps/enabled"
    adb shell "echo interactive > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor"
```

- 手动设定大小核freq
```bash
    请根据oppidx档位设定：（oppidx档位请参考以下cat 出来的idx）
    adb shell "cat/proc/cpufreq/cpufreq_ptpod_freq_volt"
    如：（设定最高频率）
    adb shell "echo 0 >/proc/cpufreq/cpufreq_oppidx"  (idx ：0  is CPU frequency mapping)
```

   2. 对于是否系统过热而引起的性能差异判断方式：

-  为避免thermal关闭cpu，导致performance差异 ，关闭thermal测试：
```bash
adb shell "echo 120000 130000 >/proc/cpufreq/cpufreq_ptpod_temperature_limit"
adb shell "/system/bin/thermal_manager/etc/.tp/.th120.mtc"  （重启后失效）
adb shell "echo 0 > /proc/cpufreq/cpufreq_limited_power"
```


---

# Low power mode(LPM)
```bash
adb shell "echo Y > /sys/module/lpm_levels/parameters/sleep_disabled“

i7s:/sys/module/lpm_levels # cat parameters/sleep_disabled
N
```


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p35-系统开发之cpu)

---


[上一篇文章 P34_系统开发之Display](https://github.com/hfreeman2008/android_core_framework/blob/main/P34_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay.md)



---

# 结束语

<img src="../Images/end_001.png">