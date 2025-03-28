# P38: 系统开发之permission PermissionManager

<img src="../flower/flower_p29.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

[返回 P38_系统开发之permission](https://github.com/hfreeman2008/android_core_framework/blob/main/P38_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bpermission/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bpermission.md)


---


# 







---

# 参考文档



---


# PermissionManager 与 AppOpsManager 核心对比

## 一、职责与设计目标差异‌
| ‌维度   | ‌PermissionManager‌‌ | ‌AppOpsManager‌‌     |
|--------|------|----------|
| 核心定位‌   | 管理应用声明的标准权限（如CAMERA、LOCATION），处理动态权限授予与状态跟踪   | 控制应用对敏感操作的实际执行权限（如后台定位、剪贴板读取），提供运行时拦截能力‌     |
| 权限模型‌   | 基于安装时声明（AndroidManifest）和运行时动态申请   | 基于动态策略控制，可覆盖传统权限设置（如授予权限但限制操作）‌     |
| 用户可见性‌   | 权限状态通过系统弹窗或设置界面直接暴露给用户   | 操作模式（如静默拒绝）通常由系统工具或企业策略配置，用户无感知     |





## 二、权限控制粒度对比‌
| ‌类别‌‌   | PermissionManager‌‌ | AppOpsManager‌‌     |
|--------|------|----------|
| 控制对象‌   | 标准权限组（如STORAGE、CONTACTS）   | 具体操作（OP_STR标识，如OP_READ_SMS、OP_CAMERA）‌     |
| 覆盖范围‌   | 仅管理声明在AndroidManifest中的权限   | 可控制未声明权限的敏感操作（如剪贴板读取、后台定位）‌     |
| 策略优先级‌   | 权限授予状态决定应用能否执行相关操作   | 即使权限被授予，仍可通过setMode()拦截操作     |




## 三、接口设计与使用场景‌

| ‌功能‌   | ‌PermissionManager‌‌ | AppOpsManager     |
|--------|------|----------|
| ‌‌核心接口‌   | checkPermission()：权限状态检查 requestPermissions()：动态申请权限   | checkOp():检测权限  noteOp()：记录单次操作 setMode()：动态调整操作模式 startOp()：标记长时操作     |
| 权限状态   | PackageManager.PERMISSION_GRANTED（已授予） PackageManager.PERMISSION_DENIED（未授予）   | AppOpsManager.MODE_ALLOWED：允许应用执行该操作。AppOpsManager.MODE_IGNORED：忽略应用对该操作的请求。AppOpsManager.MODE_ERRORED：发生错误。AppOpsManager.MODE_DEFAULT：使用默认的权限设置。     |
| ‌典型使用场景   | ‌应用内动态权限申请，权限合规性检查   | ‌后台敏感操作限制（如禁止应用读取剪贴板） ，企业设备管控（禁用非必要权限）‌|
| 权限依赖   | ‌普通应用可直接调用接口   | 需系统级应用签名或android:sharedUserId="android.uid.system"     |
| 用户交互   | 需要用户授予权限   | 无需用户交互，操作权限由系统或应用控制     |




## ‌四、数据存储与兼容性‌
| ‌维度‌‌   | PermissionManager‌‌ | AppOpsManager‌‌     |
|--------|------|----------|
| 配置存储‌   | 权限状态存储在/data/system/packages.xml   | 操作模式记录在/data/system/appops.xml‌     |
| 版本兼容性‌   | Android 10+ 正式引入自    | Android 4.3 引入，部分操作码需高版本适配（如 Android 13+ 新增OP_READ_MEDIA_IMAGES）‌     |
| 审计能力‌   | 提供权限授予历史记录   | 记录操作调用次数、时间等详细日志（如摄像头启动次数）‌     |




## 总结：核心区别与适用场景‌
PermissionManager‌
- ‌重点‌：管理应用声明的标准权限，遵循 Android 官方权限模型，提供用户可见的交互流程 
- ‌适用场景‌：常规动态权限申请、权限状态检查、权限组管理 


AppOpsManager‌
- ‌重点‌：控制应用实际操作的执行权限，支持细粒度策略覆盖，适用于系统级或企业级深度管控 
- ‌适用场景‌：后台敏感操作拦截、无声明权限的隐私保护、操作日志审计 ‌

## ‌选择建议‌
常规开发‌：优先使用 PermissionManager 处理动态权限流程，确保符合 Google Play 政策 

系统工具或企业应用‌：结合 AppOpsManager 实现更严格的权限策略（如禁止后台录音），需注意系统权限要求








---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p38-系统开发之permission-PermissionManager)

---


[返回 P38_系统开发之permission](https://github.com/hfreeman2008/android_core_framework/blob/main/P38_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bpermission/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8Bpermission.md)



---

# 结束语

<img src="../Images/end_001.png">