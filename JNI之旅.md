### 一.下载并导入NDK
### 二.配置环境变量
###### 1.NDK_HOME : NDK的路径
###### 2.Path : 尾部添加;%NDK_HOME%
### 三.编写hello.c文件

```
#include <stdlib.h>
#include <stdio.h>
#include <jni.h>

//jstring
//Java_com_example_hellojni_HelloJni_stringFromJNI( JNIEnv* env,
//                                                  jobject thiz )
//JNIEnv* env 是结构体JNINativeInterface 的二级指针
// JNIEnv 是结构体JNINativeInterface 的一级指针
// JNINativeInterface结构体中定义了大量的函数指针 这些函数指针在jni开发中很常用
// (*env)->
//jobject  调用本地函数的java对象 在这个例子中 就是MainActivity的实例
//c本地函数命名规则  Java_包名_类名_本地方法名
//jstring     (*NewStringUTF)(JNIEnv*, const char*);
jstring Java_cn_yjjc_com_jnistudy_MainActivity_helloFromJNI(JNIEnv* env,jobject thiz){
	char* cstr = "hello from c!";
	return (*env)->NewStringUTF(env,cstr);
}
```

### 四.编译hello.c文件成.so文件
###### 1.在hello.c同级目录下创建Android.mk文件，编写内容为：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE    := hello
LOCAL_SRC_FILES := hello.c

include $(BUILD_SHARED_LIBRARY)

```
###### 2.创建Application.mk文件（选择支持系统硬件环境），编写内容为：

```
# 1.支持所有系统硬件环境
APP_ABI := all
# 2.仅仅支持x86
APP_ABI := x86
# 3.支持x86和arm
APP_ABI := x86 armeabi
# 4.缺省值等价于APP_ABI := armeabi
```

###### 3.打开cmd命令窗口

```
1.切换到hello.c所在目录：
cd /d 全路径(hello.c)

2.使用ndk编译出.so 文件：ndk-build
```
### 五.将.so 文件copy到项目中
###### 1. .so文件所在目录为build.c文件的parent->parent下的->libs文件夹下的->armeabi文件夹下
###### 2.将.so文件copy到Android Studio项目的armeabi文件夹下

### 六.Java调用JNI，然后编译运行项目
###### 1.声明jni方法(用native修饰)

```
private native String helloFromJNI();
```

###### 2.调用jni方法前，加载.so文件

```
System.loadLibrary("hello");
```
###### 3.调用jni方法

```
textView.setText(helloFromJNI());
```



