#微会SDK的Android接入
## AndroidManifest.xml的修改
```xml
<!-- service declarations -->
<!-- NOTE: for YYSdkService & YYReceiver, set android:process:":svc" for individual process. -->
<service android:name="com.yy.sdk.service.YYSdkService" 
	android:process=":svc"/>
<service
	android:name="com.yysdk.mobile.mediasdk.YYMediaService" >
</service>
<service
	android:name="com.yysdk.mobile.videosdk.YYVideoService">
</service>
 <receiver android:name="com.yy.sdk.util.NetworkReceiver">
	<intent-filter>
		<action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
	</intent-filter>
</receiver>

<!-- only declare this receiver if background call is needed. -->
<receiver android:name="com.yy.sdk.service.YYReceiver"
	android:process=":svc">
	<intent-filter>
		<action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
	</intent-filter>
	<intent-filter>
		<action android:name="android.intent.action.EXTERNAL_APPLICATIONS_AVAILABLE" />
	</intent-filter>
	<intent-filter>
		<category android:name="android.intent.category.DEFAULT" />
		<action android:name="android.intent.action.PACKAGE_ADDED" />
		<action android:name="android.intent.action.PACKAGE_CHANGED" />
		<action android:name="android.intent.action.PACKAGE_INSTALL" />
		<action android:name="android.intent.action.PACKAGE_REMOVED" />
		<action android:name="android.intent.action.PACKAGE_REPLACED" />
		<data android:scheme="package" />
	</intent-filter>
	<intent-filter>
		<action android:name="android.intent.action.MEDIA_MOUNTED" />
		<action android:name="android.intent.action.MEDIA_EJECT" />
		<action android:name="android.intent.action.MEDIA_UNMOUNTED" />
		<action android:name="android.intent.action.MEDIA_SHARED" />
		<action android:name="android.intent.action.MEDIA_SCANNER_STARTED" />
		<action android:name="android.intent.action.MEDIA_SCANNER_FINISHED" />
		<action android:name="android.intent.action.MEDIA_REMOVED" />
		<action android:name="android.intent.action.MEDIA_BAD_REMOVA" />
		<data android:scheme="file" />
	</intent-filter>
	<intent-filter>
		<action android:name="android.intent.action.ACTION_HEADSET_PLUG" />
	</intent-filter>
	<intent-filter>
		<action android:name="android.intent.action.MEDIA_BUTTON" />
	</intent-filter>
	<intent-filter>
		<action android:name="com.yy.sdk.action.DUMMY" />
	</intent-filter>
</receiver>
```

## 微会的AppID和AppSecret

## 微会SDK的初始化
```java
// 微会SDK的对象
public YYMobileClient mMobileSdk;

mMobileSdk = YYMobileSDK.getSDKClient(this);
mMobileSdk.setEnvironment(EnvironmentType.TYPE_PUSHLISHED);
//注册SDK监听接口，监听SDK状态
mMobileSdk.addClientListener(this);


// 通过AppID, AppSecret, UserId进行登陆
// AuthType:
//     TYPE_YYUID(0),
//     TYPE_OAUTH(1),
//     TYPE_PASSWD(2),
//     TYPE_USERNAME(4);
mMobileSdk.login("appId", "appSecret", AuthType.TYPE_NONE, "userId", null);

```












