# StatusBar

<img src="../flower/flower_p24.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)


---


[返回 P34: 系统开发之Display](https://github.com/hfreeman2008/android_core_framework/blob/main/P34_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay.md)


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---


# StatusBarManager

StatusBarManager 是一个系统服务类，用于管理 Android 系统的状态栏。它允许应用控制状态栏的显示和隐藏，以及设置状态栏图标等。是一个提供给 App 或系统应用调用的 客户端接口；



## 常用接口

| 方法                                                                         | 说明                         |
|----------------------------------------------------------------------------|----------------------------|
| setDisabledForSetup(boolean disabled)                                      | 设置 SetupWizard 模式下状态栏是否禁用  |
| setSystemUiVisibility(int vis, int mask)                                   | 设置 System UI 可见性           |
| expandNotificationsPanel()                                                 | 展开通知栏（下拉）                  |
| collapsePanels()                                                           | 收起通知栏                      |
| disable(int what)                                                          | 禁用某些状态栏图标或交互（需要系统权限）       |
| disable2(int what)                                                         | 禁用更复杂的交互（如 quick settings） |
| setIcon(String slot, int iconId, int iconLevel, String contentDescription) | 设置状态栏图标                    |
| removeIcon(String slot)                                                    | 移除图标                       |




## expandNotificationsPanel
功能：展开状态栏的通知面板。

```java
StatusBarManager statusBarManager = (StatusBarManager) getSystemService(Context.STATUS_BAR_SERVICE);
statusBarManager.expandNotificationsPanel();
```


## expandSettingsPanel()
功能：展开状态栏的设置面板。
```java
StatusBarManager statusBarManager = (StatusBarManager) getSystemService(Context.STATUS_BAR_SERVICE);
statusBarManager.expandSettingsPanel();
```


## collapsePanels
功能：收起所有展开的状态栏面板，包括通知面板和设置面板。
```java
StatusBarManager statusBar = (StatusBarManager) getSystemService(Context.STATUS_BAR_SERVICE);
statusBar.collapsePanels();
```


## setIcon
```java
setIcon(String slot, int iconId, int iconLevel, CharSequence contentDescription)
```


功能：设置状态栏上指定位置的图标。

参数说明：

- slot：图标的标识符，用于指定要设置的图标位置。
- iconId：图标的资源 ID。
- iconLevel：图标的级别，某些图标可能有不同的状态级别。
- contentDescription：图标的描述信息，用于辅助功能。

```java
StatusBarManager statusBarManager = (StatusBarManager) getSystemService(Context.STATUS_BAR_SERVICE);
statusBarManager.setIcon("custom_icon_slot", R.drawable.custom_icon, 0, "Custom Icon");
```


## removeIcon
removeIcon(String slot)

功能：移除状态栏上指定位置的图标。
```java
StatusBarManager statusBarManager = (StatusBarManager) getSystemService(Context.STATUS_BAR_SERVICE);
statusBarManager.removeIcon("custom_icon_slot");
```


## 状态栏禁用标志位（Flags）

StatusBarManager 和 StatusBarManagerService 中使用位掩码来控制状态栏的功能。

常见的标志位包括：
- DISABLE_EXPAND                    禁用状态栏的下拉展开功能。
- DISABLE_NOTIFICATION_ALERTS        禁用通知提醒。
- DISABLE_NOTIFICATION_ICONS    禁用状态栏中的通知图标。
- DISABLE_SYSTEM_INFO            禁用系统信息（如时间、电量）。
- DISABLE_HOME                    禁用 Home 键。
- DISABLE_RECENT                禁用最近任务键。
- DISABLE_BACK                    禁用返回键。
- DISABLE_NONE                    启用所有功能。



禁用通知提醒和禁用状态栏的下拉展开功能
```java
int flags = StatusBarManager.DISABLE_EXPAND | StatusBarManager.DISABLE_NOTIFICATION_ALERTS;
statusBarManager.disable(flags);
```

设置导航栏是否disable home,back,recent键
```java
StatusBarManager mStatusBar = (StatusBarManager) mContext.getSystemService(Context.STATUS_BAR_SERVICE);
final int disableNavigationBar = (View.STATUS_BAR_DISABLE_HOME
        | View.STATUS_BAR_DISABLE_BACK
        | View.STATUS_BAR_DISABLE_RECENT);
if (mStatusBar != null) {
    if (enable) {
        mStatusBar.disable(disableNavigationBar);
    } else {
        mStatusBar.disable(StatusBarManager.DISABLE_NONE);
    }
}
```

---


# StatusBarManagerService

框架层的 系统服务类，是 StatusBarManager 的真正实现者，通过 Binder 机制提供服务。它维护了所有对状态栏的底层调用，如：
- 接收客户端请求（expand/collapse）
- 管理系统图标、通知图标
- 控制 System UI 显示状态
- 通知状态栏窗口状态变化
- 管理 disable 状态
- 与 SystemUI 进程通信


## 常用接口


- registerStatusBar(StatusBar statusBar)
功能：注册一个状态栏实例，通常由系统的状态栏实现类调用，用于将自身注册到 StatusBarManagerService 中。

- unregisterStatusBar(StatusBar statusBar)
功能：取消注册一个状态栏实例。

- addNotification(StatusBarNotification notification)
功能：添加一个通知到状态栏。当系统接收到新的通知时，会调用该方法将通知信息传递给 StatusBarManagerService 进行处理。

- expandNotificationsPanel()：展开通知面板。
- collapsePanels()：收起通知面板。
- disable(int what, IBinder token, String pkg)：禁用状态栏的某些功能。
- disable2(int what, IBinder token, String pkg)：禁用状态栏的某些功能（更高级的版本）。
- setIcon(String slot, String iconPackage, int iconId, int iconLevel, String contentDescription)：设置状态栏图标。
- setIconVisibility(String slot, boolean visible)：设置状态栏图标的可见性。
- removeIcon(String slot)：移除状态栏图标。
- expandSettingsPanel(String subPanel)：展开设置面板。



---

# 状态栏相关的接口

```java
Window window = this.getWindow();      
window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_HIDE_NAVIGATION | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION);
window.setStatusBarColor(Color.BLACK);
```

---


# 与 SystemUI 的关系
- StatusBarManagerService与 SystemUI 进程通信，SystemUI 实现了 IStatusBar 接口。

```java
private volatile IStatusBar mBar;

对应：
frameworks\base\packages\SystemUI\src\com\android\systemui\statusbar\phone\StatusBar.java
```

- 启动过程中 SystemServer 会启动服务并注册 StatusBarManagerService。
- 然后 SystemUI 启动时会调用 registerStatusBar() 注册回调。




---




# 相关命令

## 展开通知面板
```java
adb shell service call statusbar 1
```


## 收起所有面板
```java
adb shell service call statusbar 2
```


## 收起所有面板
```java
adb shell service call statusbar 3
```

## 全局沉浸模式‌
```java
adb shell settings put global policy_control immersive.full=* 
```



## 强制显示通知栏‌
```java
adb shell am broadcast -a android.intent.action.SHOW_NOTIFICATIONS
```


## StatusBarManagerService日志
```java
adb logcat -s StatusBarManagerService
```

## 验证statusbar服务是否运行‌
```java
adb shell service list | grep statusbar
```


## dumpsys statusbar
```java
adb shell dumpsys statusbar
```




---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#statusbar)

---


[返回 P34: 系统开发之Display](https://github.com/hfreeman2008/android_core_framework/blob/main/P34_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay.md)


---

# 结束语

<img src="../Images/end_001.png">