---
title: Android的ndk开发
date: 2018-02-16 00:00:00
categories:
- CTF/Android
tags: Android NDK JNI 
---



## 相关概念

> 参考：[Android：JNI 与 NDK到底是什么？（含实例教学)](http://blog.csdn.net/carson_ho/article/details/73250163)

### JNI

- 定义：Java Native Interface，即 Java本地接口
- 作用：使得Java与本地其他类型语言（如C、C++）交互，即在Java代码里调用C、C++等语言的代码或C、C++代码调用Java代码
- 说明：JNI是Java调用Native语言的一种特性，是属于Java的，与Android无直接关系

### NDK

- 定义：Native Development Kit，是 Android的一个工具开发包 
- 作用：快速开发C、C++的动态库，并自动将so和应用一起打包成 APK 
即可通过 NDK在 Android中 使用 JNI与本地代码（如C、C++）交互
- 安装：使用Android Studio中SDK Manager直接安装即可，目录:`sdk/ndk-bundle`

### JNI与NDK关系

![image](http://upload-images.jianshu.io/upload_images/944365-6607e9321d3cbddd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> JNI是一套编程接口，用来实现Java代码与本地的C/C++代码进行交互；  
> NDK是Google开发的一套开发和编译工具集，可以生成动态链接库，主要用于Android的JNI开发；

## 使用ndk-build编译可运行的二进制文件

### 新建目录并建立以下文件

```bash
➜  Desktop tree JNI 
JNI
├── Android.mk
├── Application.mk
└── main.c

```

### 建立NDK工程

Android.mk
```mk
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE    := ndk_sample
LOCAL_SRC_FILES := main.c
LOCAL_CFLAGS += -D DEBUG
LOCAL_LDFLAGS += -llog
LOCAL_LDLIBS += -L$(SYSROOT)/usr/lib
LOCAL_C_INCLUDES += $(LOCAL_PATH)/include
include $(BUILD_EXECUTABLE)
```

Application.mk

```mk
APP_ABI := armeabi-v7a
APP_PIE := true
NDK_TOOLCHAIN_VERSION := clang
APP_PLATFORM := android-21
```

main.c
```c
#include <stdio.h>
int main() {
  printf("hello world\n");
  return 0;
}
```

### 编译运行

在JNI工程目录下，或者其父目录执行ndk-build命令，推入手机运行

```bash
➜  Desktop ndk-build 
[armeabi-v7a] Compile thumb  : ndk_sample <= main.c
[armeabi-v7a] Executable     : ndk_sample
[armeabi-v7a] Install        : ndk_sample => libs/armeabi-v7a/ndk_sample
➜  Desktop adb push libs/armeabi-v7a/ndk_sample /data/local/tmp
libs/armeabi-v7a/ndk_sample: 1 file pushed. 0.2 MB/s (9644 bytes in 0.041s)
➜  Desktop adb shell
sagit:/ $ cd /data/local/tmp
sagit:/data/local/tmp $ ./ndk_sample
hello world
```

## 使用ndk-build编译动态链接库

### 在MainActivity中添加JNI调用声明

```java
public class MainActivity extends Activity{

	static{
		System.loadLibrary("native");
	}

	public native String getString();

}
```

### 使用javah生成native头文件

```bash
➜  cd ~/AndroidStudioProjects/MyApplication/app/src/main/java/
➜  javah com.xuanxuan.myapplication.MainActivity
➜  ls
com                                       com_xuanxuan_myapplication_MainActivity.h
```

将生成的native头文件复制到工程目录中

```c

/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class com_xuanxuan_myapplication_MainActivity */

#ifndef _Included_com_xuanxuan_myapplication_MainActivity
#define _Included_com_xuanxuan_myapplication_MainActivity
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     com_xuanxuan_myapplication_MainActivity
 * Method:    getString
 * Signature: ()Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_com_xuanxuan_myapplication_MainActivity_getString
  (JNIEnv *, jobject);

#ifdef __cplusplus
}
#endif
#endif

```

### 编写main.c

```c
#include "com_xuanxuan_myapplication_MainActivity.h"
JNIEXPORT jstring JNICALL Java_com_xuanxuan_myapplication_MainActivity_getString(JNIEnv*env, jobject thiz){
    return(*env) -> NewStringUTF(env ,"hello xuanxuan");
}
```

> 这里的用法要符合JNI的语法标准

- 函数声明固定格式：`JNIEXPORT`
- 返回类型：`jstring`
- JNI调用：`JNICALL`
- Java_完整类名_方法名：`Java_com_xuanxuan_myapplication_MainActivity_getString`
- 函数参数：调用jni.h封装好的函数指针：`JNIEnv`；Java类本身：`jobject`

### 修改Android.mk以下两项

```mk
LOCAL_MODULE    := native
include $(BUILD_SHARED_LIBRARY)
```

### 使用ndk-build编译并拷贝至工程中

工程目录为`main`目录下的`jniLibs/armeabi-v7a/`，编译apk运行即可

```bash
$ ndk-build
[armeabi-v7a] Compile thumb  : native <= main.c
[armeabi-v7a] SharedLibrary  : libnative.so
[armeabi-v7a] Install        : libnative.so =>
libs/armeabi-v7a/libnative.so
$ cd app/src/main
$ mkdir -p jniLibs/armeabi-v7a
$ cp .../armeabi-v7a/libnative.so jniLibs/armeabi-v7a
```

## Android Studio集成ndk开发

Android Studio2.2以上版本实现NDK开发，只需要在新建工程选择支持c++开发即可

- 配置好NDK后，Android Studio会自动生成C++文件并设置好调用的代码
- 你只需要根据需求修改C++文件 & Android就可以使用了



