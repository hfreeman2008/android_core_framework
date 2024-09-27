# P23: 系统开发之如何新建一个system_bin

<img src="../flower/flower_p23.png">

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#参考资料)

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






---





---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p23-系统开发之如何新建一个system_bin)

---

# 结束语

<img src="../Images/end_001.png">
