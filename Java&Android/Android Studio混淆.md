# Android Studio混淆

[官方使用文档][1]

[Guard使用语法][2]



## 一、启用ProGuard

创建了Android Studio工程后，`build.gradle`中的`minifyEnabled`属性表示release版本中是否启用ProGuar。

> 其中`getDefaultProguardFile('proguard-android.txt')`获取的是ProGuard的默认设置，可以在Android SDK的`tools/proguard/folder`中找到。 同样`proguard-android-optimize.txt`也可以使用，它会启用最优化的混淆。开启最优化混淆时，ProGuard会分析字节码，会使得app更小并运行更快。



使用`proguard-rules.pro`来定制自己的ProGuard规则，

## 二、配置ProGuard
对于一些情况，使用默认的ProGuard的配置文件就可以满足需求，但是对于一些其他情况，可能ProGuard移除掉其认为不使用的代码：
> * 只在`AndroidManifest.xml`中引用了的class
* 在JNI中调用了的方法
* 动态引用的字段和方法

在使用了默认的ProGuard配置文件后，可能碰到`ClassNotFoundException`的异常，可以通过添加`-keep`来修复这样的错误。
```android
-keep public class <MyClass>
```

## 三、ProGuard使用
#### Keep的使用
* -keep [,modifier,...] class_specification
> 指定代码中所有使用到类和类成员的地方都不做混淆
* -keepclassmembers [,modifier,...] class_specification
> 类被保护的同时，指定类成员被保护。
* -keepclasseswithmembers [,modifier,...] class_specification
> 指定类和类成员被保护，在这种情况下，所有指定的类成员将被保护。

...



## 四、例子
```java
// ---- 一些引入的SDK的混淆示例
-keep class com.xx.**{*;}		// 针对所有com.xx.前缀的类名的不混淆
-keep class com.yy.zz			// 具体到某个SDK中的类

// google的Gson库混淆中用到的
-keepattributes Signature
-dontwarn com.google.gson.**
-keep class sun.misc.Unsafe { *; }
-keep class com.google.gson.** { *; }
-keep class com.google.gson.JsonObject { *; }
-keep class com.google.gson.examples.android.model.** { *; }
-keepclassmembers class * implements java.io.Serializable {
   static final long serialVersionUID;
   private static final java.io.ObjectStreamField[] serialPersistentFields;
   private void writeObject(java.io.ObjectOutputStream);
   private void readObject(java.io.ObjectInputStream);
   java.lang.Object writeReplace();
   java.lang.Object readResolve();
}
-keep public class * implements java.io.Serializable {*;}
```

























































































































[1]:http://developer.android.com/tools/help/proguard.html

[2]:http://proguard.sourceforge.net/index.html#manual/usage.html