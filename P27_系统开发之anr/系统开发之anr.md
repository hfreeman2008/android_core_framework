
# P27: 系统开发之anr

<img src="../flower/flower_p27.png">

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

# ANR

ANR(application not response)，应用无响应。

整个ANR机制是横跨几个层：
App层：应用主线程的处理逻辑；
Framework层：主要有AMS、BroadcastQueue、ActiveServices、InputmanagerService、InputMonitor、InputChannel、ProcessCpuTracker等；
Native层：InputDispatcher.cpp；


---


# 哪些场景会造成ANR：



---



```sh

```




```sh

```



```sh

```





```java


```


---



![系统属性架构设计](系统属性架构设计.png)



---



---

# 参考资料

1.Android ANR分析

https://blog.csdn.net/yxz329130952/article/details/50087731


2.理解Android ANR的触发原理

http://gityuan.com/2016/07/02/android-anr/


3.解读Java进程的Trace文件

http://gityuan.com/2016/11/26/art-trace/

4.Native进程之Trace原理

http://gityuan.com/2016/11/27/native-traces/


5.Android打印Trace堆栈

http://gityuan.com/2017/07/09/android_debug/



---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p27-系统开发之anr)

---

# 结束语

<img src="../Images/end_001.png">