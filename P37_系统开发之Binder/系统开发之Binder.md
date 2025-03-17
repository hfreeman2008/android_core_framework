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

## IBinder 接口‌
核心作用‌：Binder 通信的基础接口，定义了跨进程通信的核心方法 transact() 和 queryLocalInterface()，所有 Binder 对象（本地或远程）均需实现该接口‌。
关键方法‌：

```java
transact(int code, Parcel data, Parcel reply, int flags)：客户端通过此方法向服务端发送请求。
queryLocalInterface(String descriptor)：尝试获取本地接口实现，避免跨进程调用。
```


## IInterface 接口‌
核心作用‌：定义 Binder 服务接口的基类，所有 AIDL 生成的接口（如 IMyService）均继承自 IInterface，用于统一本地和远程接口的调用规范‌。
关键方法‌：
```java
asBinder()：返回关联的 IBinder 对象。
```

## BpBinder 与 BBinder‌
- BpBinder‌（Proxy Binder）：
客户端代理类，封装远程 Binder 的引用，通过 transact() 向服务端发送请求‌。属于 IBinder 的实现类，由 Binder 驱动生成。
- BBinder‌（Base Binder）：
服务端本地 Binder 的基类，负责处理客户端请求，需实现 onTransact() 方法解析请求并执行逻辑‌。

## BpInterface 与 BnInterface‌
- BpInterface‌：
远程接口基类，客户端通过生成的代理类（如 BpMyService）调用服务端方法‌。
继承自 BpBinder，实现 AIDL 定义的接口。
- BnInterface‌：
本地接口基类，服务端需继承此类并实现具体的业务逻辑（如 BnMyService）‌。
继承自 BBinder，重写 onTransact() 处理客户端请求。


## AIDL 生成的 Stub 与 Proxy‌
- Stub 类‌：
继承 BnInterface，服务端需实现 Stub 中定义的抽象方法（如 add()）‌。

通过 onTransact() 解析客户端请求并分发给具体实现。

- Proxy 类‌：
继承 BpInterface，客户端通过 Proxy 对象调用远程服务，内部调用 BpBinder.transact() 发送数据‌。

## Parcel 类‌
核心作用‌：跨进程数据传输的容器，支持序列化与反序列化。客户端和服务端通过 Parcel 打包和解包数据‌。

典型使用场景‌：

客户端在 transact() 中将参数写入 Parcel。

服务端在 onTransact() 中从 Parcel 读取参数，处理后写入返回结果。

## ProcessState 与 IPCThreadState‌

ProcessState‌：

管理进程的 Binder 通信资源（如线程池），负责打开 Binder 驱动并初始化通信环境‌。

IPCThreadState‌：

封装线程级 Binder 通信逻辑，包括请求的发送与接收循环（如 transact() 和 executeCommand()）‌。
接口协作关系示意图

- 客户端调用流程：
BpInterface（Proxy） → BpBinder.transact() → IPCThreadState → Binder 驱动 → 服务端 BBinder.onTransact() → BnInterface（Stub） → 实际业务逻辑

- 服务端响应流程：
BBinder.onTransact() → 解析请求 → 调用 Stub 实现的方法 → 结果通过 Parcel 返回 → Binder 驱动 → 客户端接收结果

通过上述接口的协作，Binder 实现了跨进程的透明调用，客户端和服务端只需关注业务逻辑，底层通信由 Binder 框架自动处理‌。


---

# 客户端如何理解 Binder‌
在 Android 客户端开发中，‌Binder‌ 是跨进程通信（IPC）的核心机制，客户端通过 ‌Binder 代理对象（Proxy）‌ 与服务端（如系统服务或其他进程的组件）交互。


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