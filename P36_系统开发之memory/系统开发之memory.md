# P36: 系统开发之memory

<img src="../flower/flower_p26.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

[上一篇文章 P35_系统开发之cpu](https://github.com/hfreeman2008/android_core_framework/blob/main/P35_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bcpu/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bcpu.md)






---

# memory脑图

![memory脑图](./image/memory脑图.png)


---

# Java 内存分配模型

1）、方法区：存储类信息、常量、静态变量等。=> 所有线程共享

2）、虚拟机栈：存储局部变量表、操作数栈等。

3）、本地方法栈：不同与虚拟机栈为 Java 方法服务、它是为 Native 方法服务的。

4）、堆：内存最大的区域，每一个对象实际分配内存都是在堆上进行分配的，，而在虚拟机栈中分配的只是引用，这些引用会指向堆中真正存储的对象。此外，堆也是垃圾回收器（GC）所主要作用的区域，并且，内存泄漏也都是发生在这个区域。=> 所有线程共享

5）、程序计数器：存储当前线程执行目标方法执行到了第几行


---

# 内存分配策略

- 静态存储区（方法区）

内存在程序编译的时候就已经分配好，这块内存在程序整个运行期间都存在，它主要是用来存放静态数据、全局 static 数据和常量

- 栈区

在执行函数时，函数内部局部变量的存储单元都可以在栈上创建，函数执行结束时这些存储单元自动被释放，栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限

- 堆区(heap)

动态内存分配，Java/Android 程序在适当的时候使用 new 关键字申请所需要大小的对象内存，然后通过 GC 决定在不需要这块对象内存的时候回收它，但是由于我们的疏忽导致该对象在不需要继续使用的之后，GC 仍然没办法回收该内存区域，这就代表发生了内存泄漏


---

# 四大内存指标：

VSS - Virtual Set Size 虚拟耗用内存（包含共享库占用的内存）

RSS - Resident Set Size 实际使用物理内存（包含共享库占用的内存）

PSS - Proportional Set Size 实际使用的物理内存（比例分配共享库占用的内存）

USS - Unique Set Size 进程独自占用的物理内存（不包含共享库占用的内存）

### 内存指标中英文对照表

| 英文简称 | 英文全称               | 内存类型 | 核心含义                                                                 | 等价关系                          |
|----------|------------------------|----------|-------------------------------------------------------------------------|-----------------------------------|
| USS | Unique Set Size        | 物理内存 | 进程独占的物理内存（不包含共享库）                                       | `USS = 私有内存`                 |
| PSS | Proportional Set Size  | 物理内存 | USS + 按比例分配的共享库内存（例如：共享库内存 ÷ 共享进程数）            | `PSS = USS + 共享库分摊值`       |
| RSS  | Resident Set Size      | 物理内存 | USS + 所有共享库内存（包含重复计算的共享部分）                           | `RSS = USS + 共享库内存`         |
| VSS  | Virtual Set Size       | 虚拟内存 | 进程可访问的虚拟内存总量（含未实际分配的虚拟内存）                       | `VSS = RSS + 未分配物理内存`     |



从上可知,它们之间内存的大小关系为VSS >= RSS >= PSS >= USS.



---

# adb shell dumpsys meminfo 
适用场景： 

查看进程的oom adj，或者dalvik/native等区域内存情况，或者某个进程或apk的内存情况；

```bash
Applications Memory Usage (in Kilobytes):
Uptime: 24044 Realtime: 24044

Total PSS by process:
     79,474K: system (pid 3103)
     34,375K: zygote (pid 2684)
     23,107K: com.jamdeo.tv.vod:p0 (pid 3874)
        446K: lmkd (pid 2850)
        429K: svcxfar_x (pid 3029)
        422K: xirisvc2.4 (pid 2854)
        412K: svciflybl (pid 3072)
        371K: factory_svc (pid 2859)
          0K: bootanimation (pid 2296)
          0K: bootanimation (pid 2350)

Total PSS by OOM adjustment:
    238,861K: Native
         34,375K: zygote (pid 2684)
         19,148K: surfaceflinger (pid 2406)
            412K: svciflybl (pid 3072)
            371K: factory_svc (pid 2859)
              0K: bootanimation (pid 2296)
              0K: bootanimation (pid 2350)
     79,474K: System
         79,474K: system (pid 3103)
     59,862K: Persistent
         14,357K: com.jamdeo.tv.service (pid 3231)
         13,180K: com.android.systemui (pid 3238)
         12,624K: com.keylab.speech.core.vidaa (pid 3716)
    179,529K: Foreground
        156,422K: com.jamdeo.tv.vod (pid 3319 / activities)
         23,107K: com.jamdeo.tv.vod:p0 (pid 3874)

Total PSS by category:
    177,393K: .so mmap
    110,553K: .dex mmap
......
      1,896K: Ashmem
         16K: Cursor
          0K: Gfx dev
          0K: EGL mtrack
          0K: Other mtrack

Total RAM: 1,313,620K (status critical)
 Free RAM:   458,708K (        0K cached pss +   269,952K cached kernel +   188,756K free)
 Used RAM:   862,707K (  575,291K used pss +   287,416K kernel)
 Lost RAM:    -7,795K
   Tuning: 192 (large 256), oom    81,920K, restore limit    27,306K (high-end-gfx)
```


---


```bash

```

```bash

```




---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p36-系统开发之memory)

---


[上一篇文章 P35_系统开发之cpu](https://github.com/hfreeman2008/android_core_framework/blob/main/P35_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bcpu/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bcpu.md)



---

# 结束语

<img src="../Images/end_001.png">