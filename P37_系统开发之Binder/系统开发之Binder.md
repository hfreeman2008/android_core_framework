# P37: 系统开发之Binder

<img src="../flower/flower_p27.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

[上一篇文章 P35_系统开发之cpu](https://github.com/hfreeman2008/android_core_framework/blob/main/P35_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bcpu/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bcpu.md)






---


# Binder架构


![Binder架构](./image/Binder架构1.png)


![Binder架构2](./image/Binder架构2.png)

![Binder架构3](./image/Binder架构3.png)


---

# Binder分层架构图


![Binder分层架构图1](./image/Binder分层架构图1.png)

![Binder分层架构图2](./image/Binder分层架构图2.png)


---

# 关键概念：

## Stub（存根）

定义：Stub是服务端的接口实现，继承自IBinder接口。它代表了服务端暴露给客户端调用的接口。

功能：Stub负责接收来自客户端的请求，通过onTransact方法处理客户端的调用请求。在这个方法中，Stub会根据不同的请求码（code）执行对应的服务逻辑，并通过Parcel对象传递参数和返回值。

生成：通常，使用AIDL（Android Interface Definition Language）文件定义接口时，编译器会自动为服务端生成Stub类的实现。


## Proxy（代理）
定义：Proxy是客户端侧的Stub的本地代理，同样继承自IBinder接口。它是Stub的一个远程代理对象，使得客户端可以像调用本地对象一样调用远程服务的方法。

功能：当客户端调用Proxy上的方法时，Proxy会将调用序列化（通过Parcel对象），然后通过Binder驱动发送到服务端。同时，Proxy还会监听来自服务端的返回结果，并将其反序列化后返回给客户端。

生成：同样，使用AIDL定义接口时，编译器也会为客户端自动生成一个Proxy类。



## asBinder
作用：asBinder方法用于获取Binder对象的引用，这个引用可以被用来进行跨进程通信。在Stub和Proxy中都会实现这个方法，但含义稍有不同。

对于Stub，asBinder返回的就是Stub自己，因为它本身就是Binder的一个实现，可以直接参与跨进程通信。

对于Proxy，asBinder返回的是一个Binder代理对象，这个代理对象内部会处理跨进程的调用转发，确保客户端的调用能够到达服务端的Stub。

## stub.asinterface
这个接口主要是客户端通过Binder引用拿到IxxxxService实例：
```java
IxxxxService.Stub.asInterface(IBinder obj) ：
```


当bindService之后，客户端会得到一个Binder引用，不是IxxxxService.Proxy实例。Ok, 但如果服务端和客户端都是在同一个进程呢，还需要利用IPC吗？这样就不需要了，直接将IxxxxService当做普通的对象调用就成了。Google利用IxxxxService.Stub.asInterface函数对这两种不同的情况进行了统一，也就是不管你是在同一进程还是不同进 程，那么在拿到Binder引用后，调用IxxxxService.Stub.asInterface(IBinder obj) 即可得到一个IxxxxService 实例，然后你只管调用IxxxxService里的函数就成了。

```java
import com.android.internal.app.IBatteryStats;

IBatteryStats mBatteryStats = IBatteryStats.Stub.asInterface(
                    ServiceManager.getService("batterystats"));
mBatteryStats.noteBleScanResults(mWorkSource, 100);
```


---


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


---

# 参考文档

[Android 性能优化之内存泄漏检测以及内存优化（中）](https://blog.csdn.net/self_study/article/details/66969064)





---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p37-系统开发之binder)

---


[上一篇文章 P35_系统开发之cpu](https://github.com/hfreeman2008/android_core_framework/blob/main/P35_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bcpu/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bcpu.md)



---

# 结束语

<img src="../Images/end_001.png">