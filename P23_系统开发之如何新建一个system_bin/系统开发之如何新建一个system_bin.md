# P23: 系统开发之如何新建一个system_bin

<img src="../flower/flower_p23.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

[上一篇文章 P22_系统开发之浅谈LightsService](https://github.com/hfreeman2008/android_core_framework/blob/main/P22_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B5%85%E8%B0%88LightsService/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B5%85%E8%B0%88LightsService.md)





[下一篇文章 P24_系统开发之回调接口注册注销](https://github.com/hfreeman2008/android_core_framework/blob/main/P24_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E5%9B%9E%E8%B0%83%E6%8E%A5%E5%8F%A3%E6%B3%A8%E5%86%8C%E6%B3%A8%E9%94%80/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E5%9B%9E%E8%B0%83%E6%8E%A5%E5%8F%A3%E6%B3%A8%E5%86%8C%E6%B3%A8%E9%94%80.md)


---


# 需求：

客户要求提供一个接口，可以清除sdcard目录下的所有文件和内容


---

# 实现方案：

主要思路是：

1) 监听系统属性sys.clear_sdcard_reset.on，当系统属性sys.clear_sdcard_reset.on为1时启动cmdclearsdcard的bin文件

2) 在cmdclearsdcard中执行：

```makefile
rm -rf /sdcard/*
```

---


# 具体实现：

## (1)如何新建一个/system/bin/ bin文件cmdclearsdcard：

主要参考:frameworks/native/cmds/

以cmdclearsdcard为例：

frameworks/native/cmds/cmdclearsdcard/Android.mk

```makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_SRC_FILES:= cmdclearsdcard.c
LOCAL_MODULE:= cmdclearsdcard
LOCAL_CFLAGS := -Wall
LOCAL_SHARED_LIBRARIES := libcutils liblog
LOCAL_INIT_RC := cmdclearsdcard.rc
include $(BUILD_EXECUTABLE)
```

frameworks/native/cmds/cmdclearsdcard/cmdclearsdcard.c

```c
#define LOG_NDEBUG 0
#define LOG_TAG "cmdinitgoogletts"
#include <cutils/properties.h>
#include <cutils/android_reboot.h>
#include <utils/Log.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <sys/prctl.h>
#include <sys/stat.h>
#include <errno.h>
#include <cutils/sockets.h>

int main(){
    int exe_result;
    ALOGE("execute cmd start");
    exe_result = system("rm -rf /sdcard/*");
    ALOGE("execute rm sdcard result=%d", exe_result);

    exe_result = system("touch /sdcard/clearsdcardfinsh");
    ALOGE("execute touch result=%d", exe_result);

    exe_result = system("chmod 777 /sdcard/clearsdcardfinsh");
    ALOGE("execute chmod result=%d", exe_result);

    return 0;  
}
```


## (2)监听系统属性sys.clear_sdcard_reset.on，当系统属性sys.clear_sdcard_reset.on为1时启动cmdclearsdcard的bin文件

frameworks/native/cmds/cmdclearsdcard/cmdclearsdcard.rc

```makefile
service cmdclearsdcard /system/bin/cmdclearsdcard
    class main
    user root
    group root
    disabled
    oneshot
    seclabel u:r:cmdroot:s0
    socket cmdroot stream 660 root system

on property:sys.clear_sdcard_reset.on=1
    start cmdclearsdcard
    setprop sys.clear_sdcard_reset.on 0
```


---

## (3)定义bin

system / sepolicy/prebuilts/api/33.0/private/file_contexts

```makefile
/system/bin/cmdclearsdcard        u:object_r:cmdroot_exec:s0
```

---

## (4)添加应用到系统中：

build/make/target/product/base_system.mk


```makefile
PRODUCT_PACKAGES += \
......
    wm \
    cmdblockdev \
    cmdimproveperformance \
    cmdrecoveryperformance \
    cmdinitgoogletts \
 +   cmdclearsdcard \
```

---


## (5)添加selinux权限


---

这个方案非常有用，特别是对于监听系统属性改变来执行什么命令是特别的方便。

rss_hwm_reset也是一个非常经典的样例：
frameworks\native\cmds\rss_hwm_reset\Android.bp

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p23-系统开发之如何新建一个system_bin)

---

[上一篇文章 P22_系统开发之浅谈LightsService](https://github.com/hfreeman2008/android_core_framework/blob/main/P22_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B5%85%E8%B0%88LightsService/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B5%85%E8%B0%88LightsService.md)





[下一篇文章 P24_系统开发之回调接口注册注销](https://github.com/hfreeman2008/android_core_framework/blob/main/P24_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E5%9B%9E%E8%B0%83%E6%8E%A5%E5%8F%A3%E6%B3%A8%E5%86%8C%E6%B3%A8%E9%94%80/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E5%9B%9E%E8%B0%83%E6%8E%A5%E5%8F%A3%E6%B3%A8%E5%86%8C%E6%B3%A8%E9%94%80.md)


---

# 结束语

<img src="../Images/end_001.png">
