# android 问题汇总

## 编译

1. 异常编译问题：the number of method references in a .dex file cannot exceed 64k.

- 如果应用不需要兼容 Android5.0 以下版本的话，只需要修改/android/build.gradle 文件的 minSdkVersion 改为 21 皆可，然后重新打包。

```gradle
buildscript {
    ext {
        buildToolsVersion = "31.0.0"
        minSdkVersion = 21
        compileSdkVersion = 31
        targetSdkVersion = 31

        if (System.properties['os.arch'] == "aarch64") {
            // For M1 Users we need to use the NDK 24 which added support for aarch64
            ndkVersion = "24.0.8215888"
        } else {
            // Otherwise we default to the side-by-side NDK version from AGP.
            ndkVersion = "21.4.7075529"
        }
    }
  // ...
}
```

- 修改文件/android/app/build.gradle

```gradle
android {
    // ..

    defaultConfig {
      multiDexEnable true
    }
}
<!-- 同时添加一个依赖，兼容Android5.0以下 -->

dependencies {

  // ...
  implementation "androidx.multidex:multidex:2.0.1"
}
```

- 修改文件/android/app/src/main/java/com/marathon/MainApplication.java 重写一个方法

```java
import androidx.multiidex.MultiDex;

public class MainApplication extends Application implements ReactApplication {

  @Override
  protected void attachBaseContext(Context base) {
    super.attachBaseContext(base);
    MultiDex.install(this);
  }
}
```

### 编译后，Android 无法访问 http 协议

1. Android9.0 版本之后，禁止使用 http 协议，只能使用 HTTPS 协议，如果一定要使用 http，也是可以配置的

1.1 在/android/app/src/main/res 目录下创建一个 xml 的文件夹。

1.2 在该目录下创建一个 network_security_config.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<network-security-config>
  <base-config cleartextTrafficPermitted="true" />
</network-security-config>
```

1.3 配置/android/app/src/main/AndroidManifest.xml 文件

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.cogito">
    <!-- ... -->
    <application
      android:allowBackup="false"
      android:networkSecurityConfig="@xml/network_security_config"
      android:theme="@style/AppTheme">
    </application>
</manifest>

```

2. **注意：** 这个问题只有在编译之后才会出现。开发环境下，是可以访问 http 协议的。

## 配置
