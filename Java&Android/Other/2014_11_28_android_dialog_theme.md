#Android的Activity的载入动画
== 以Dialog类似方式出现，背景蒙灰，同时需要横向占满全屏 ==
- 自定义style：
```xml
    <style name="PopupActivity" parent="@android:style/Theme.Translucent">
        <item name="android:windowBackground">@drawable/transparent</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowIsTranslucent">true</item>
        <item name="android:backgroundDimEnabled">true</item>
    </style>
```
其中`<item name="android:backgroundDimEnabled">true</item>`的目的就是让上一层的Activity变暗
- 修改==Manifest.xml==中将需要此显示效果的Activity的属性：
```xml
<activity
	android:name="com.test.DimActivity"
	android:theme="@style/PopupActivity" >
</activity>
```
- 设置此类Activity的出场动画：
> 此类Activity若要相Dialog一样出内容部分外均有半透明效果，需要在Activity对应的**xml**文件中，设置非内容区域的背景色为透明如：`#00000000`
```java
overridePendingTransition(R.anim.push_bottom_in, R.anim.push_bottom_out);
```



http://www.oschina.net/question/54100_30266