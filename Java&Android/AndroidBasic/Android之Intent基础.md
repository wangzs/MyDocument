# Android之[Intent][1]
## 1. Intent的几种使用方法






## 2. 与Android系统相关Intent的使用
#### 2.1 系统自带相关app调用
http://blog.csdn.net/lilu_leo/article/details/6938729
* 相册：

* 联系人：
	```java
	// 打开联系人界面（仍在当前app应用内）
	Intent intent = new Intent();
	intent.setAction(Intent.ACTION_VIEW);
	intent.setData(ContactsContract.Contacts.CONTENT_URI);
	startActivity(intent);

	// 打开联系人界面，并定位到对应id为2的联系人
	Intent intent = new Intent();
	intent.setAction(Intent.ACTION_VIEW);
	intent.setData(Uri.parse("content://contacts/people/2"));
	startActivity(intent);

	```
* 相机：

* 电话：

* 通话记录

* 浏览器：

* 短信：



## 3. 数据传输之对象（序列化相关）













[1]:http://developer.android.com/reference/android/content/Intent.html
