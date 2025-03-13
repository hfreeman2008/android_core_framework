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