##### 1.java.lang.UnsatisfiedLinkError: Native method not found
- 本地方法没有找到
- 1.本地函数名写错
- 2.忘记加载so库，没有调用System.loadLibrary
##### 2.findLibrary return null
- 1.加载动态链接库时，动态链接库的名字写错了
- 2.平台类型错误，把只支持arm平台的.so文件部署到x86的设备上(修改Android.mk中的APP_ABI)