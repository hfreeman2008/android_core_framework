# P32: 系统开发之IMS输入模块

<img src="../flower/flower_p22.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

[上一篇文章 P30_系统开发之RRO](https://github.com/hfreeman2008/android_core_framework/blob/main/P30_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BRRO/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BRRO.md)


---


# 输入模块框架：

![输入模块框架](./image/输入模块框架.png)

## framework

```java
frameworks/base/core/java/android/hardware/input/InputManager.java
frameworks\base\services\core\java\com\android\server\input\InputManagerService.java
frameworks/base/services/core/jni/com_android_server_input_InputManagerService.cpp
```


InputManagerService是Android框架层一个非核心服务，主要是提供一个IMS输入系统启动的入口，同时对应用层提供业务相关接口。

frameworks\base\services\core\java\com\android\server\wm\WindowManagerService.java

WindowManagerService有一个核心的功能，输入系统的中转站；

![WindowManagerService](./image/WindowManagerService.png)



frameworks\base\services\core\java\com\android\server\policy\PhoneWindowManager.java

针对key的具体处理逻辑:
```java
interceptKeyBeforeDispatching
interceptKeyBeforeQueueing
```


```java
frameworks\base\core\java\android\view\ViewRootImpl.java
frameworks\base\core\java\android\view\View.java
frameworks\base\core\java\android\view\InputEventSender.java
frameworks\base\core\java\android\view\InputEventSender.java
```

frameworks\base\core\java\android\view\View.java

![View](View.png)


## native

### libinputreader

```java
./frameworks/native/services/inputflinger/reader/EventHub.cpp
./frameworks/native/services/inputflinger/reader/include/EventHub.h

frameworks\native\services\inputflinger\reader\InputReader.cpp
frameworks\native\services\inputflinger\reader\include\InputReader.h
```




### inputflinger

input事件的管理类，数据传递类，也是输入系统native层核心的模块。
```java
frameworks\native\services\inputflinger\dispatcher\InputDispatcher.h
frameworks\native\services\inputflinger\dispatcher\InputDispatcher.cpp

frameworks/native/services/inputflinger/InputManager.h
frameworks\native\services\inputflinger\InputManager.cpp
```



## hal层

```java
hardware\libhardware\modules
hardware\libhardware\modules\input\evdev
```



## 驱动层

```java
kernel/msm-4.19/drivers/input/
kernel/msm-4.19/drivers/input/keyboard/gpio_keys.c
```


```java
/dev/input/event0
......
/dev/input/eventN
```

---

# 事件抽象接口-InputEvent

```java
frameworks\base\core\java\android\view\InputEvent.java
frameworks\base\core\java\android\view\KeyEvent.java
```

# UEVENT 机制
"uevent" 是 Linux 系统中的一种事件通知机制，用于向用户空间发送有关内核和设备状态变化的通知。这种机制通常用于设备驱动程序、热插拔事件以及设备状态变化等场景，以便用户空间应用程序能够在这些事件发生时做出相应的响应.



# 关键类



---



```java

```


```java

```

---





---











---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p32-系统开发之ims输入模块)

---


[上一篇文章 P30_系统开发之RRO](https://github.com/hfreeman2008/android_core_framework/blob/main/P30_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BRRO/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BRRO.md)



---

# 结束语

<img src="../Images/end_001.png">
