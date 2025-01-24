# View控件soundEffectsEnabled属性和Activity的excludeFromRecents属性

<img src="../flower/flower_p24.png">

---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)


[返回 P34: 系统开发之Display](https://github.com/hfreeman2008/android_core_framework/blob/main/P34_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay.md)


---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---


# 标签属性定义


frameworks\base\core\res\res\values\public.xml

```xml
<public type="attr" name="clearTaskOnLaunch" id="0x01010015" />
<public type="attr" name="stateNotNeeded" id="0x01010016" />
//是否不在最近应用列表中显示
<public type="attr" name="excludeFromRecents" id="0x01010017" />

//声音效果
<public type="attr" name="soundEffectsEnabled" id="0x01010215" />
```

frameworks\base\core\res\res\values\attrs_manifest.xml

```xml
<!-- Indicates that an Activity does not need to have its freeze state
     (as returned by {@link android.app.Activity#onSaveInstanceState}
     retained in order to be restarted.  Generally you use this for activities
     that do not store any state.  When this flag is set, if for some reason
     the activity is killed before it has a chance to save its state,
     then the system will not remove it from the activity stack like
     it normally would.  Instead, the next time the user navigates to
     it its {@link android.app.Activity#onCreate} method will be called
     with a null icicle, just like it was starting for the first time.

     <p>This is used by the Home activity to make sure it does not get
     removed if it crashes for some reason. -->
<attr name="stateNotNeeded" format="boolean" />

<!-- Indicates that an Activity should be excluded from the list of
     recently launched activities. -->
//是否不在最近应用列表中显示
<attr name="excludeFromRecents" format="boolean" />
```

frameworks\base\core\res\res\values\attrs.xml

```xml
<!-- Boolean that controls whether a view should have sound effects
     enabled for events such as clicking and touching. -->
//声音效果
<attr name="soundEffectsEnabled" format="boolean" />
```

---

# soundEffectsEnabled

(1)在应用中针对view来定义：
```xml
android:soundEffectsEnabled="false"
```

(2)功能的具体实现
frameworks\base\core\java\android\view\View.java

```java
//定义标识位
int viewFlagValues = 0;
int viewFlagMasks = 0;

//初始化
View.View方法
case com.android.internal.R.styleable.View_soundEffectsEnabled:
    if (!a.getBoolean(attr, true)) {
        viewFlagValues &= ~SOUND_EFFECTS_ENABLED;
        viewFlagMasks |= SOUND_EFFECTS_ENABLED;
    }
    break;
```

对标识位的操作接口

```java
/**
 * Set whether this view should have sound effects enabled for events such as
 * clicking and touching.
 *
 * <p>You may wish to disable sound effects for a view if you already play sounds,
 * for instance, a dial key that plays dtmf tones.
 *
 * @param soundEffectsEnabled whether sound effects are enabled for this view.
 * @see #isSoundEffectsEnabled()
 * @see #playSoundEffect(int)
 * @attr ref android.R.styleable#View_soundEffectsEnabled
 */
public void setSoundEffectsEnabled(boolean soundEffectsEnabled) {
    setFlags(soundEffectsEnabled ? SOUND_EFFECTS_ENABLED: 0, SOUND_EFFECTS_ENABLED);
}

/**
 * @return whether this view should have sound effects enabled for events such as
 *     clicking and touching.
 *
 * @see #setSoundEffectsEnabled(boolean)
 * @see #playSoundEffect(int)
 * @attr ref android.R.styleable#View_soundEffectsEnabled
 */
@ViewDebug.ExportedProperty
@InspectableProperty
public boolean isSoundEffectsEnabled() {
    return SOUND_EFFECTS_ENABLED == (mViewFlags & SOUND_EFFECTS_ENABLED);
}
```

决定是否可以播放声音效果

```java
/**
 * Play a sound effect for this view.
 *
 * <p>The framework will play sound effects for some built in actions, such as
 * clicking, but you may wish to play these effects in your widget,
 * for instance, for internal navigation.
 *
 * <p>The sound effect will only be played if sound effects are enabled by the user, and
 * {@link #isSoundEffectsEnabled()} is true.
 *
 * @param soundConstant One of the constants defined in {@link SoundEffectConstants}
 */
public void playSoundEffect(int soundConstant) {
    if (mAttachInfo == null || mAttachInfo.mRootCallbacks == null || !isSoundEffectsEnabled()) {
        return;
    }
    mAttachInfo.mRootCallbacks.playSoundEffect(soundConstant);
}
```

播放音效的具体实现
frameworks\base\core\java\android\view\ViewRootImpl.java

```java
public final class ViewRootImpl implements ViewParent,
        View.AttachInfo.Callbacks, ThreadedRenderer.DrawCallbacks {
            
public void playSoundEffect(int effectId) {
    checkThread();
    try {
        final AudioManager audioManager = getAudioManager();
        switch (effectId) {
            case SoundEffectConstants.CLICK:
                audioManager.playSoundEffect(AudioManager.FX_KEY_CLICK);
                return;
            case SoundEffectConstants.NAVIGATION_DOWN:
                audioManager.playSoundEffect(AudioManager.FX_FOCUS_NAVIGATION_DOWN);
                return;
            case SoundEffectConstants.NAVIGATION_LEFT:
                audioManager.playSoundEffect(AudioManager.FX_FOCUS_NAVIGATION_LEFT);
                return;
            case SoundEffectConstants.NAVIGATION_RIGHT:
                audioManager.playSoundEffect(AudioManager.FX_FOCUS_NAVIGATION_RIGHT);
                return;
            case SoundEffectConstants.NAVIGATION_UP:
                audioManager.playSoundEffect(AudioManager.FX_FOCUS_NAVIGATION_UP);
                return;
            default:
                throw new IllegalArgumentException("unknown effect id " + effectId +
                        " not defined in " + SoundEffectConstants.class.getCanonicalName());
        }
    } catch (IllegalStateException e) {
        // Exception thrown by getAudioManager() when mView is null
        Log.e(mTag, "FATAL EXCEPTION when attempting to play sound effect: " + e);
        e.printStackTrace();
    }
}
```

整个代码流程清晰明了，简单易懂

---

# excludeFromRecents


(1)app中定义：

```xml
<activity android:name="com.android.internal.app.XXXXXActivity"
        android:excludeFromRecents="true">
```

(2)功能的具体实现

定义此activity的标志位

frameworks\base\core\java\android\content\pm\ActivityInfo.java

```java
/**
 * Bit in {@link #flags} that indicates that the activity should not
 * appear in the list of recently launched activities.  Set from the
 * {@link android.R.attr#excludeFromRecents} attribute.
 */
public static final int FLAG_EXCLUDE_FROM_RECENTS = 0x0020;
```

给其标志位赋值

frameworks\base\core\java\android\content\pm\PackageParser.java

PackageParser.parseActivity

```java
if (sa.getBoolean(R.styleable.AndroidManifestActivity_excludeFromRecents, false)) {
    a.info.flags |= ActivityInfo.FLAG_EXCLUDE_FROM_RECENTS;
}
```

frameworks\base\services\core\java\com\android\server\wm\ActivityRecord.java

ActivityRecord.ActivityRecord
```java
if ((aInfo.flags & FLAG_EXCLUDE_FROM_RECENTS) != 0) {
    intent.addFlags(FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS);
}
```

frameworks\base\core\java\android\content\Intent.java

```java
/**
 * If set, the new activity is not kept in the list of recently launched
 * activities.
 */
public static final int FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS = 0x00800000;
```

frameworks\base\services\core\java\com\android\server\wm\RecentTasks.java
```java
/**
 * @return whether the given active task should be presented to the user through SystemUI.
 */
@VisibleForTesting
boolean isVisibleRecentTask(Task task) {
    if (DEBUG_RECENTS_TRIM_TASKS) Slog.d(TAG, "isVisibleRecentTask: task=" + task
            + " minVis=" + mMinNumVisibleTasks + " maxVis=" + mMaxNumVisibleTasks
            + " sessionDuration=" + mActiveTasksSessionDurationMs
            + " inactiveDuration=" + task.getInactiveDuration()
            + " activityType=" + task.getActivityType()
            + " windowingMode=" + task.getWindowingMode()
            + " intentFlags=" + task.getBaseIntent().getFlags());

    switch (task.getActivityType()) {
        case ACTIVITY_TYPE_HOME:
        case ACTIVITY_TYPE_RECENTS:
        case ACTIVITY_TYPE_DREAM:
            // Ignore certain activity types completely
            return false;
        case ACTIVITY_TYPE_ASSISTANT:
            // Ignore assistant that chose to be excluded from Recents, even if it's a top
            // task.
            if ((task.getBaseIntent().getFlags() & FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS)
                    == FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS) {
                return false;
            }
            break;
    }
```


```java
/**
 * @return whether the given visible task is within the policy range.
 */
private boolean isInVisibleRange(Task task, int taskIndex, int numVisibleTasks,
        boolean skipExcludedCheck) {
    if (!skipExcludedCheck) {
        // Keep the most recent task even if it is excluded from recents
        final boolean isExcludeFromRecents =
                (task.getBaseIntent().getFlags() & FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS)
                        == FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS;
        if (isExcludeFromRecents) {
            if (DEBUG_RECENTS_TRIM_TASKS) Slog.d(TAG, "\texcludeFromRecents=true");
            return taskIndex == 0;
        }
    }

    if (mMinNumVisibleTasks >= 0 && numVisibleTasks <= mMinNumVisibleTasks) {
        // Always keep up to the min number of recent tasks, after that fall through to the
        // checks below
        return true;
    }

    if (mMaxNumVisibleTasks >= 0) {
        // Always keep up to the max number of recent tasks, but return false afterwards
        return numVisibleTasks <= mMaxNumVisibleTasks;
    }

    if (mActiveTasksSessionDurationMs > 0) {
        // Keep the task if the inactive time is within the session window, this check must come
        // after the checks for the min/max visible task range
        if (task.getInactiveDuration() <= mActiveTasksSessionDurationMs) {
            return true;
        }
    }

    return false;
}
```


```java
/**
 * @return ids of tasks that are presented in Recents UI.
 */
SparseBooleanArray getRecentTaskIds() {
    final SparseBooleanArray res = new SparseBooleanArray();
    final int size = mTasks.size();
    int numVisibleTasks = 0;
    for (int i = 0; i < size; i++) {
        final Task task = mTasks.get(i);
        if (isVisibleRecentTask(task)) {
            numVisibleTasks++;
            if (isInVisibleRange(task, i, numVisibleTasks, false /* skipExcludedCheck */)) {
                res.put(task.mTaskId, true);
            }
        }
    }
    return res;
}
```



---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#view的控件xml属性和activity的xml属性解析)

---


[返回 P34: 系统开发之Display](https://github.com/hfreeman2008/android_core_framework/blob/main/P34_%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E4%B9%8BDisplay.md)






---

# 结束语

<img src="../Images/end_001.png">