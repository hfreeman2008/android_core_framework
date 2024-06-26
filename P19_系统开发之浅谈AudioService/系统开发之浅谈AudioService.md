# P19_系统开发之浅谈AudioService

<img src="../flower/flower_p19.png">

---

# AudioService 类的作用

AudioService 主要用于管理 负责管理应用程序和系统的音频资源。在操作系统中分配、控制和处理音频资源，以提供高质量、可靠和灵活的音频服务。

还负责管理系统音频路由、音频格式转换、音量控制、音频设备的连接和断开、通知应用程序和服务启动/停止的音频事件等。

---

# 获取 AudioService 的方式：

```java
方式1
AudioManager manager = (AudioManager) mContext.getSystemService(Context.AUDIO_SERVICE);

方式2
private static IAudioService sService;
IBinder b = ServiceManager.getService(Context.AUDIO_SERVICE);
sService = IAudioService.Stub.asInterface(b);

方式3 (system server进程使用)
AudioManagerInternal audioManager =LocalServices.getService(AudioManagerInternal.class);
```

---

# AudioService 调用流程

<img src="AudioService_whole.png">

图一 AudioService调用流程

以接口adjustSuggestedStreamVolume为例：

(1)app:
frameworks\base\packages\SystemUI\src\com\android\keyguard\KeyguardHostView.java

```java
if (mAudioManager == null) {
    mAudioManager = (AudioManager) getContext().getSystemService(
            Context.AUDIO_SERVICE);
}

// Volume buttons should only function for music (local or remote).
// TODO: Actually handle MUTE.
mAudioManager.adjustSuggestedStreamVolume(
        keyCode == KeyEvent.KEYCODE_VOLUME_UP
                ? AudioManager.ADJUST_RAISE
                : AudioManager.ADJUST_LOWER /* direction */,
        AudioManager.STREAM_MUSIC /* stream */, 0 /* flags */);
```


(2)frameworks\base\media\java\android\media\AudioManager.java

```java
public void adjustSuggestedStreamVolume(int direction, int suggestedStreamType, int flags) {
    MediaSessionLegacyHelper helper = MediaSessionLegacyHelper.getHelper(getContext());
    helper.sendAdjustVolumeBy(suggestedStreamType, direction, flags);
}
```

(3)frameworks\base\media\java\android\media\session\MediaSessionLegacyHelper.java

```java
public void sendAdjustVolumeBy(int suggestedStream, int delta, int flags) {
    mSessionManager.dispatchAdjustVolume(suggestedStream, delta, flags);
    if (DEBUG) {
        Log.d(TAG, "dispatched volume adjustment");
    }
}
```

(4)frameworks\base\media\java\android\media\session\MediaSessionManager.java

```java
private final ISessionManager mService;

public void dispatchAdjustVolume(int suggestedStream, int direction, int flags) {
    try {
        mService.dispatchAdjustVolume(mContext.getPackageName(), mContext.getOpPackageName(),
                suggestedStream, direction, flags);
    } catch (RemoteException e) {
        Log.e(TAG, "Failed to send adjust volume.", e);
    }
}

```

(5)frameworks\base\media\java\android\media\session\ISessionManager.aidl

```java
void dispatchAdjustVolume(String packageName, String opPackageName, int suggestedStream,
        int delta, int flags);
```

(6)frameworks\base\services\core\java\com\android\server\media\MediaSessionService.java

```java
public void dispatchAdjustVolume(String packageName, String opPackageName,
        int suggestedStream, int delta, int flags) {
    final int pid = Binder.getCallingPid();
    final int uid = Binder.getCallingUid();
    final long token = Binder.clearCallingIdentity();
    try {
        synchronized (mLock) {
            dispatchAdjustVolumeLocked(packageName, opPackageName, pid, uid, false,
                    suggestedStream, delta, flags);
        }
    } finally {
        Binder.restoreCallingIdentity(token);
    }
}

```

(7)frameworks\base\services\core\java\com\android\server\media\MediaSessionService.java

```java
private void dispatchAdjustVolumeLocked(String packageName, String opPackageName, int pid,
        int uid, boolean asSystemService, int suggestedStream, int direction, int flags) {
......
if (session == null || preferSuggestedStream) {
......
    try {
        mAudioManagerInternal.adjustSuggestedStreamVolumeForUid(suggestedStream,
                direction, flags, callingOpPackageName, callingUid, callingPid);
    }
} else {
    ......
    session.adjustVolume(packageName, opPackageName, pid, uid, asSystemService,
            direction, flags, true);
}

```

(8)frameworks\base\services\core\java\com\android\server\audio\AudioService.java

```java
public void adjustSuggestedStreamVolumeForUid(int streamType, int direction, int flags,
        String callingPackage, int uid, int pid) {
    // direction and stream type swap here because the public
    // adjustSuggested has a different order than the other methods.
    adjustSuggestedStreamVolume(direction, streamType, flags, callingPackage,
            callingPackage, uid, hasModifyAudioSettings, VOL_ADJUST_NORMAL);
}

private void adjustSuggestedStreamVolume(int direction, int suggestedStreamType, int flags,
        String callingPackage, String caller, int uid, boolean hasModifyAudioSettings,
        int keyEventMode) {
         ......

}

```

(9)aidl:
frameworks\base\media\java\android\media\IAudioService.aidl

```java
oneway void adjustSuggestedStreamVolume(int direction, int suggestedStreamType, int flags,
        String callingPackage, String caller);

```

---



```java


```


```java


```

```java


```

---


# 结束语

<img src="../Images/end_001.png">