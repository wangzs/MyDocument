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




## 二、UI相关
