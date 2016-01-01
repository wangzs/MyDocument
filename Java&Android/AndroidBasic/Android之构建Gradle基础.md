# Android之构建Gradle基础
## 1. 项目的build配置(项目所在的目录下的build.gradle)
top-level的项目gradle编译配置文件作用于所有子模块
```gradle
// buildscript 用于定义gradnle的仓库和依赖
buildscript {
    repositories {
		    // 支持的仓库有JCenter, MavenCentral, Ivy等
        jcenter()
        mavenCentral()
        maven { url "http://mvnrepository.com/artifact" }
    }
    dependencies {
        // 配置android的gradle插件
        classpath 'com.android.tools.build:gradle:1.0.1'

        // NOTE: Do not place your application dependencies here: they belong
        // in the individual module build.gradle files
    }
}

allprojects {
   repositories {
       jcenter()
   }
}
```
在Android Studio中，可以通过`File`->`Project Structure `查看项目的一些基本配置（Ctrl+Alt+Shift+S）

## 2. 子模块的build配置（各个模块之间不影响）
* Android的设置：
> *  compileSdkVersion
  * buildToolsVersion
* defaultConfig 和 productFlavors
> * manifest 的属性：applicationId / minSdkVersion / targetSdkVersion / test information
* buildTypes
> * 编译属性：debuggable / ProGuard / debug signing / version name suffix / testinformation
* 依赖
> remote依赖：[maven仓库][1]的库在gradle中使用`group:name:version`

一个[官方android的gradle例子][2]：

```gradle
// 应用gradle提供的android的插件，提供了构建和测试android应用的所需的东西
apply plugin: 'com.android.application'

// android{} 配置的是当前项目或模块构建时的相关参数
android {
    compileSdkVersion 20          // 编译用的android SDK版本
    buildToolsVersion "20.0.0"    // 构建工具的版本

    defaultConfig {
        applicationId "com.mycompany.myapplication"     // 应用的包名
        minSdkVersion 13
        targetSdkVersion 20
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            // 是否打开混淆
            minifyEnabled false
            // 压缩对齐生成的apk包
            zipAlignEnabled true
            // 设置release的签名（需要设置signingConfigs）
            signingConfig signingConfigs.release
            // 去掉没用的资源
            shrinkResources true
            // 混淆文件的位置
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
         debug {
            debuggable true
        }
    }

    // 设置signingConfigs
    signingConfigs {
        debug {
        }
        release {
            // 实际的值从另一个不提交到版本管理的一个文件signing.properties中获取
            storeFile
            storePassword
            keyAlias
            keyPassword
        }
    }

    lintOptions {
      // 是否关闭lint检查的error
      abortOnError false
      checkReleaseBuilds false
    }
}

File propFile = file('signing.properties');
if (propFile.exists()) {
    def Properties props = new Properties()
    props.load(new FileInputStream(propFile))
    if (props.containsKey('STOREFILE') && props.containsKey('STOREPASSWORD') &&
            props.containsKey('KEYALIAS') && props.containsKey('KEYPASSWORD')) {
        android.signingConfigs.release.storeFile = file(props['STOREFILE'])
        android.signingConfigs.release.storePassword = props['STOREPASSWORD']
        android.signingConfigs.release.keyAlias = props['KEYALIAS']
        android.signingConfigs.release.keyPassword = props['KEYPASSWORD']
    } else {
        android.buildTypes.release.signingConfig = null
    }
} else {
    android.buildTypes.release.signingConfig = null
}

dependencies {
    // 编译build.gradle所在目录下的libs目录中所有的jar包
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:20.0.0'

    // 编译app2的子模块
    compile project(path: ':app2, configuration: 'android-endpoints')
}
```
记得在`local.properties`文件设置sdk路径：`sdk.dir=sdk.dir=D\:\\Program Files\\Android`；也可以设置名为ANDROID_HOME的环境变量

## 3. [具体的构建配置][3]
#### 3.1 Manifest相关的配置
* minSdkVersion
* targetSdkVersion
* versionCode
* versionName
* applicationId (the effective packageName)
* testApplicationId (used by the test APK)
* testInstrumentationRunner

配置可以动态设置，如：
```gradle
def computeVersionName() {
    // 实际逻辑
}

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"

    defaultConfig {
        versionCode 12
        versionName computeVersionName()
        minSdkVersion 16
        targetSdkVersion 23
    }
}
```

#### 3.2 buildTypes设置
示例：
```gradle
android {
  buildTypes {
  	debug {
      //添加设置调试时的一些编译参数：在java代码中就可以通过BuildConfig.LOG_DEBUG获取设置的值来控制一些log的输出
        buildConfigField "boolean", "LOG_DEBUG", "true"
        applicationIdSuffix ".debug"      // 修改debug版的包名
  	}
    // 用buildTypes.debug初始化jnidebug，即创建了一个新的debugTypes
  	jnidebug.initWith(buildTypes.debug)
    // 创建了jnidebug后对其进行配置
  	jnidebug {
  	   packageNameSuffix ".jnidebug"
  	   jniDebuggable true
  	}
  }
}
```
创建一个buildType后，默认会创建一个匹配的sourceSet：`src/<buildtypename>`，也可以重设source set的位置：
```
android {
  sourceSets.jnidebug.setRoot('foo/jnidebug')
}
```

#### 3.3 签名的设置
一个应用签名需要：
* keystore
* keystore的密码
* key的别名和密码
* 存储的类别

debug的Build Type被设置为自动使用debug签名配置，debug keystore的路径在：`个人目录/.android/debug.keystore`
自定义配置签名：
```gradle
android {
    signingConfigs {
        debug {
            storeFile file("debug.keystore")
        }

        myConfig {ss
            storeFile file("other.keystore")
            storePassword "android"
            keyAlias "androiddebugkey"
            keyPassword "android"
        }
    }

    buildTypes {
        release {
            debuggable true
            jniDebuggable true
            signingConfig signingConfigs.myConfig
        }
    }
}
```
上面的签名方式不安全，可以通过设置一个不会出现在版本控制的文件中，再在gradle中获取设置的值。如创建一个名`signing.properties`的文件：
```file
STOREFILE=F:\\keystore\\my_keystore
STOREPASSWORD=my_store_password
KEYALIAS=my_key_alias
KEYPASSWORD=my_key_password
```
并在gradle配置中设置如下：
```gradle
android {
    signingConfigs {
        debug {
            storeFile file("debug.keystore")
        }

        release {
            storeFile
            storePassword
            keyAlias
            keyPassword
        }
    }

    buildTypes {
       release {
           signingConfig signingConfigs.release
       }
   }
}

File propFile = file('signing.properties');
if (propFile.exists()) {
    def Properties props = new Properties()
    props.load(new FileInputStream(propFile))
    if (props.containsKey('STOREFILE') && props.containsKey('STOREPASSWORD') &&
            props.containsKey('KEYALIAS') && props.containsKey('KEYPASSWORD')) {
        android.signingConfigs.release.storeFile = file(props['STOREFILE'])
        android.signingConfigs.release.storePassword = props['STOREPASSWORD']
        android.signingConfigs.release.keyAlias = props['KEYALIAS']
        android.signingConfigs.release.keyPassword = props['KEYPASSWORD']
    } else {
        android.buildTypes.release.signingConfig = null
    }
} else {
    android.buildTypes.release.signingConfig = null
}
```

#### 3.4 混淆的设置
```gradle
android {
    buildTypes {
        release {
            minifyEnabled true    // 是否开启混淆
            proguardFile getDefaultProguardFile('proguard-android.txt')
        }
    }

    // 设置不同的渠道
    productFlavors {
        flavor1 {
        }
        flavor2 {
            // 针对特定的渠道作一些额外的混淆
            proguardFile 'some-other-rules.txt'
        }
    }
}
```

#### 3.5 多渠道打包
* 首先在AndroidManifest.xml中设置PlaceHolder
  ```xml
  <meta-data
    android:name="UMENG_CHANNEL"
    android:value="${UMENG_CHANNEL_VALUE}" />
  ```
* 在gradle文件中设置：
  ```gradle
  // 一个个设置
  productFlavors {
        yingyongbao {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "yingyongbao"]
        }
        xiaomi {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"]
        }
        wandoujia {
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "wandoujia"]
        }
    }

    // 或者使用批量修改
    productFlavors {
        yingyongbao {}
        xiaomi {}
        wandoujia {}
    }
     // 批量处理
    productFlavors.all {
           flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
    }
  ```
<strong  style="background:#687645">渠道包非常多时，打包会很耗时，推荐[美团打包方案][4]</strong>


## 4.依赖库
在`android`元素外添加`dependencies`：
```gradle
dependencies {
  // 配置libs目录下所有的jar包
  compile fileTree(dir: 'libs', include: ['*.jar'])
  // 单独配置xxx目录下的foo.jar包
  compile files('xxx/foo.jar')

  // 添加远程仓库的包
  compile 'io.reactivex:rxjava:1.0.14'
  compile 'io.reactivex:rxandroid:1.0.1'

  // 依赖本地的另一个库工程(注意不要循环引用)
	compile	project(':otherlib')
}
```
主要的依赖配置：
* compile：	main	application
* androidTestCompile：	test	application
* debugCompile：	debug	Build	Type
* releaseCompile：	release	Build	Type


### 一些资料
* [Android Development With Gradle][5]
* [Gradle for Android][6]


[1]:http://search.maven.org/
[2]:https://developer.android.com/tools/building/plugin-for-gradle.html
[3]:http://google.github.io/android-gradle-dsl/
[4]:http://tech.meituan.com/mt-apk-packaging.html
[5]:https://www.youtube.com/watch?v=0bhbQdZLpIE
[6]:https://www.youtube.com/watch?v=rXww768LUUM
