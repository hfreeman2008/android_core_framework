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

![View](./image/View.png)


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



## InputManagerService

### 消息监听-notifySwitch
该方法为Native的回调方法，用于上报IMS事件，如耳机插入事件等。
```java
private void notifySwitch(long whenNanos, int switchValues, int switchMask) {
    if (DEBUG) {
        Slog.d(TAG, "notifySwitch: values=" + Integer.toHexString(switchValues)
                + ", mask=" + Integer.toHexString(switchMask));
    }

    if ((switchMask & SW_LID_BIT) != 0) {
        final boolean lidOpen = ((switchValues & SW_LID_BIT) == 0);
        mWindowManagerCallbacks.notifyLidSwitchChanged(whenNanos, lidOpen);
    }

    if ((switchMask & SW_CAMERA_LENS_COVER_BIT) != 0) {
        final boolean lensCovered = ((switchValues & SW_CAMERA_LENS_COVER_BIT) != 0);
        mWindowManagerCallbacks.notifyCameraLensCoverSwitchChanged(whenNanos, lensCovered);
    }

    if (mUseDevInputEventForAudioJack && (switchMask & SW_JACK_BITS) != 0) {
        mWiredAccessoryCallbacks.notifyWiredAccessoryChanged(whenNanos, switchValues,
                switchMask);
    }

    if ((switchMask & SW_TABLET_MODE_BIT) != 0) {
        SomeArgs args = SomeArgs.obtain();
        args.argi1 = (int) (whenNanos & 0xFFFFFFFF);
        args.argi2 = (int) (whenNanos >> 32);
        args.arg1 = Boolean.valueOf((switchValues & SW_TABLET_MODE_BIT) != 0);
        mHandler.obtainMessage(MSG_DELIVER_TABLET_MODE_CHANGED,
                args).sendToTarget();
    }

    if ((switchMask & SW_MUTE_DEVICE_BIT) != 0) {
        final boolean micMute = ((switchValues & SW_MUTE_DEVICE_BIT) != 0);
        AudioManager audioManager = mContext.getSystemService(AudioManager.class);
        audioManager.setMicrophoneMuteFromSwitch(micMute);
    }
}
```


### notifyANR-anr相关

```java
private long notifyANR(InputApplicationHandle inputApplicationHandle, IBinder token,
        String reason) {
    return mWindowManagerCallbacks.notifyANR(inputApplicationHandle,
            token, reason);
}
```

### interceptKeyBeforeDispatching--key事件分发

```java
private long interceptKeyBeforeDispatching(IBinder focus, KeyEvent event, int policyFlags) {
    return mWindowManagerCallbacks.interceptKeyBeforeDispatching(focus, event, policyFlags);
}
```

InputManagerCallback.java
```java
/**
 * Provides an opportunity for the window manager policy to process a key before
 * ordinary dispatch.
 */
@Override
public long interceptKeyBeforeDispatching(
        IBinder focusedToken, KeyEvent event, int policyFlags) {
    return mService.mPolicy.interceptKeyBeforeDispatching(focusedToken, event, policyFlags);
}
```

WindowManagerPolicy
```java
long interceptKeyBeforeDispatching(IBinder focusedToken, KeyEvent event, int policyFlags);
```

PhoneWindowManager
```java
public class PhoneWindowManager implements WindowManagerPolicy {
    @Override
public long interceptKeyBeforeDispatching(IBinder focusedToken, KeyEvent event,
        int policyFlags) {
            ......
        }    
}
```


## InputManager

frameworks\native\services\inputflinger\InputManager.cpp

启动事件管理服务

启动两个核心的阻塞线程，一个是事件分发线程，一个是事件读取线程。

```cpp
status_t InputManager::start() {
    status_t result = mDispatcher->start();//启动事件分发服务
    if (result) {
        ALOGE("Could not start InputDispatcher thread due to error %d.", result);
        return result;
    }

    result = mReader->start();//启动事件读取服务
    if (result) {
        ALOGE("Could not start InputReader due to error %d.", result);

        mDispatcher->stop();
        return result;
    }

    return OK;
}
```


## EventHub

EventHub：事件集线器，它将全部的输入事件通过一个接口getEvents()，将从多个输入设备节点中读取的事件交给InputReader，是输入系统最底层的一个组件。

(1)RawEvent结构体

mEventBuffer用于描述原始输入事件，其类型为RawEvent，相关结构体如下:

frameworks\native\services\inputflinger\reader\include\EventHub.h

```java
/*
 * A raw event as retrieved from the EventHub.
 */
struct RawEvent {
    nsecs_t when;//事件时间戳
    int32_t deviceId;//产生事件的设备ID
    int32_t type;//事件类型
    int32_t code;//事件编码
    int32_t value;//事件值
};
```


(2)EventHub->getEvents事件

getEvents()是事件处理的核心方法，其通过EPOLL机制和INOTIFY，从多个设备节点读取事件。

```c++
size_t EventHub::getEvents(int timeoutMillis, RawEvent* buffer, size_t bufferSize) {
    ......
    struct input_event readBuffer[bufferSize];

    RawEvent* event = buffer;
    size_t capacity = bufferSize;
    bool awoken = false;
    for (;;) {
        nsecs_t now = systemTime(SYSTEM_TIME_MONOTONIC);
        ......
        if (mNeedToScanDevices) {//Step 1.扫描设备
            mNeedToScanDevices = false;
            scanDevicesLocked();
            mNeedToSendFinishedDeviceScan = true;
        }


        // Grab the next input event.
        bool deviceChanged = false;
        while (mPendingEventIndex < mPendingEventCount) {//Step 2.处理未被InputReader取走的输入事件与设备事件
            const struct epoll_event& eventItem = mPendingEventItems[mPendingEventIndex++];
            if (eventItem.data.fd == mINotifyFd) {
                if (eventItem.events & EPOLLIN) {
                    mPendingINotify = true;
                } else {
                    ALOGW("Received unexpected epoll event 0x%08x for INotify.", eventItem.events);
                }
                continue;
            }
           ......
            // This must be an input event
            if (eventItem.events & EPOLLIN) {
                //Step 3.读取底层上报事件
                int32_t readSize =
                        read(device->fd, readBuffer, sizeof(struct input_event) * capacity);
                if (readSize == 0 || (readSize < 0 && errno == ENODEV)) {
                    // Device was removed before INotify noticed.
                    ALOGW("could not get event, removed? (fd: %d size: %" PRId32
                          " bufferSize: %zu capacity: %zu errno: %d)\n",
                          device->fd, readSize, bufferSize, capacity, errno);
                    deviceChanged = true;
                    closeDeviceLocked(device);
                } else if (readSize < 0) {
                    if (errno != EAGAIN && errno != EINTR) {
                        ALOGW("could not get event (errno=%d)", errno);
                    }
                } else if ((readSize % sizeof(struct input_event)) != 0) {
                    ALOGE("could not get event (wrong size: %d)", readSize);
                } else {
                    int32_t deviceId = device->id == mBuiltInKeyboardId ? 0 : device->id;

                    size_t count = size_t(readSize) / sizeof(struct input_event);
                    //构建需要上报的事件
                    for (size_t i = 0; i < count; i++) {
                        struct input_event& iev = readBuffer[i];
                        event->when = processEventTimestamp(iev);
                        event->deviceId = deviceId;
                        event->type = iev.type;
                        event->code = iev.code;
                        event->value = iev.value;
                        event += 1;//将event指针移动到下一个可用于填充事件的RawEvent对象
                        capacity -= 1;
                    }
        ......
        mLock.unlock(); // release lock before poll
        //Step 4.阻塞，等待事件各种类型消息
        int pollResult = epoll_wait(mEpollFd, mPendingEventItems, EPOLL_MAX_EVENTS, timeoutMillis);
        mLock.lock(); // reacquire lock after poll
      ......
    // All done, return the number of events we read.
    return event - buffer;
}
```

- Step 1. 扫描设备，会获取input/dev/下的所有设备，并将各个设备注册到epoll线程池里，监听各个设备的消息状态；
- Step 2. 处理未被InputReader取走的输入事件与设备事件，一般情况下有事件上报时，epoll_wait会读取到mPendingEventItems值，即mPendingEventCount值，即会进入该流程；
- Step 3. 读取底层上报事件，根据上报的fd设备，读取对应的设备节点。即可以获取到上报的事件内容。如下为屏幕点击对应的上报事件：
- Step 4. 通过epoll机制，阻塞当前进程，等待设备节点变更，事件上报。


![EventHub](EventHub.png)



## InputReader
事件读取服务，读取驱动上报事件



```java

```


```java

```


```java

```


```java

```


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
