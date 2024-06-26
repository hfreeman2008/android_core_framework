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

# 通过音量键是如何调整音量

在按下音量键的时候，会先经过PhoneWindowManager的处理是否拦截：
```java
public long interceptKeyBeforeDispatching(IBinder focusedToken, KeyEvent event,
        int policyFlags) {
            ......
else if (keyCode == KeyEvent.KEYCODE_VOLUME_UP
        || keyCode == KeyEvent.KEYCODE_VOLUME_DOWN
        || keyCode == KeyEvent.KEYCODE_VOLUME_MUTE) {
    if (mUseTvRouting || mHandleVolumeKeysInWM) {
        // On TVs or when the configuration is enabled, volume keys never
        // go to the foreground app.
        dispatchDirectAudioEvent(event);
        return -1;
    }

}
private void dispatchDirectAudioEvent(KeyEvent event) {
    try {
        getAudioService().handleVolumeKey(event, mUseTvRouting,
                mContext.getOpPackageName(), TAG);
    } catch (Exception e) {
        Log.e(TAG, "Error dispatching volume key in handleVolumeKey for event:"
                + event, e);
    }
}

```

frameworks\base\services\core\java\com\android\server\audio\AudioService.java

```java
public void handleVolumeKey(@NonNull KeyEvent event, boolean isOnTv,
        @NonNull String callingPackage, @NonNull String caller) {
   .......
    switch (event.getKeyCode()) {
        case KeyEvent.KEYCODE_VOLUME_UP:
                adjustSuggestedStreamVolume(AudioManager.ADJUST_RAISE,
                        AudioManager.USE_DEFAULT_STREAM_TYPE, flags, callingPackage, caller,
                        Binder.getCallingUid(), true, keyEventMode);
            break;
        case KeyEvent.KEYCODE_VOLUME_DOWN:
                adjustSuggestedStreamVolume(AudioManager.ADJUST_LOWER,
                        AudioManager.USE_DEFAULT_STREAM_TYPE, flags, callingPackage, caller,
                        Binder.getCallingUid(), true, keyEventMode);
            break;
        case KeyEvent.KEYCODE_VOLUME_MUTE:
            if (event.getAction() == KeyEvent.ACTION_DOWN && event.getRepeatCount() == 0) {
                adjustSuggestedStreamVolume(AudioManager.ADJUST_TOGGLE_MUTE,
                        AudioManager.USE_DEFAULT_STREAM_TYPE, flags, callingPackage, caller,
                        Binder.getCallingUid(), true, VOL_ADJUST_NORMAL);
            }
            break;
        default:
            Log.e(TAG, "Invalid key code " + event.getKeyCode() + " sent by " + callingPackage);
            return; // not needed but added if code gets added below this switch statement
    }
}

/** @see AudioManager#adjustVolume(int, int) */
public void adjustSuggestedStreamVolume(int direction, int suggestedStreamType, int flags,
        String callingPackage, String caller) {
    boolean hasModifyAudioSettings =
            mContext.checkCallingPermission(Manifest.permission.MODIFY_AUDIO_SETTINGS)
            == PackageManager.PERMISSION_GRANTED;
    adjustSuggestedStreamVolume(direction, suggestedStreamType, flags, callingPackage,
            caller, Binder.getCallingUid(), hasModifyAudioSettings, VOL_ADJUST_NORMAL);
}

private void adjustSuggestedStreamVolume(int direction, int suggestedStreamType, int flags,
        String callingPackage, String caller, int uid, boolean hasModifyAudioSettings,
        int keyEventMode) {
......
    adjustStreamVolume(streamType, direction, flags, callingPackage, caller, uid,
            hasModifyAudioSettings, keyEventMode);
}

```


# 启动AudioService 服务：

SystemServer.java

```java
t.traceBegin("StartAudioService");
if (!isArc) {
    mSystemServiceManager.startService(AudioService.Lifecycle.class);
} else {
    String className = context.getResources()
            .getString(R.string.config_deviceSpecificAudioService);
    try {
        mSystemServiceManager.startService(className + "$Lifecycle");
    } catch (Throwable e) {
        reportWtf("starting " + className, e);
    }
}
t.traceEnd();
```

---

# 注册Audio

SystemServiceRegistry.java

```java
registerService(Context.AUDIO_SERVICE, AudioManager.class,
        new CachedServiceFetcher<AudioManager>() {
    @Override
    public AudioManager createService(ContextImpl ctx) {
        return new AudioManager(ctx);
    }});
```

---

# AudioService 类图

<img src="AudioService_class.png">

图三 AudioService类图

---

# handler消息

有一个handler:


```java
private class AudioHandler extends Handler
```

消息列表：

```java
// AudioHandler messages
private static final int MSG_SET_DEVICE_VOLUME = 0;
private static final int MSG_PERSIST_VOLUME = 1;
private static final int MSG_PERSIST_VOLUME_GROUP = 2;
private static final int MSG_PERSIST_RINGER_MODE = 3;
private static final int MSG_AUDIO_SERVER_DIED = 4;
private static final int MSG_PLAY_SOUND_EFFECT = 5;
private static final int MSG_LOAD_SOUND_EFFECTS = 7;
private static final int MSG_SET_FORCE_USE = 8;
private static final int MSG_BT_HEADSET_CNCT_FAILED = 9;
private static final int MSG_SET_ALL_VOLUMES = 10;
private static final int MSG_CHECK_MUSIC_ACTIVE = 11;
private static final int MSG_CONFIGURE_SAFE_MEDIA_VOLUME = 12;
private static final int MSG_CONFIGURE_SAFE_MEDIA_VOLUME_FORCED = 13;
private static final int MSG_PERSIST_SAFE_VOLUME_STATE = 14;
private static final int MSG_UNLOAD_SOUND_EFFECTS = 15;
private static final int MSG_SYSTEM_READY = 16;
private static final int MSG_PERSIST_MUSIC_ACTIVE_MS = 17;
private static final int MSG_UNMUTE_STREAM = 18;
private static final int MSG_DYN_POLICY_MIX_STATE_UPDATE = 19;
private static final int MSG_INDICATE_SYSTEM_READY = 20;
private static final int MSG_ACCESSORY_PLUG_MEDIA_UNMUTE = 21;
private static final int MSG_NOTIFY_VOL_EVENT = 22;
private static final int MSG_DISPATCH_AUDIO_SERVER_STATE = 23;
private static final int MSG_ENABLE_SURROUND_FORMATS = 24;
private static final int MSG_UPDATE_RINGER_MODE = 25;
private static final int MSG_SET_DEVICE_STREAM_VOLUME = 26;
private static final int MSG_OBSERVE_DEVICES_FOR_ALL_STREAMS = 27;
private static final int MSG_HDMI_VOLUME_CHECK = 28;
private static final int MSG_PLAYBACK_CONFIG_CHANGE = 29;
private static final int MSG_BROADCAST_MICROPHONE_MUTE = 30;
private static final int MSG_CHECK_MODE_FOR_UID = 31;
private static final int MSG_REINIT_VOLUMES = 32;
```

---

# dump信息

命令：
```java
adb shell dumpsys audio
```


```java
Audio event log: audio services lifecycle
06-04 20:03:29:785 AudioService()

Message handler (watch for unhandled messages):
  Handler (com.android.server.audio.AudioService$AudioHandler) {6dfb37a} @ 51717218
    Looper (AudioService, tid 104) {4ac72b}
      (Total messages: 0, polling=true, quitting=false)

MediaFocusControl dump time: 上午10:25:13

Audio Focus stack entries (last is top of stack):

No external focus policy

 Notify on duck:  true

 In ring or call: false

Audio event log: focus commands as seen by MediaFocusControl
Multi Audio Focus enabled :false

Stream volumes (device: index)
- STREAM_VOICE_CALL:
   Muted: false
   Muted Internally: false
   Min: 1
   Max: 5
   streamVolume:3
   Current: 40000000 (default): 3
   Devices: earpiece
- STREAM_SYSTEM:
   Muted: false
   Muted Internally: false
   Min: 0
   Max: 7
   streamVolume:5
   Current: 40000000 (default): 5
   Devices: speaker
```

---

# 日志开关

```java
/** Debug audio mode */
protected static final boolean DEBUG_MODE = false;

/** Debug audio policy feature */
protected static final boolean DEBUG_AP = false;

/** Debug volumes */
protected static final boolean DEBUG_VOL = true;//false;

/** debug calls to devices APIs */
protected static final boolean DEBUG_DEVICES = false;

/** debug SCO modes */
protected static final boolean DEBUG_SCO = true;
```

---

# Lifecycle--publishBinderService


```java
public static final class Lifecycle extends SystemService {
    private AudioService mService;

    public Lifecycle(Context context) {
        super(context);
        mService = new AudioService(context);
    }

    @Override
    public void onStart() {
        publishBinderService(Context.AUDIO_SERVICE, mService);
    }

    @Override
    public void onBootPhase(int phase) {
        if (phase == SystemService.PHASE_ACTIVITY_MANAGER_READY) {
            mService.systemReady();
        }
    }
}
```

其他进程读取服务的方法：

```java
AudioManager manager = (AudioManager) mContext.getSystemService(Context.AUDIO_SERVICE);
```

---

# LocalServices--AudioServiceInternal

```java
LocalServices.addService(AudioManagerInternal.class, new AudioServiceInternal());
```

system server进程使用:
```java
AudioManagerInternal audioManager =LocalServices.getService(AudioManagerInternal.class);
```

---

# 各种类型音量的最大值和最小值
```java
   /** Maximum volume index values for audio streams */
    private static int[] MAX_STREAM_VOLUME = new int[] {
        5,  // STREAM_VOICE_CALL
        7,  // STREAM_SYSTEM
        7,  // STREAM_RING
        15, // STREAM_MUSIC
        7,  // STREAM_ALARM
        7,  // STREAM_NOTIFICATION
        15, // STREAM_BLUETOOTH_SCO
        7,  // STREAM_SYSTEM_ENFORCED
        15, // STREAM_DTMF
        15, // STREAM_TTS
        15  // STREAM_ACCESSIBILITY
    };

    /** Minimum volume index values for audio streams */
    private static int[] MIN_STREAM_VOLUME = new int[] {
        1,  // STREAM_VOICE_CALL
        0,  // STREAM_SYSTEM
        0,  // STREAM_RING
        0,  // STREAM_MUSIC
        0,  // STREAM_ALARM
        0,  // STREAM_NOTIFICATION
        0,  // STREAM_BLUETOOTH_SCO
        0,  // STREAM_SYSTEM_ENFORCED
        0,  // STREAM_DTMF
        0,  // STREAM_TTS
        0   // STREAM_ACCESSIBILITY
    };
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

---


# 结束语

<img src="../Images/end_001.png">