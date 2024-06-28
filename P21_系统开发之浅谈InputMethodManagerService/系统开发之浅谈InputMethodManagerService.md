# P21: 系统开发之浅谈InputMethodManagerService

<img src="../flower/flower_p20.png">

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


```java

```


```java

```


```java

```

```java

```

---

# 结束语

<img src="../Images/end_001.png">