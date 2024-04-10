# 专题三: AndroidStudio不同渠道打包

# 目的或需求
主要是一套代码，编译出二个不同包名的应用，并且可以自动根据不同的配置来动态替换包名，应用名，应用图标，字符串，图片等。

按比较专业的术语描述此需求为：
要实现一个壳工程打出不同样式的包，也就是使用Gradle中的productFlavors，在做定制或适配的时候，不需要建立多个工程、来回切换项目分支、逐个编译apk，使用productFlavors可以帮我们简化这一步操作，快速打包所有项目版本的apk。


# 实现效果
## 生成二个不同包名的apk，并且应用图标和应用名称实现动态替换
<img src=".\Image\logo_and_name.PNG">

图一：桌面显示二个不同包名应用动态替换不同应用图标和应用名称的效果图

## 图标，字符串实现动态替换
<img src=".\Image\activity_ui.PNG">

图二：显示二个不同应用图标和字符串动态替换的效果图

# Demo源码
这个demo源码，非常的简单，主要是显示一个activity界面，此界面包括二个TextView和一个ImageView。如上面的图二。

DemoDiffPkg2.zip
这个就是Demo的源码

# 实现方案

## app\build.gradle
```java
android.buildFeatures.buildConfig true

flavorDimensions "channel"
productFlavors{
    mtk{
        manifestPlaceholders = [str:"mtk",
                                package_name:"com.example.demodiffpkg2_mtk",
                                APP_LOGO_ICON:"@drawable/logo_mtk",
                                APP_NAME:"@string/app_name_mtk"]
        applicationId "com.example.demodiffpkg2_mtk"
        versionCode 1
        versionName "1.0"

        //resValue "string","app_name","demodiffpkg2_mtk"
        resValue "string","text_name","demodiffpkg2_mtk"

        buildConfigField("String", "HOST", "\"mtk\"")
        buildConfigField("boolean", "isEnable", "true")
    }

    qcom{
        manifestPlaceholders = [str:"qcom",
                                package_name:"com.example.demodiffpkg2_qcom",
                                APP_LOGO_ICON:"@drawable/logo_qcom",
                                APP_NAME:"@string/app_name_qcom"]
        applicationId "com.example.demodiffpkg2_qcom"
        versionCode 2
        versionName "2.0"

        //resValue "string","app_name","demodiffpkg2_qcom"
        resValue "string","text_name","demodiffpkg2_qcom"

        buildConfigField("String", "HOST", "\"qcom\"")
        buildConfigField("boolean", "isEnable", "false")
    }
}
```
## 其配置项的解释说明：
### 配置生成不同的包名
对应我们添加二个项目(mtk,qcom)，分别生成包名为(com.example.demodiffpkg2_mtk，com.example.demodiffpkg2_qcom)的二个apk应用。
```java
productFlavors{
    mtk{
        manifestPlaceholders = [
                                package_name:"com.example.demodiffpkg2_mtk",
                                ]
        applicationId "com.example.demodiffpkg2_mtk"
        ......
    }

    qcom{
        manifestPlaceholders = [
                                package_name:"com.example.demodiffpkg2_qcom",
                                ]
        applicationId "com.example.demodiffpkg2_qcom"
        ......
    }
}
```
### 动态替换应用图标和应用名
```java
productFlavors{
    mtk{
        manifestPlaceholders = [
                                APP_LOGO_ICON:"@drawable/logo_mtk",
                                APP_NAME:"@string/app_name_mtk"]
        ......
    }

    qcom{
        manifestPlaceholders = [
                                APP_LOGO_ICON:"@drawable/logo_qcom",
                                APP_NAME:"@string/app_name_qcom"]
        ......
    }
}
```

在app\src\main\AndroidManifest.xml中配置对应的动态应用图标和应用名
```java
<application
    android:icon="${APP_LOGO_ICON}"
    android:label="${APP_NAME}"
    >
```
logo_qcom.png和logo_mtk.png二张对应应用图标的图片：

<img src=".\Image\logo_mtk_qcom.png">

其不同的应用名对应：
app\src\main\res\values\strings.xml

```xml
<string name="app_name_mtk">demodiffpkg2_mtk</string>
<string name="app_name_qcom">demodiffpkg2_qcom</string>
```

### 资源的overlay（包括图片，字符串），也可以说是动态替换资源文件----特别推荐！！
这部分，主要是activity主界面的图片和字符串：

对于应用mtk，我们将图片显示为mtk，字符串显示为tv1 mtk；

对于应用qcom，我们将图片显示为QualcoMM，字符串显示为tv1 qcom；

<img src=".\Image\overlay_activity_ui.png">

图三：资源文件overlay动态替换的效果图

界面布局为：
```xml
<ImageView
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:layout_margin="30dp"
    android:background="@drawable/image"
    />

<TextView
    android:id="@+id/tv1"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_margin="30dp"
    android:text="@string/text_name_1"
    />
```

其主要的思想是：
在对应平行app/src/main此目录，添加我们定义的不同项目，分别添加需要overlay动态替换的资源文件。
这个完全就是android开发中常用的overlay替换的思路，此思路完全可以替换所有的资源文件，而不是仅仅为图片和字符串，并且此思路还有一个好处，就是可以兼顾到资源文件的国际化，各种不同的场景，最是适合我们的客制化开发，特别推荐。

如我们创建mtk和qcom二个目录：
```xml
mtk/java
mtk/res/drawable/image.png
mtk/res/values/strings.xml

qcom/java
qcom/res/drawable/image.png
qcom/res/values/strings.xml
```

字符串资源的动态替换：

mtk/res/values/strings.xml
```xml
<resources>
    <string name="text_name_1">tv1 mtk</string>
</resources>
```


qcom/res/values/strings.xml
```xml
<resources>
    <string name="text_name_1">tv1 qcom</string>
</resources>
```
其drawable下的图片资源，values下的字符串，color等其他的资源，都是可以根据不同的项目来overlay动态替换的。

<img src=".\Image\overlay_res_ui.png">


图四：资源文件overlay动态替换的代码结构




### resValue简单的字符串动态替换
上面的资源overlay动态替换作用非常大，并且还可以实现字符串的国符化，当然缺点是需要创建文件，resValue可以非常方便的实现简单的字符串动态替换
```java
flavorDimensions "channel"
productFlavors{
    mtk{
        ......
        resValue "string","text_name","demodiffpkg2_mtk"
    }

    qcom{
        ......
        resValue "string","text_name","demodiffpkg2_qcom"
    }
}
```

这个主要是实现主界面的第一个TextView的动态替换：

<img src=".\Image\resvalue_tv.png">


其布局文件为：
```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_margin="30dp"
    android:text="@string/text_name" />
```


其会自动生成为：
DemoDiffPkg2\app\build\generated\res\resValues\mtk\debug\values\gradleResValues.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- Automatically generated file. DO NOT MODIFY -->

    <!-- Value from product flavor: mtk -->
    <string name="text_name" translatable="false">demodiffpkg2_mtk</string>

</resources>
```



### buildConfigField动态配置
这个是主要是对应自动生成BuildConfig类，方便项目来客制化：
```java
android.buildFeatures.buildConfig true

flavorDimensions "channel"
productFlavors{
    mtk{
        ......
        buildConfigField("String", "HOST", "\"mtk\"")
        buildConfigField("boolean", "isEnable", "true")
    }

    qcom{
        ......
        buildConfigField("String", "HOST", "\"qcom\"")
        buildConfigField("boolean", "isEnable", "false")
    }
}
```


对应会自动生成类：
app\build\generated\source\buildConfig\mtk\debug\com\example\demodiffpkg2\BuildConfig.java
<img src=".\Image\buildconfig_auto.png">

java文件的调用方式为：
```java
boolean isOK = BuildConfig.isEnable;
```


### 动态替换版本号
这个非常明显，只是动态设置一下：versionCode，versionName
```java
flavorDimensions "channel"
productFlavors{
    mtk{
        ......
        versionCode 1
        versionName "1.0"
    }

    qcom{
        ......
        versionCode 2
        versionName "2.0"
    }
}
```


# 编译apk
在Build Variants----Active Build Variant中选择我们对应的项目来编译：

<img src=".\Image\make_build.png">


# 参考资料
https://blog.csdn.net/woshizisezise/article/details/96303750
Android实现多渠道打包，动态替换包名、Icon、图片等资源，解决因applicationId和BuildConfig路径不匹配的问题


# 附文档：
Android实现多渠道打包，动态替换包名、Icon、图片等资源的方案.pdf

# 附Demo的源码
DemoDiffPkg2.zip
