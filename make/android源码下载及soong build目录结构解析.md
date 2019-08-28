# android源码下载及soong build目录结构解析 
## **写在前面**

> 在做android系统开发时，会涉及到底层**system,framework框架以及package**相关,如烧镜像版本**大小核**，ota升级，provision，settings等内容，完成这些模块功能的时候，少不了源码编译相关的知识

本次会利用空余时间，介绍完android源码编译相关内容；**先求一波关注，未完成部份会在两周内完成。**

**版权声明CopyRight：**

> **本内容作者：sunst，转载或引用请[标明出处](https://www.zhihu.com/people/qydq)，违者追究法律责任！！！**

**关于android源码编译相关的知识，会分为以下内容介绍，本篇介绍[AOSP（Android Open Source Project）](https://source.android.com)开放源代码下载到本地，以及对soong build系统源代码目录结构进行粗略的说明。**


**很多人不知道，其实```国内```通过如下网页可以在线浏览android源码。**  
  
[1.androidXref在线源代码查看](http://androidxref.com/)  
  
[2.android社区在线源代码查看](https://www.androidos.net.cn/android/6.0.1_r16/xref)  
  
>有了源代码以后，我们就可以对某些关键的**源代码进行分析，看看它们的实现原理**，比如像IntentService，HandlerThread，AMS，PMS；另一方面我们还可以修改源码，编译出属于自己**定制系统**；后面我会对这两方面内容进行整理，希望大家能点个赞关注我知乎主页啥的(滑稽笑)。
 
下载的目的一方面是为了方便我们阅读源码，嘛
 **对于可以科学上网的同学**
 
* [官网android代码库-需要科学上网](https://android.googlesource.com/)  
* [官网android源码目录elinux Master-android](https://elinux.org/Master-android)  

## 一：android 8.1.0 AOSP源码下载 
 这里是下载**android 8.1.0**的AOSP源代码，AOSP项目由不同的子项目组成，为了方便进行管理，Google采用Git对AOSP项目进行多仓库管理，我们可以用repo工具下载，但是这里我用另一种方式-编写python脚本去下载android源代码，这里大家也可以关注我的人工智能专栏，虽然最近忙android，ai这块放下了，但是有时间也会继续总结学习人工智能相关的内容的。
 
### 1.需要安装的文件  
  
首先安装msysgit:http://www.git-scm.com/download/  
安装python：http://www.python.org  
安装git:https://www.git.cn  
  
### 2.创建安装目录  

mkdir Android  
cd Android  
git clone git://mirrors.ustc.edu.cn/aosp/platform/manifest  
  
### 3.android版本操作  
  
cd manifest  
git tag (列出安卓版本号)  
git checkout android-5.1.1_r18 ##checkout你想下载的版本号  
  
#### Tips:  
>checkout之后，manifest/default.xml文件中记录的就是android5.1系统各个模块的路径  
  
### 4.编写python下载文件download.py 
```
import xml.dom.minidom  
import os  
from subprocess import call  
  
#downloaded source path  
rootdir = "C:/sun/users/sourcecode/manifest/android-8.1.0_r1"  
  
#git program path  
git = "C:/sun/library/git/Git/bin/git.exe"  
  
dom = xml.dom.minidom.parse("C:/sun/users/sourcecode/manifest/default.xml")  
root = dom.documentElement  
  
prefix = git + " clone https://android.googlesource.com/"  
suffix = ".git"  
  
if not os.path.exists(rootdir):  
os.mkdir(rootdir)  
  
for node in root.getElementsByTagName("project"):  
os.chdir(rootdir)  
d = node.getAttribute("path")  
last = d.rfind("/")  
if last != -1:  
d = rootdir + "/" + d[:last]  
if not os.path.exists(d):  
os.makedirs(d)  
os.chdir(d)  
cmd = prefix + node.getAttribute("name") + suffix  
call(cmd)  
```
### 5.执行python文件开始下载  

cmd  
 
python download.py  
  
#### 重要提示Tips:

>下载非常耗时，这个下载操作应该放到夜间进行。  

**多说一点使用repo来下载参考**  
  
https://blog.csdn.net/hunter___/article/details/80972878  
  
采用国内的镜像源进行下载.目前,可用的镜像源一般是科大和清华的,具体使用差不多,这里我选择清华大学镜像  
  
repo工具下载及安装  
  
通过执行以下命令实现repo工具的下载和安装  
mkdir ~/bin  
PATH=~/bin:$PATH  
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo  
chmod a+x ~/bin/repo  
  
## 二：Android 源码目录结构解析

下载源码以后，下载下来后我们也不能大眼瞪小眼，你不认它，它也不认识你，那岂不就白浪费时间下载了，这里对soong build目录结构进行一个简单的认识
### 1. 目录结构
>[abi](#1)  
[art](#2)  
[bionic](#3)  
[bootable](#4)  
[build](#5)  
[cts](#6)  
[dalvik](#7)  
[developers](#8)  
[development](#9)  
[device](#10)  
[docs](#11)  
[external](#12)  
[frameworks](#13)  
[hardware](#14)  
[libcore](#15)  
[libnativehelper](#16)  
[ndk](#17)  
[out](#18)  
[packages](#19)  
[pdk](#20)  
[platform_testing](#21)  
[prebuilts](#22)  
[sdk](#23)  
[system](#24)  
[tools](#25)  
[vendor](#26)

### 2. 目录结构具体
#### abi（Application Binary Interface）

具体请看[官方文档](https://developer.android.com/ndk/guides/abis)

#### art（Android runtime）

具体请看[ART and Dalvik](https://source.android.com/devices/tech/dalvik/)

#### bionic

一些基础库（以下只是列举了几个，并非全部）

> libm（library math）  
> libc（library c）：在 glibc 的基础上做了裁剪与修改的，为了规避GNU GPL等商业行为的约束  
> [libstdc++](https://developer.android.com/ndk/guides/cpp-support)（library standard C++）：并非完整版，只做了简单支持  
> linker：装载链接相关库

#### bootable

> recovery

bootable 下仅包含 recovery 此文件夹，其实就是启动 Android recovery 模式相关的代码

#### build

Android Build 系统，用来定制各种编译规则。主要由 makefile 组成。  
比如在编译时要执行的 `source build/envsetup.sh` 就位于 build 下。  
推荐一篇以前看过的介绍 Build 比较好的文章 [理解 Android Build 系统](https://www.ibm.com/developerworks/cn/opensource/os-cn-android-build/index.html)

#### cts（Compatibility Test Suite）

一个自动化测试工具 [CTS](https://source.android.com/compatibility/cts/)。确保 make 出来的系统没问题，注意如果要是修改了源码的话相关的 testcase 也是要修改的。

#### dalvik

dalvik 虚拟机，与 art 有千丝万缕的关系，具体也可以看 [ART and Dalvik](https://source.android.com/devices/tech/dalvik/)

#### developers

主要是一些可运行的 Android 示例项目，可以单独拉出来运行。

#### development

仍然是一些工具性的东西，全部子文件夹如下：

> apps  
> build  
> cmds  
> docs  
> host  
> ide  
> libraries  
> ndk  
> perftests  
> samples  
> scripts  
> sdk  
> sdk_overlay  
> sys-img  
> testrunner  
> tools  
> tutorials

-   apps 中包含了一些并没有在系统中部署的应用
-   ndk 中就是和 ndk 相关的东西
-   samples 中则是一些示例 app
-   像 MonkeyTest 相关代码位于 development/cmds/monkey 中
-   eclipse、emacs、intellij、xcode的配置信息位于 development/ide 中  
    emulator, simulator, and stuff for the NDK and SDK

#### device

包含不同品牌手机独有的设备信息，具体目录如下：

> asus  
> common  
> generic  
> google  
> htc  
> huawei  
> lge  
> moto  
> sample

#### docs

> source.android.com

仅包含此文件夹，该文件夹下相关文件就是生成 source.android.com 站点的具体素材及代码

#### external

一些开源的第三方组件，这里仅列了一下大家比较熟悉的如glide、junit、okhttp、sqlite 等

> aac  
> apache-http  
> bison  
> chromium-webview  
> easymock  
> glide  
> google-breakpad  
> google-fonts  
> jpeg  
> junit  
> lldb  
> llvm  
> ltrace  
> markdown  
> okhttp  
> opencv  
> proguard  
> protobuf  
> robolectric  
> scrypt  
> selinux  
> smali  
> sqlite  
> strace  
> tcpdump  
> valgrind  
> webrtc  
> zlib

#### frameworks

这就是 Android 中大家熟悉的 Frameworks，应用程序框架层啦，全部子文件夹如下：

> av  
> base  
> compile  
> data-binding  
> ex  
> mff  
> minikin  
> ml  
> multidex  
> native  
> opt  
> rs  
> support  
> volley  
> webview  
> wilhelm

-   Android support 包 com.android.support:support-v4、v7 等都位于 frameworks/support 文件夹下
-   webview 就位于 frameworks/webview 文件夹下
-   各种 Service，比如ActivityManagerService、SystemService、WindowManagerService、InputManagerService等就位于 frameworks/base 文件夹下
-   keystore、opengl 等也位于 frameworks/base 文件夹下

#### hardware

主要包含了 android [HAL](https://source.android.com/devices/architecture/hal)（硬件抽象层）相关代码。硬件抽象层介于 Linux内核驱动程序与 Android 系统之间。对 Linux 驱动进行了封装，使操作系统级别可以忽略底层实现的细节。

#### libcore

一些核心库

#### libnativehelper

JNI 相关的一些类

#### ndk

原生开发工具包

#### out

编译完后输出的所有相关文件都位于此文件夹下，包括生成的各种 img 就位于 out/target/product/hammerhead 下

#### packages

各种内置的 apk、ContentProvider、输入法、壁纸等，所有文件夹如下：

> apps  
> experimental  
> inputmethods  
> providers  
> screensavers  
> services  
> wallpapers

-   蓝牙、浏览器、相机、邮件、音乐、NFC 等都位于 packages/apps 下面
-   MediaProvider、DownloadProvider、MmsProvider等都位于 packages/providers 下
-   壁纸相关位于 packages/wallpapers 下

#### pdk（Platform Development Kit）

平台开发套件，仅包含了一些供硬件抽象层开发使用的必要组件，供一些 OEM 厂商用来适配及测试最新的Android 系统，加快第三方厂商的更新速度。  
加快OEM厂商的update速度

#### platform_testing

平台相关的一些测试用例

#### prebuilts

一些预构建成二进制的库 [prebuilts](https://developer.android.com/ndk/guides/prebuilts)  
其中关于 build 时 bison 问题的主角就位于 prebuilts/misc/darwin-x86 下的 bison。

#### sdk

看了下里边挺多被废弃的代码，所以我也吃不准这个文件夹的意义何在，所以暂时先不写了

#### system

Android 的部分系统源码及一些工具，主要是在各种 java 启动程序起来前的部分。工具比如 adb、fastboot、keystore 等，其他如 mkbootimg、init 进程等。

#### tools

工具，近包含 fat32lib 与 gradle，具体文件目录如下

> external
> 
> > fat32lib  
> > gradle

#### vendor

包含不同供应商的私有的二进制库，仅包含如下三个文件夹：

> broadcom  
> lge  
> qcom

## 参考资料

* [官网android代码库-需要科学上网](https://android.googlesource.com/)  
* [官网android源码目录elinux Master-android](https://elinux.org/Master-android)  

---

以上便是android源码下载及soong build目录结构解析的全部内容

请尊重劳动成果，注意文中**版权声明**，以上未完成未完善部份，可以[点击关注我知乎](https://www.zhihu.com/people/qydq/activities)留意，Android专栏不定时更新，也可以同时关注人工智能专栏，本内容作者`sunst`，有问题请沟通`qyddai@gmail.com`