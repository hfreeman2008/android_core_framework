# P15_系统开发之浅谈PackageManagerService

<img src="../flower/flower_u_green_white_000.png">

---

# PackageManagerService 类的作用：

管理应用，包括应用安装，删除，更新，应用的位置，应用的权限管理，截图，dex等。

追踪所有的apks，有三个重要的锁：mLock，mInstallLock，mSnapshotLock

```java
* Keep track of all those APKs everywhere.
* Internally there are three important locks:mLock，mInstallLock，mSnapshotLock

如果有修改，请跑一下cts验证修改是否会影响cts：
* $ runtest -c android.content.pm.PackageManagerTests frameworks-core
* $ cts-tradefed run commandAndExit cts -m CtsAppSecurityHostTestCases
```

---

# 获取pkms的方式：

```java
方式1
PackageManager pm = mContext.getPackageManager();

方式2
PackageManager mPm;
mPm = (PackageManager)getSystemService("package");

方式3
IPackageManager mPm;
mPm = IPackageManager.Stub.asInterface(ServiceManager.getService("package"));

方式4
IPackageManager mPM;
mPM = AppGlobals.getPackageManager();

方式5
IPackageManager mPM = ActivityThread.getPackageManager();

方式6 (system server进程使用)
PackageManagerInternal pm = LocalServices.getService(PackageManagerInternal.class);
```

---

# PackageManagerService调用流程



图一 PackageManagerService调用流程

以getContentCaptureServicePackageName为例，查看PackageManagerService调用流程：

(1)app应用中调用getContentCaptureServicePackageName:

```java
getPackageManager().getContentCaptureServicePackageName();
```

(2)PackageManager.java定义getContentCaptureServicePackageName

```java
/**
 * @return the system defined content capture package name, or null if there's none.
 *
 * @hide
 */
@TestApi
@Nullable
public String getContentCaptureServicePackageName() {
    throw new UnsupportedOperationException(
            "getContentCaptureServicePackageName not implemented in subclass");
}
```

(3)PackageManager的子类ApplicationPackageManager实现getContentCaptureServicePackageName

```java
@Override
public String getContentCaptureServicePackageName() {
    try {
        return mPM.getContentCaptureServicePackageName();
    } catch (RemoteException e) {
        throw e.rethrowAsRuntimeException();
    }
}
```

(4)IPackageManager.aidl定义getContentCaptureServicePackageName

```java
String getContentCaptureServicePackageName();
```

(5)IPackageManagerBase实现getContentCaptureServicePackageName逻辑

```java
@Override
@Deprecated
public final String getContentCaptureServicePackageName() {
    return mService.ensureSystemPackageName(snapshot(),
            mService.getPackageFromComponentString(
                    R.string.config_defaultContentCaptureService));
}
```

(6)PackageManagerService调用ensureSystemPackageName逻辑

```java
@Nullable
String ensureSystemPackageName(@NonNull Computer snapshot,
        @Nullable String packageName) {
    if (packageName == null) {
        return null;
    }
    final long token = Binder.clearCallingIdentity();
    try {
        if (snapshot.getPackageInfo(packageName, MATCH_FACTORY_ONLY,
                UserHandle.USER_SYSTEM) == null) {
            PackageInfo packageInfo =
                    snapshot.getPackageInfo(packageName, 0, UserHandle.USER_SYSTEM);
            if (packageInfo != null) {
                EventLog.writeEvent(0x534e4554, "145981139", packageInfo.applicationInfo.uid,
                        "");
            }
            Log.w(TAG, "Missing required system package: " + packageName + (packageInfo != null
                    ? ", but found with extended search." : "."));
            return null;
        }
    } finally {
        Binder.restoreCallingIdentity(token);
    }
    return packageName;
}
```

(7)在SystemServer.startBootstrapServices启动PackageManagerService服务：

```java
t.traceBegin("StartPackageManagerService");
try {
    Watchdog.getInstance().pauseWatchingCurrentThread("packagemanagermain");
    mPackageManagerService = PackageManagerService.main(mSystemContext, installer,
            mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore);
} finally {
    Watchdog.getInstance().resumeWatchingCurrentThread("packagemanagermain");
}

// Now that the package manager has started, register the dex load reporter to capture any
// dex files loaded by system server.
// These dex files will be optimized by the BackgroundDexOptService.
SystemServerDexLoadReporter.configureSystemServerDexReporter(mPackageManagerService);

mPackageManager = mSystemContext.getPackageManager();
t.traceEnd();
```

---


# PackageManagerService类图

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


# 结束语

<img src="../Images/end_001.png">
