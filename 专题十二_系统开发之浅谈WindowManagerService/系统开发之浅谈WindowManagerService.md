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

