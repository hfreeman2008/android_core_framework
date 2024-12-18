# Sensor笔记

<img src="../flower/flower_p28.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

# sensor 相关代码

```java
frameworks:
frameworks/native/services/sensorservice/SensorService.cpp
frameworks\base\core\java\android\hardware\SystemSensorManager.java  

jni:
frameworks\base\core\jni\android_hardware_SensorManager.cpp 

hardware:
vendor\mediatek\proprietary\hardware\sensor

kernel:
vendor/qcom/proprietary/sensors-see
kernel/msm-4.19/drivers/sensors
kernel/drivers/misc/mediatek/sensor_bio
```

---

# Add ltr559 and lsm6ds3 json

vendor/qcom/proprietary/sensors-see/ssc/registry/config/config_list.txt
```java
lsm6dso_0.json,
+ltr559_0.json,
sdm710_ak991x_0.json,
sdm710_qrd_ak991x_0.json,
sdm710_bmp285_0.json,
sdm710_cm3526_0.json,
+sdm710_lsm6ds3_0.json,
sdm710_lsm6dso_0.json,
+sdm710_ltr559_0.json,
```
再添加三个文件:
```java
vendor/qcom/proprietary/sensors-see/ssc/registry/config/ltr559_0.json
vendor/qcom/proprietary/sensors-see/ssc/registry/config/sdm710_lsm6ds3_0.json
vendor/qcom/proprietary/sensors-see/ssc/registry/config/sdm710_ltr559_0.json
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

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#sensor笔记)

---

# 结束语

<img src="../Images/end_001.png">