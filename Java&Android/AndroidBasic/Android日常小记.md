# Android的日常开发
## 一、系统功能相关
#### 1.1 判断某个包是否正在运行
```java
public boolean isAppRunning(Context context, String packageName) {
    ActivityManager manager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    List<ActivityManager.RunningAppProcessInfo> runningList = manager.getRunningAppProcesses();
    if (runningList != null && runningList.size() > 0) {
        for (ActivityManager.RunningAppProcessInfo info : runningList) {
            if (StringUtils.equals(info.processName, packageName)) {  // 注：StringUtils为apache的commons库中工具类
                return true;
            }
        }
    }
    return false;
}
```
#### 1.2 [Java中执行shell][2]
```java
public static void sudo(String...strings) {
    try{
        Process su = Runtime.getRuntime().exec("su");
        DataOutputStream outputStream = new DataOutputStream(su.getOutputStream());

        for (String s : strings) {
            outputStream.writeBytes(s+"\n");
            outputStream.flush();
        }

        outputStream.writeBytes("exit\n");
        outputStream.flush();
        try {
            su.waitFor();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        outputStream.close();
    }catch(IOException e){
        e.printStackTrace();
    }
}

// 使用例子
public static void suMkDir(String path) {
  if (!new File(path).isDirectory()) {
    sudo("mkdir -p "+path);
  }
}
```

#### 1.3 App的install与uninstall
```java
// install APK
public static void installAPK(String apkPath) {
  // apkPath如: "/sdcard/0/xxx/abc.apk"
  Intent intent = new Intent(Intent.ACTION_VIEW);
  intent.setDataAndType(Uri.fromFile(new File(apkPath)), "application/vnd.android.package-archive");
  startActivity(intent);
}

// uninstall APK
public static void uninstallAPK(String packageName) {
  Intent intent = new Intent(Intent.ACTION_DELETE, 
                      Uri.fromParts("package", packageName, null));
  startActivity(intent);
}
// ACTION_INSTALL_PACKAGE && ACTION_UNINSTALL_PACKAGE
```




## 二、UI相关
#### 2.1 使用[Dialog][1]注意事项
* 不要直接创建Dialog基类实例，多使用`AlertDialog`、`DatePickerDialog`或者`TimePickerDialog`。
* 因为Dialog如果打开后，屏幕旋转后会出现leaked相关的异常，因为Activity销毁前，所有的dialog都需要dismiss掉，需要在`onDestroy`中dismiss掉已经打开的dialog；同时为了恢复旋转前的dialog的打开状态也需要在`onSaveInstanceState`中保存dialog是否打开的状态，以便在`onCreate(Bundle savedInstanceState)`中利用`savedInstanceState`来获取dialog之前的状态，并恢复旋转屏幕前状态。
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    if (savedInstanceState != null && savedInstanceState.getBoolean("dialog_state")) {
      dialog.show();  // 显示之前的dialog
    }
}
@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    if(dialog != null && dialog.isShowing()) {
        outState.putBoolean("dialog_state", true);
    }
    else {
        outState.putBoolean("dialog_state", false);
    }
}
```
* 上面的几种dialog使用都比较麻烦，最好都改用`DialogFragment`来代替（官方推荐使用此类）
```java
// 一个只有title的dialog
new DialogFragment() {
    @NonNull
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
        builder.setTitle("测试AlertDialog");
        return builder.create();
    }
}.show(getActivity().getSupportFragmentManager(), "testDialogFragment");
// 也可以自定义content区域的内容：
// builder.setView()  builder.setItems()等方法可以快速设置dialog的一些固定样式内容
```

* 可以使用Activity的dialog属性，让Activity像dialog一样的显示效果: 
```xml
<!-- 一般版本的dialog的显示属性 -->
<activity android:theme="@android:style/Theme.Dialog" />
<!-- 对Appcompat版本的dialog的显示属性 -->
<style name="mytheme" parent="Theme.AppCompat.Light.Dialog">
    <item name="windowNoTitle">true</item>
</style>
```

#### 2.2 一个自定义字体的TextView
```java
public class CustomTextView extends TextView {
    public CustomTextView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }
    public CustomTextView(Context context) {
        super(context);
    }
    public CustomTextView(Context context, AttributeSet attrs)
    {
        super(context, attrs);
        this.setTypeface(Typeface.createFromAsset(context.getAssets(), "fonts/xxx.TTF")); // 字体是放在assert/fonts目录中的
    }
}
```






## 三、其它
#### 3.1 Umeng相关
* 强制更新（结合umeng更新与umeng在线参数）
```java
public interface OnForceUpdateListener {
    void doSomething();
}
public static void foreceUmengUpdate(final Activity activity, final OnForceUpdateListener listener) {
       int needUpdateVersion = 12;    // 12：低于该值的都被强制更新；实际可通过umeng在线参数获取其数值
       if (needUpdateVersion == 0) {
           return ;
       }
       try {
           int currVersion = activity.getPackageManager().getPackageInfo(activity.getPackageName(), 0).versionCode;
           if (currVersion < needUpdateVersion) {
               UmengUpdateAgent.setUpdateOnlyWifi(false);  // 在任意网络环境下都进行更新自动提醒
               UmengUpdateAgent.forceUpdate(activity);     // 调用umeng更新接口

               UmengUpdateAgent.setDialogListener(new UmengDialogButtonListener() {
                   @Override
                   public void onClick(int status) {
                       switch (status) {
                           case UpdateStatus.Update:
                               // 关闭页面，并开始下载升级
                               activity.finish();
                               break;
                           default:
                               // 回调中弹出对话框提示用户需要升级，不然强制用户退出
                               listener.doSomething();
                       }
                   }
               });
           } else {
               UmengUpdateAgent.setDefault();
               UmengUpdateAgent.update(activity);
           }
       } catch (PackageManager.NameNotFoundException e) {
           e.printStackTrace();
       }
   }
```








[1]:http://developer.android.com/guide/topics/ui/dialogs.html
[2]:http://stackoverflow.com/questions/20932102/execute-shell-command-from-android/26654728#26654728
[3]:
