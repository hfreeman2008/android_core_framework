# P21: 系统开发之浅谈InputMethodManagerService

<img src="../flower/flower_p21.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

# InputMethodManagerService 作用

输入法的选择,切换,管理,输入法界面的管理,软键盘的显示与管理,输入法全屏显示,输入与位置的关系。

---

# 获取 InputMethodManagerService：

```java
方式1
InputMethodManager mImm = (InputMethodManager)getSystemService(INPUT_METHOD_SERVICE);

方式2
InputMethodManager mImm = getSystemService(InputMethodManager.class);

方式3
IInputMethodManager service = IInputMethodManager.Stub.asInterface(ServiceManager.getServiceOrThrow(Context.INPUT_METHOD_SERVICE));

方式4 (system server进程使用)
InputMethodManagerInternal mInputMethodManagerInternal = LocalServices.getService(InputMethodManagerInternal.class);
```

---

# InputMethodManagerService 调用流程

<img src="InputMethodManagerService_whole.png">

以接口getInputMethodList为例：

(1)app调用getInputMethodList

```java
InputMethodManager mImm = getSystemService(InputMethodManager.class);
List<InputMethodInfo>  mInputMethodInfoList = mImm.getInputMethodList();
```

(2)InputMethodManager调用getInputMethodList

InputMethodManager.java
```java
/**
 * Returns the list of installed input methods.
 *
 * <p>On multi user environment, this API returns a result for the calling process user.</p>
 *
 * @return {@link List} of {@link InputMethodInfo}.
 */
public List<InputMethodInfo> getInputMethodList() {
    try {
        // We intentionally do not use UserHandle.getCallingUserId() here because for system
        // services InputMethodManagerInternal.getInputMethodListAsUser() should be used
        // instead.
        return mService.getInputMethodList(UserHandle.myUserId());
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
}
```

(3)aidl定义getInputMethodList

IInputMethodManager.aidl
```java
List<InputMethodInfo> getInputMethodList(int userId);
```

(4)InputMethodManagerService--getInputMethodList

InputMethodManagerService.java
```java
public List<InputMethodInfo> getInputMethodList(@UserIdInt int userId) {
    synchronized (mMethodMap) {
        final int[] resolvedUserIds = InputMethodUtils.resolveUserId(userId,
                mSettings.getCurrentUserId(), null);
        if (resolvedUserIds.length != 1) {
            return Collections.emptyList();
        }
        final long ident = Binder.clearCallingIdentity();
        try {
            return getInputMethodListLocked(resolvedUserIds[0]);
        } finally {
            Binder.restoreCallingIdentity(ident);
        }
    }
}
```

(5)注册 InputMethodManagerService

SystemServiceRegistry.java
```java
registerService(Context.INPUT_METHOD_SERVICE, InputMethodManager.class,
        new ServiceFetcher<InputMethodManager>() {
    @Override
    public InputMethodManager getService(ContextImpl ctx) {
        return InputMethodManager.forContext(ctx.getOuterContext());
    }});
```

(6)启动InputMethodManagerService：

SystemServer.startOtherServices
```java
t.traceBegin("StartInputMethodManagerLifecycle");
if (InputMethodSystemProperty.MULTI_CLIENT_IME_ENABLED) {
    mSystemServiceManager.startService(
            MultiClientInputMethodManagerService.Lifecycle.class);
} else {
    mSystemServiceManager.startService(InputMethodManagerService.Lifecycle.class);
}
t.traceEnd();
```


---

# InputMethodManagerService类图

<img src="InputMethodManagerService_class.png">


---

# 日志开关

```java
frameworks\base\services\core\java\com\android\server\inputmethod\InputMethodManagerService.java
static final boolean DEBUG = true;//false;

frameworks\base\services\core\java\com\android\server\wm\WindowManagerDebugConfig.java
static final boolean DEBUG_INPUT_METHOD = true;//false;

frameworks\base\core\java\android\view\inputmethod\InputMethodManager.java
static final boolean DEBUG = false;

frameworks\base\core\java\android\inputmethodservice\InputMethodService.java
static final boolean DEBUG = false;

frameworks\base\services\core\java\com\android\server\inputmethod\InputMethodUtils.java
public static final boolean DEBUG = false;

frameworks\base\services\core\java\com\android\server\inputmethod\InputMethodSubtypeSwitchingController.java
private static final boolean DEBUG = false;

frameworks\base\core\java\android\inputmethodservice\SoftInputWindow.java
private static final boolean DEBUG = true;//false;
```


---

# dump信息

```java
adb shell dumpsys input_method
```


---

# handler和消息

```java
static final int MSG_SHOW_IM_SUBTYPE_PICKER = 1;
static final int MSG_SHOW_IM_SUBTYPE_ENABLER = 2;
static final int MSG_SHOW_IM_CONFIG = 3;
static final int MSG_UNBIND_INPUT = 1000;
static final int MSG_BIND_INPUT = 1010;
static final int MSG_SHOW_SOFT_INPUT = 1020;
static final int MSG_HIDE_SOFT_INPUT = 1030;
static final int MSG_HIDE_CURRENT_INPUT_METHOD = 1035;
static final int MSG_INITIALIZE_IME = 1040;
static final int MSG_CREATE_SESSION = 1050;
static final int MSG_REMOVE_IME_SURFACE = 1060;
static final int MSG_REMOVE_IME_SURFACE_FROM_WINDOW = 1061;
static final int MSG_START_INPUT = 2000;
static final int MSG_UNBIND_CLIENT = 3000;
static final int MSG_BIND_CLIENT = 3010;
static final int MSG_SET_ACTIVE = 3020;
static final int MSG_SET_INTERACTIVE = 3030;
static final int MSG_REPORT_FULLSCREEN_MODE = 3045;
static final int MSG_REPORT_PRE_RENDERED = 3060;
static final int MSG_APPLY_IME_VISIBILITY = 3070;
static final int MSG_HARD_KEYBOARD_SWITCH_CHANGED = 4000;
static final int MSG_SYSTEM_UNLOCK_USER = 5000;
static final int MSG_DISPATCH_ON_INPUT_METHOD_LIST_UPDATED = 5010;
static final int MSG_INLINE_SUGGESTIONS_REQUEST = 6000;
static final int MSG_NOTIFY_IME_UID_TO_AUDIO_SERVICE = 7000;
```


---

# publishBinderService

onStart()方法中：
```java
publishBinderService(Context.INPUT_METHOD_SERVICE, mService);
```

其他进程获取 InputMethodManagerService :
```java
InputMethodManager mImm = (InputMethodManager)getSystemService(INPUT_METHOD_SERVICE);
```

---

# LocalServices--InputMethodManagerInternal


```java
LocalServices.addService(InputMethodManagerInternal.class,
        new LocalServiceImpl(mService));
```


在system server进程中：
```java
InputMethodManagerInternal mInputMethodManagerInternal = LocalServices.getService(InputMethodManagerInternal.class);
```

---

# ime命令

```java
adb shell ime list
adb shell ime list -a
adb shell ime list -s
```

```java
adb shell ime
ime <command>:
  list [-a] [-s]
    prints all enabled input methods.
      -a: see all input methods
      -s: only a single summary line of each
  enable [--user <USER_ID>] <ID>
    allows the given input method ID to be used.
      --user <USER_ID>: Specify which user to enable. Assumes the current user if not specified.
  disable [--user <USER_ID>] <ID>
    disallows the given input method ID to be used.
      --user <USER_ID>: Specify which user to disable. Assumes the current user if not specified.
  set [--user <USER_ID>] <ID>
    switches to the given input method ID.
      --user <USER_ID>: Specify which user to enable. Assumes the current user if not specified.
  reset [--user <USER_ID>]
    reset currently selected/enabled IMEs to the default ones as if the device is initially booted w
    ith the current locale.
      --user <USER_ID>: Specify which user to reset. Assumes the current user if not specified.
```

---


# 相关Settings字段


```java
Settings.Secure.DEFAULT_INPUT_METHOD
Settings.Secure.ENABLED_INPUT_METHODS
Settings.Secure.SELECTED_INPUT_METHOD_SUBTYPE
Settings.Secure.SHOW_IME_WITH_HARD_KEYBOARD
Settings.Secure.ACCESSIBILITY_SOFT_KEYBOARD_MODE
```

```shell
adb shell settings get secure default_input_method
adb shell settings get secure enabled_input_methods
adb shell settings get secure selected_input_method_subtype
adb shell settings get secure show_ime_with_hard_keyboard
adb shell settings get secure accessibility_soft_keyboard_mode

adb shell settings get secure default_input_method
com.sohu.inputmethod.sogou.oem/.SogouIME

adb shell settings get secure enabled_input_methods
com.android.inputmethod.latin/.LatinIME:com.sohu.inputmethod.sogou.oem/.SogouIME

adb shell settings get secure selected_input_method_subtype
adb shell settings get secure show_ime_with_hard_keyboard
adb shell settings get secure accessibility_soft_keyboard_mode
```

---


# showInputMethodMenu---------显示输入法菜单

```java
mDialogBuilder = new AlertDialog.Builder(settingsContext);
mDialogBuilder.setOnCancelListener(new OnCancelListener() {
    @Override
    public void onCancel(DialogInterface dialog) {
        hideInputMethodMenu();
    }
});

final Context dialogContext = mDialogBuilder.getContext();
final TypedArray a = dialogContext.obtainStyledAttributes(null,
        com.android.internal.R.styleable.DialogPreference,
        com.android.internal.R.attr.alertDialogStyle, 0);
final Drawable dialogIcon = a.getDrawable(
        com.android.internal.R.styleable.DialogPreference_dialogIcon);
a.recycle();

mDialogBuilder.setIcon(dialogIcon);

final LayoutInflater inflater = dialogContext.getSystemService(LayoutInflater.class);
final View tv = inflater.inflate(
        com.android.internal.R.layout.input_method_switch_dialog_title, null);
mDialogBuilder.setCustomTitle(tv);

// Setup layout for a toggle switch of the hardware keyboard
mSwitchingDialogTitleView = tv;
mSwitchingDialogTitleView
        .findViewById(com.android.internal.R.id.hard_keyboard_section)
        .setVisibility(mWindowManagerInternal.isHardKeyboardAvailable()
                ? View.VISIBLE : View.GONE);
final Switch hardKeySwitch = (Switch) mSwitchingDialogTitleView.findViewById(
        com.android.internal.R.id.hard_keyboard_switch);
hardKeySwitch.setChecked(mShowImeWithHardKeyboard);
hardKeySwitch.setOnCheckedChangeListener(new OnCheckedChangeListener() {
    @Override
    public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
        mSettings.setShowImeWithHardKeyboard(isChecked);
        // Ensure that the input method dialog is dismissed when changing
        // the hardware keyboard state.
        hideInputMethodMenu();
    }
});

final ImeSubtypeListAdapter adapter = new ImeSubtypeListAdapter(dialogContext,
        com.android.internal.R.layout.input_method_switch_item, imList, checkedItem);
final OnClickListener choiceListener = new OnClickListener() {
    @Override
    public void onClick(final DialogInterface dialog, final int which) {
        synchronized (mMethodMap) {
            if (mIms == null || mIms.length <= which || mSubtypeIds == null
                    || mSubtypeIds.length <= which) {
                return;
            }
            final InputMethodInfo im = mIms[which];
            int subtypeId = mSubtypeIds[which];
            adapter.mCheckedItem = which;
            adapter.notifyDataSetChanged();
            hideInputMethodMenu();
            if (im != null) {
                if (subtypeId < 0 || subtypeId >= im.getSubtypeCount()) {
                    subtypeId = NOT_A_SUBTYPE_ID;
                }
                setInputMethodLocked(im.getId(), subtypeId);
            }
        }
    }
};
mDialogBuilder.setSingleChoiceItems(adapter, checkedItem, choiceListener);

mSwitchingDialog = mDialogBuilder.create();
mSwitchingDialog.setCanceledOnTouchOutside(true);
final Window w = mSwitchingDialog.getWindow();
final WindowManager.LayoutParams attrs = w.getAttributes();
w.setType(TYPE_INPUT_METHOD_DIALOG);
// Use an alternate token for the dialog for that window manager can group the token
// with other IME windows based on type vs. grouping based on whichever token happens
// to get selected by the system later on.
attrs.token = mSwitchingDialogToken;
attrs.privateFlags |= PRIVATE_FLAG_SHOW_FOR_ALL_USERS;
attrs.setTitle("Select input method");
w.setAttributes(attrs);
updateSystemUi(mCurToken, mImeWindowVis, mBackDisposition);
mSwitchingDialog.show();
```


---

# input_method布局文件

frameworks\base\core\java\android\inputmethodservice\InputMethodService.java

```java
mThemeAttrs = obtainStyledAttributes(android.R.styleable.InputMethodService);
mRootView = mInflater.inflate(com.android.internal.R.layout.input_method, null);
```

资源文件：

```java
frameworks\base\core\res\res\layout\input_method.xml
frameworks\base\core\res\res\layout\input_method_switch_dialog_title.xml
frameworks\base\core\res\res\layout\input_method_switch_item.xml
frameworks\base\core\res\res\anim\input_method_enter.xml
frameworks\base\core\res\res\anim\input_method_exit.xml
frameworks\base\core\res\res\anim\input_method_extract_enter.xml
frameworks\base\core\res\res\anim\input_method_extract_exit.xml
frameworks\base\core\res\res\anim\input_method_fancy_enter.xml
frameworks\base\core\res\res\anim\input_method_fancy_exit.xml
```


---

# android开发浅谈之 InputMethodManagerService

1.[android开发浅谈之InputMethodManagerService](https://blog.csdn.net/hfreeman2008/article/details/117963600)

https://blog.csdn.net/hfreeman2008/article/details/117963600



2.[InputMethodManager](https://developer.android.google.cn/reference/android/view/inputmethod/InputMethodManager)

https://developer.android.google.cn/reference/android/view/inputmethod/InputMethodManager


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p21-系统开发之浅谈inputmethodmanagerservice)


---

[上一篇文章 P20_系统开发之浅谈NotificationManagerService](https://github.com/hfreeman2008/android_core_framework/blob/main/P20_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B5%85%E8%B0%88NotificationManagerService/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B5%85%E8%B0%88NotificationManagerService.md)



[下一篇文章 P22_系统开发之浅谈LightsService](https://github.com/hfreeman2008/android_core_framework/blob/main/P22_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B5%85%E8%B0%88LightsService/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8B%E6%B5%85%E8%B0%88LightsService.md)

---


# 结束语

<img src="../Images/end_001.png">

