# 专题十二_系统开发之浅谈WindowManagerService

<img src="flows_wms_001.png">



# WindowManagerService 类的作用：

wms是窗口的管理者，负责窗口的启动，添加，删除和更新，窗口动画，窗口的大小和层级，还有壁纸，水印，输入系统的中转站，Surface管理等各个方面的一个综合管理体。

# 获取wms的方式：

```java
方式1
WindowManager mWindowManager;
mWindowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);


方式2
WindowManager mWindowManager;
mWindowManager = mContext.getSystemService(WindowManager.class);


方式3
IWindowManager sWindowManagerService;
sWindowManagerService = IWindowManager.Stub.asInterface(ServiceManager.getService("window"));


方式4
WindowManagerInternal mWindowManagerService;
mWindowManagerService = WindowManagerGlobal.getWindowManagerService();

方式5
WindowManagerInternal mWindowManagerService;
mWindowManagerService = LocalServices.getService(WindowManagerInternal.class);
```

# WindowManagerService调用流程

<img src="wms_whole.png">

以isKeyguardLocked为例，查看WindowManagerService调用流程：

(1)app应用中调用isKeyguardLocked:

```java
WindowManagerGlobal.getWindowManagerService().isKeyguardLocked()
```

(2)获取IWindowManager服务

 WindowManagerGlobal.getWindowManagerService()

```java
public static IWindowManager getWindowManagerService() {
    synchronized (WindowManagerGlobal.class) {
        if (sWindowManagerService == null) {
            sWindowManagerService = IWindowManager.Stub.asInterface(
                    ServiceManager.getService("window"));
            try {
                if (sWindowManagerService != null) {
                    ValueAnimator.setDurationScale(
                            sWindowManagerService.getCurrentAnimatorScale());
                    sUseBLASTAdapter = sWindowManagerService.useBLAST();
                }
            } catch (RemoteException e) {
                throw e.rethrowFromSystemServer();
            }
        }
        return sWindowManagerService;
    }
}
```

(3)IWindowManager.aidl定义isKeyguardLocked

```java
@UnsupportedAppUsage
boolean isKeyguardLocked();
```

(4)WindowManagerService.isKeyguardLocked

```java
public boolean isKeyguardLocked() {
    return mPolicy.isKeyguardLocked();
}
```

(5)WindowManagerPolicy.isKeyguardLocked

```java
public boolean isKeyguardLocked();
```

# WindowManagerService添加view的详细分析

参考：

[android开发浅谈之Window和WindowManager](https://blog.csdn.net/hfreeman2008/article/details/111882109)

https://blog.csdn.net/hfreeman2008/article/details/111882109


[android开发浅谈之DecorView与ViewRootImpl](https://blog.csdn.net/hfreeman2008/article/details/111913489)

https://blog.csdn.net/hfreeman2008/article/details/111913489


[android开发浅谈之View测量流程(Measure)](https://blog.csdn.net/hfreeman2008/article/details/111996784)

https://blog.csdn.net/hfreeman2008/article/details/111996784




# 以addView()接口为例，看一下其调用流程







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


```java

```





# 结束语

<img src="../Images/end_001.png">

