#### 一、本地方法名出现_，则.c文件中的方法名为_1
#### 二、特别复杂的本地名需要使用生成头文件方法
###### 1.打开cmd窗体，cd /d C:\chenlong\game\Git\MVP_Design\jnistudy\src\main\java
-------------注意：切换到src\main\java文件夹下
###### 2.将java文件生编译成头文件(包含native方法的转换命名):
- 1.简便格式（AS项目中有时不能用）
javah cn.yjjc.com.jnistudy.MainActivity
-------------注意：右键MainActivity--选择Copy Reference即可复制全类名

- 2.记住通用格式——
在工程的bin目录下，输入命令：
1. javah -classpath . -jni 类路径.JNI类
2. javah -classpath . -jni cn.yjjc.com.jnistudy.MainActivity
