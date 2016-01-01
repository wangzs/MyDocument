# Android反编译

## 1. 结合IDA调试so
#### 1.1 基本调试方法
```sh
C:\Users\zs>adb push d:\Tools\IDAPro6.6\dbgsrv\android_server /data/local/tmp
7235 KB/s (570904 bytes in 0.077s)

// 使用abd进入shell
C:\Users\zs>adb shell

// 机器需要root才能执行下面的（否则出现/system/bin/sh: su: not found）
shell@mocha:/ $ su
root@mocha:/ # cd data/local/tmp/

// 查看tmp目录中是否有andorid_server文件
root@mocha:/data/local/tmp # ls

// 修改其权限
root@mocha:/data/local/tmp # chmod 755 android_server

// 打开该程序
root@mocha:/data/local/tmp # ./android_server
IDA Android 32-bit remote debug server(ST) v1.17. Hex-Rays (c) 2004-2014
Listening on port #23946...
=========================================================
[1] Accepting connection from 127.0.0.1...


// 新开一个命令行窗口
C:\Users\zs> adb forward tcp:23946 tcp:23946

// 将需要调试的so文件放到IDA中，选择IDA中[debugger]-[Attach to process...]
* 修改弹出窗口中host为: localhost或者127.0.0.1   port：23946
* 点击确认后，弹出当前手机中进程列表，选择需要调试的进程（如果进程不存在，先点击取消，并在手机中打开app，再重新attach）
```