# android源码下编译apk
目前在做android系统开发，涉及到底层```framwork```的ota升级，provision，settings等内容，完成这些模块功能的时候，少不了源码编译相关的知识，本次会利用工作空余时间，介绍android源码编译相关内容；其实关注过我[知乎或者专栏](https://www.zhihu.com/people/qydq/activities)的人，可以看到在上篇```结构化数据probuff使用]```和ota升级4篇内容中就已经涉及到android源码相关的知识点了，如下：
>* [结构化数据protobuff的使用。](https://zhuanlan.zhihu.com/p/76659624)
>* [(很重要-小团子也值得一看)如何用android:sharedUserId属性生成带有系统权限的apk?。](https://zhuanlan.zhihu.com/p/69259334)

关于android源码编译相关的知识，会分为以下内容介绍，本篇android源代码下编译apk。
>1.android源代码下载。
2.android源码编译。
3.android soong build系统目录结构解析。
4.编写Android.mk文件，mk文件简单语法。
5.android源码下编译apk。
6.android源码下编译项目生成带系统权限的apk。

## 1.进入android源码目录下的build下执行：
```
source envsetup.sh 
```
然后后继续在该路径下执行
```
lunch
```  
具体可以参考上篇android源码编译 (暂未完成，
  
## 2.编写完成工程，编译出签名后的APK文件  
  
## 3.编写Android.mk文件，放入工程目录下  
```  
LOCAL_PATH : = $(call my-dir)  
include $(CLEAR_VARS)  
# Module name should match apk name to be installed  
LOCAL_MODULE := LiFangFang//x  
LOCAL_MODULE_TAGS := optional  
# LOCAL_SRC_FILES := $(call all-java-files-under, src)  
LOCAL_SRC_FILES := $(LOCAL_MODULE).apk//x  
LOCAL_MODULE_CLASS := APPS//x
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)  
# LOCAL_PACKAGE_NAME := LiFangFang  
LOCAL_CERTIFICATE := platform  
LOCAL_MODULE_PATH := $(TARGET_OUT_DATA_APPS)//x
#LOCAL_MODULE_PATH := $(TARGET_OUT_VENDOR_APPS)//x  
#LOCAL_PRIVILEGED_MODULE := true//x  
# include $(BUILD_PACKAGE)  
include $(BUILD_PREBUILT)  
# Use the folloing include to make our test apk.  
include $(call all-makefiles-under,$(LOCAL_PATH))  
```  
说明：
>编译后会生成一个Lifangfang.apk文件，mk文件中带x注释的内容第一种形式的mk文件，把带x删除后是另一种形式的mk文件。

## 4.编译  
拷贝apk至packages/apps/下  
>~$ cp -rf LiFangFang.apk ~/android/packages/apps/LiFangFang/LiFangFang.apk  
>进入目录 ~/...../LiFangFang$ mm  
  
编译成功：会在目录下生成lifangfang.apk，具体
>out/target/product/product_name/system/app/LiFangFang.apk  
  
## 5.运行  
安装在机子上运行之。 adb install xxx/LiFangFang.apk 
 
---

以上便是android源代码下编译apk的全部内容，内容很少，但还算详细。

请尊重劳动成果，注意文中**版权声明**，以上未完成部份，可以[点击关注我知乎](https://www.zhihu.com/people/qydq/activities)留意，Android专栏不定时更新，也可以同时关注人工智能专栏，本内容作者`sunst`，有问题请沟通`qyddai@gmail.com`
