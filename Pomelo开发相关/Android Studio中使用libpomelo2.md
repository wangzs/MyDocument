# Android Studio中使用libpomelo2
## 编译libpomelo2出错解决
#### 1. undefined reference to 'getifaddrs'问题
```error
D:\SvnSpace\mcbox\mconline\app\src\main\jni\libpomelo2\deps\uv\src\unix\linux-core.c
Error:(770) undefined reference to 'getifaddrs'
Error:(848) undefined reference to 'freeifaddrs'
Error:error: linker command failed with exit code 1 (use -v to see invocation)
```
**解决方法**
```code
修改：libpomelo2\deps\uv\Android.mk，向其中添加如下：
    src/unix/android-ifaddrs.c \
```


#### 2.nknown type name 'pthread_rwlock_t'的问题
```error
D:\SvnSpace\mcbox\mconline\app\src\main\jni\libpomelo2\deps\uv\include\uv-unix.h
Error:(135, 9) error: unknown type name 'pthread_rwlock_t'

原因：Android sdk 2.3 以下 不支持 pthread_rwlock_t
```
**解决方法**
```code
向src\main\jni\Application.mk文件中添加
APP_PLATFORM := android-9
```



## Android项目中pomelo2的配置示例
== 目录结构 ==
> main/jni目录中加入libpo2的目录，在jni目录中添加Android.mk和Application.mk两个文件

**Android.mk内容示例**
```mk
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

LOCAL_MODULE := mconline_shared
LOCAL_MODULE_FILENAME := libmconline

LOCAL_SRC_FILES := \
$(LOCAL_PATH)/runtime/com_netease_pomelo_Client.c

LOCAL_C_INCLUDES := \
$(LOCAL_PATH)/runtime \
$(LOCAL_PATH)/libpomelo2/deps/uv/include \
$(LOCAL_PATH)/libpomelo2/include

LOCAL_WHOLE_STATIC_LIBRARIES := pomelo_static

LOCAL_CFLAGS    := -D__ANDROID__
LOCAL_CFLAGS    += -DPC_NO_UV_TLS_TRANS

include $(BUILD_SHARED_LIBRARY)

$(call import-module, libpomelo2)
```
** 文件Application.mk内容示例 **
```mk
APP_STL := gnustl_static
#use clang by default, uncomment next line to use gcc4.8
#NDK_TOOLCHAIN_VERSION=4.8
NDK_TOOLCHAIN_VERSION=clang

APP_CPPFLAGS := -frtti -DCC_ENABLE_CHIPMUNK_INTEGRATION=1 -std=c++11 -fsigned-char
APP_CPPFLAGS += -Wno-error=format-security
APP_LDFLAGS := -latomic

APP_ABI := armeabi armeabi-v7a
APP_PLATFORM := android-9

ifeq ($(NDK_DEBUG),1)
  APP_OPTIM := debug
else
  APP_OPTIM := release
endif
```






