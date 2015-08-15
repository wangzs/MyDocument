# [Android.mk文件语法][4]
## 一、基本概念
* Android中C/C++的静态库，编译后会生产对应的xxx.a的文件
* Android中C/C++的动态库，在编译后会生产对应的xxx.so的文件
==[注]：xxx.a文件为了引入到apk中，一般会最终编译附加到xxx.so的文件中，最后apk引用xx.so文件==

## 二、基本写法
```code
// 第一步：A module should start with LOCAL_PATH
LOCAL_PATH := $(call my-dir)

// 第二步：clears many LOCAL_XXX variables（not clear LOCAL_PATH）
include $(CLEAR_VARS)

// 第三步：所有模块中唯一的名字，不能包含任何空格：下面的会生产出libfoo.so名字的库
LOCAL_MODULE := foo
// 需要自己设置lib名字的可以根据下面的赋值(得到libnewfoo.so名字的库)
LOCAL_MODULE_FILENAME := libnewfoo

// 第四步：所有该模块需要的的c/c++源文件
LOCAL_SRC_FILES := hello-jni.c	\
				xxx.cpp

// 第五步：确定模块编译成静态库还是动态库
include $(BUILD_SHARED_LIBRARY)
```

## 三、变量和宏的详述
NDK构建系统中保留变量名：
> * LOCAL_开头的变量
* PRIVATE_, NDK_, 或者 APP开头的是构建系统内部使用的
* 小写的名字如my-dir同样是构建系统内部使用的

### 3.1 NDK定义的变量
#### == CLEAR_VARS ==
Points to a build script that undefines nearly all LOCAL_XXX variables listed in the "Developer-defined variables" section below. 每次在描述新的模块前使用。
**使用方法**
```code
include $(CLEAR_VARS)
```

#### == BUILD_SHARED_LIBRARY ==
使用该语句时，记得要已经给LOCAL_MODULE 和 LOCAL_SRC_FILES赋值了， shared-library variable导致编译系统生成的库是以`.so`为后缀的文件。
**使用方法**
```code
include $(BUILD_SHARED_LIBRARY)
```

#### == BUILD_STATIC_LIBRARY ==
使用该语句时，记得要已经给LOCAL_MODULE 和 LOCAL_SRC_FILES赋值了，static-library variable导致编译系统生成的库是以`.a`为后缀的文件。
**使用方法**
```code
include $(BUILD_STATIC_LIBRARY)
```


#### == PREBUILT_SHARED_LIBRARY ==
不同于BUILD_STATIC_LIBRARY和BUILD_SHARED_LIBRARY，此处的LOCAL_SRC_FILES赋值的是一个xx.so，一般是在引入一个本地的动态链接库时使用
**使用方法**
```code
include $(PREBUILT_SHARED_LIBRARY)


// 对应的完整例子
include $(CLEAR_VARS)
LOCAL_MODULE := yyutil
LOCAL_SRC_FILES := $(YYVOICE)/libyyutil.so
include $(PREBUILT_SHARED_LIBRARY)
```

#### == PREBUILT_STATIC_LIBRARY ==
类似于PREBUILT_SHARED_LIBRARY，只是LOCAL_SRC_FILES对应的是本地静态库


#### ==TARGET_ARCH==
一般用于在mk文件中作条件判断
```code
ifeq ($(TARGET_ARCH),arm)
LOCAL_SRC_FILES + = arm_xxx.c
else
LOCAL_SRC_FILES + = gen_xxx.c
endif
```

#### ==TARGET_PLATFORM==
指定构建系统的目标版本号：[Android NDK Native APIs.][1]
指定为Android 5.1系统时的用法：
```code
TARGET_PLATFORM := android-22
```


#### ==TARGET_ARCH_ABI==
详细的介绍 [ABI Management][2]
![](./img/arch_abi_list.jpg)
**使用样例**
```code
TARGET_ARCH_ABI := arm64-v8a
```

### 3.2 模块描述变量
#### ==LOCAL_PATH==
必须在Android.mk的开头定义，因为该变量不会被CLEAR_VARS清掉，故只用定义一次
**使用方法**
```code
LOCAL_PATH := $(call my-dir)
```

#### ==LOCAL_MODULE==
存放的是编译输出的模块名，会自动添加lib前缀以及对应的后缀名。如果需要自定义模块的输出名参考LOCAL_MODULE_FILENAME
**使用方法**
```code
LOCAL_MODULE := "foo"
```


#### ==LOCAL_MODULE_FILENAME==
可以自己定义编译出来的模块名字。
**使用方法**
```code
LOCAL_MODULE := foo
LOCAL_MODULE_FILENAME := libnewfoo
```

#### ==LOCAL_SRC_FILES==
指定模块编译需要的源码文件，路径可以是相对于LOCAL_PATH的相对路径或者使用绝对路径（总是使用/而不是windows的\）
**使用方法**
```code
LOCAL_SRC_FILES := \
	$(LOCAL_PATH)/runtime \
	src/pc_JSON.c \
	src/pc_lib.c \
	src/pc_pomelo.c
```

#### ==LOCAL_CPP_EXTENSION==
可以通过该变量指定非`.cpp`为后缀的文件为c++源文件
**使用方法**
```code
LOCAL_CPP_EXTENSION := .cxx

// From NDK r7, you can use this variable to specify multiple extensions
LOCAL_CPP_EXTENSION := .cxx .cpp .cc
```

#### ==LOCAL_CPP_FEATURES==
指定自己的code依赖了c++的一些特殊的特征
**使用方法**
```code
// Indicate that your code uses RTTI (RunTime Type Information)
LOCAL_CPP_FEATURES := rtti

// To indicate that your code uses C++ exceptions
LOCAL_CPP_FEATURES := exceptions

// You can also specify multiple values for this variable
LOCAL_CPP_FEATURES := rtti features
```

#### ==LOCAL_C_INCLUDES==
通过设定该值，指定include的搜索路径（该变量的定义需要在设置如LOCAL_CFLAGS、LOCAL_CPPFLAGS的变量前）
**使用方法**
```code
LOCAL_C_INCLUDES := $(LOCAL_PATH)/foo	\
					$(LOCAL_PATH)/libpomelo2/include
```

#### ==LOCAL_CFLAGS==
指定附加的宏定义或者编译选项（不论编译C或者C++文件，对应的选项值均会起作用）
**使用方法**
```code
LOCAL_CFLAGS    := -D__ANDROID__
LOCAL_CFLAGS    += -DPC_NO_UV_TLS_TRANS
LOCAL_CFLAGS 	+= -I(path)
```

#### ==LOCAL_CPPFLAGS==
同LOCAL_CFLAGS，不过仅仅在编译C++文件时才会传递。



#### ==LOCAL_STATIC_LIBRARIES==
指定了当前模块编译依赖的静态库列表。
* 如果当前模块是动态库，会强制将静态库linked到结果xx.so中
* 如果当前模块是静态库，仅仅表明当前的模块也会依赖列表中的静态库

#### ==LOCAL_SHARED_LIBRARIES==
指定当前模块在运行时需要依赖该变量列表中的动态库


#### ==LOCAL_WHOLE_STATIC_LIBRARIES==
与LOCAL_SHARED_LIBRARIES类似，如果在形如C中需要A、B的静态库，而静态库A又依赖了静态库B，
如果只使用LOCAL_WHOLE_STATIC_LIBRARIES时，C只会加载使用了A、B库中的函数，可能出现A中使用了B的函数却提示找不到。
LOCAL_WHOLE_STATIC_LIBRARIES时加载库的全部函数。


#### ==LOCAL_LDLIBS==
指定系统的动态链接库（如果是赋值了静态库，编译系统会忽略，并且ndk-build会打印warning）系统的库列表[Android NDK Native APIs][3]
**使用方法**
```code
// 实际引入的为 /system/lib/libz.so 的库
LOCAL_LDLIBS := -lz
```


### 3.2 NDK提供的函数宏
==my-dir：== returns the path of the last included makefile, which typically is the current Android.mk's directory


==all-subdir-makefiles：==Returns the list of Android.mk files located in all subdirectories of the current my-dir path.
==this-makefile：==Returns the path of the current makefile (from which the build system called the function).
==parent-makefile：==Returns the path of the parent makefile in the inclusion tree
==grand-parent-makefile：==Returns the path of the grandparent makefile in the inclusion tree

==import-module：==allows you to find and include a module's Android.mk file by the name of the module
```code
// 使用示例:下面会搜索NDK_MODULE_PATH路径下name指向的模块，并自动导入Android.mk
$(call import-module, （name）)
```

<strong style="background:red">NDK_MODULE_PATH：</strong>如果包含多个路径的话，使用的是==:==来分割且不可有空格，一般添加新路径的方法如下：
```code
$(call import-add-path, $(LOCAL_PATH)/xxx)

// 在上面的调用后再使用import-module比较推荐，如
$(call import-add-path, /path/to/somewhere)
$(call import-module,module_name)
```









































[1]:https://developer.android.com/ndk/guides/stable_apis.html
[2]:https://developer.android.com/ndk/guides/abis.html
[3]:https://developer.android.com/ndk/guides/stable_apis.html
[4]:https://developer.android.com/ndk/guides/android_mk.html







