## **写在前面**

> 在做android系统开发时，会涉及到底层**system,framework框架以及package**相关,如烧镜像版本**大小核**，ota升级，provision，settings等内容，完成这些模块功能的时候，少不了源码编译相关的知识

本次会利用空余时间，介绍完android源码编译相关内容；**先求一波关注，未完成部份会在两周内完成。**

**版权声明CopyRight：**

> **本内容作者：sunst，转载或引用请**[标明出处](https://www.zhihu.com/people/qydq)**，违者追究法律责任！！！**

**关于android源码编译相关的知识，会分为以下内容介绍，本篇android源码编译及烧录(mac或者Linux系统)。**
>[1.android](https://link.zhihu.com/?target=http%3A//1.android/) 8.1.0 AOSP源码下载。  
2.android soong build系统目录结构解析。  
3.android源码编译及烧录(mac或者Linux系统)。  
4.编写Android.mk文件(mk文件编写规则)。  
5.android源码下编译apk。  
6.android源码下编译项目生成带系统权限的apk。

首先说明的是，android源码编译的四个流程:1.```源码下载```;2.```构建编译环境```;3.```编译源码```;4.```运行```。这里**源码下载本神单独拎出去了(**如上思维导图1.AOSP源码下载)**就不再介绍**，本篇android源码编译及烧录(mac或者linux系统)将按照如下顺序详解。

 >- 一：构建编译环境
 >- 二：编译源码
 >1.android AOSP源码编译
 >2.Settings模块编译
 >3.sdk编译
 >- 三：烧录android系统
 >- 四：出现的错误集合

## 一：构建编译环境
在有源码的前提下，就可以构建编译环境了，在开始之前，我们先来看看一些编译要求:

### 1. 硬件要求:
64位的操作系统只能编译2.3.x以上的版本,如果你想要编译2.3.x以下的，那么需要32位的操作系统.  
磁盘空间越多越好，至少在100GB以上，如果你想要在是在虚拟机运行linux,那么至少需要16GB的RAM/swap.  

### 2. 软件要求:
_1. 操作系统要求_  
在[AOSP开源](https://android.googlesource.com/)中，主分支使用Ubuntu长期版本开发和测试的，因此也建议你使用Ubuntu进行编译，个人不推荐在虚拟机中编译2.3.x以上的代码
>window下装虚拟机怎么讲呢，实际上，对于学生，如果家里面条件好一点，能买mac还是买一台mbp做开发是最好的，16k虽然有点贵，不要像我一样只拿来看电影了就好（滑稽)。

不同版本的Ubuntu能够编译android版本也是相对应的，这里我推荐，我们只考虑android6.0以上的设备

>**Android版本**：Android 6.0至AOSP master  
>**编译时Ubuntu最低版本**：Ubuntu 14.04

_2. JDK版本要求_  
Ubuntu14.04默认没有安装jdk,本文AOSP代码将采用8.1.0，所以同样推荐，使用openjdk1.8，更具体的可以参看:[Google源码编译要求](https://source.android.com/source/requirements.html)，接下来介绍openjdk1.8的安装
>提示：如果你的是mac系统，那么可以忽略这一步。[（这里本神是参考这篇：click）]([https://www.jianshu.com/p/6fed8af8d11e](https://www.jianshu.com/p/6fed8af8d11e))，感谢🙏这位前辈。

**2.1 openjdk1.8的安装**

**第一步：添加ppa(可选)**

> sudo add-apt-repository ppa:openjdk-r/ppa

提示：对于需要采用openjdk7，需要先设置ppa
**第二步：安装openjdk1.8**
```
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```
**第三步：设置默认java和javac(可选)**

由于本人当前Ubuntu上没有安装其他版本的jdk，所以跳过此步骤
```
sudo update-alternatives --config java
sudo update-alternatives --config javac
```
**第四部：检查版本安装是否成功**
```
java -version
```
当前版本为
```
openjdk version "1.8.0_171"
OpenJDK Runtime Environment (build 1.8.0_171-8u171-b11-2~14.04-b11)
OpenJDK 64-Bit Server VM (build 25.171-b11, mixed mode)
```
安装成功。

**2.2. 然后安装所需的软件软件包**

> sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev libgl1-mesa-dev libxml2-utils xsltproc unzip

_3. 其他要求(可选)_

[Google官方构建编译环境指南](https://source.android.com/source/initializing.html)中已经说明了Ubuntu14.04，Ubuntu 12.04，Ubuntu 10.04需要添加的依赖，如下可能会用到
```
sudo apt-get install libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-dev g++-multilib
sudo apt-get install -y git flex bison gperf build-essential libncurses5-dev:i386
sudo apt-get install tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386
sudo apt-get install dpkg-dev libsdl1.2-dev libesd0-dev
sudo apt-get install git-core gnupg flex bison gperf build-essential
sudo apt-get install zip curl zlib1g-dev gcc-multilib g++-multilib
sudo apt-get install libc6-dev-i386 
sudo apt-get install lib32ncurses5-dev x11proto-core-dev libx11-dev
sudo apt-get install libgl1-mesa-dev libxml2-utils xsltproc unzip m4
sudo apt-get install lib32z-dev ccache
```

## 二：编译源码
### 1.android AOSP源码编译
  
(1)source build/envsetup.sh 初始化编译环境  
  
(2)lunch aosp_arm64-eng 选择编译目标  
lunch aosp_bullhead-userdebug  
  
所谓的编译目标就是生成的镜像要运行在什么样的设备上.这里我们设置的编译目标是aosp_arm64-eng  
  
编译目标格式说明  
编译目标的格式:BUILD-BUILDTYPE,比如上面的aosp_arm-eng的BUILD是aosp_arm,BUILDTYPE是eng.  
  
BUILD指的是特定功能的组合的特定名称,即表示编译出的镜像可以运行在什么环境.其中,aosp(Android Open Source Project)代表Android开源项目;arm表示系统是运行在arm架构的处理器上,arm64则是指64位arm架构;处理器,x86则表示x86架构的处理器;此外,还有一些单词代表了特定的Nexus设备,下面是常用的设备代码和编译目标,更多参考官方文档  
|受型号|设备代码|编译目标|  
|---|----|---|  
|Nexus 6P|angler|aosp_angler-userdebug|  
|Nexus 5X|bullhead|aosp_bullhead-userdebug|  
|Nexus 6|shamu|aosp_shamu-userdebug|  
|Nexus 5|hammerhead|aosp_hammerhead-userdebug|  
  
什么是BUILDTYPE  
  
BUILD TYPE则指的是编译类型,通常有三种:  
-user:代表这是编译出的系统镜像是可以用来正式发布到市场的版本,其权限是被限制的(如,没有root权限,不鞥年dedug等)  
-userdebug:在user版本的基础上开放了root权限和debug权限.  
-eng:代表engineer,也就是所谓的开发工程师的版本,拥有最大的权限(root等),此外还附带了许多debug工具  
  
  
(3)开始编译  
  
通过make指令进行代码编译,该指令通过-j参数来设置参与编译的线程数量,以提高编译速度.比如这里我们设置8个线程同时编译:  
  
make -j8  
  
cat /proc/cpuinfo 查看相关cpu信息  
  
(4)编译结果  
  
如果一切顺利的化,在几个小时之后,便可以编译完成.看到### make completed successfully (01:18:45(hh:mm:ss)) ###表示你编译成功了.  
### 2：模块编译  
  
除了通过make命令编译可以整个android源码外,Google也为我们提供了相应的命令来支持单独模块的编译.  
  
编译环境初始化(即执行source build/envsetup.sh)之后,我们可以得到一些有用的指令,除了上边用到的lunch,还有以下:  
  
- croot: Changes directory to the top of the tree.  
- m: Makes from the top of the tree.  
- mm: Builds all of the modules in the current directory.  
- mmm: Builds all of the modules in the supplied directories.  
- cgrep: Greps on all local C/C++ files.  
- jgrep: Greps on all local Java files.  
- resgrep: Greps on all local res/*.xml files.  
- godir: Go to the directory containing a file.  
其中mmm指令就是用来编译指定目录.通常来说,每个目录只包含一个模块.比如这里我们要编译Launcher2模块,执行指令:  
  
mmm packages/apps/Launcher2/  
  
  
稍等一会之后,如果提示:  
### make completed success fully ###  
即表示编译完成,此时在out/target/product/gereric/system/app就可以看到编译的Launcher2.apk文件了.  
  
重新打包系统镜像  
编译好指定模块后,如果我们想要将该模块对应的apk集成到系统镜像中,需要借助make snod指令重新打包系统镜像,这样我们新生成的system.img中就包含了刚才编译的Launcher2模块了.重启模拟器之后生效.  
  
  
  
单独安装模块  
我们在不断的修改某些模块,总不能每次编译完成后都要重新打包system.img,然后重启手机吧?有没有什么简单的方法呢?  
在编译完后,借助adb install命令直接将生成的apk文件安装到设备上即可,相比使用make snod,会节省很多事件.  
  
  
补充  
我们简单的来介绍out/target/product/generic/system目录下的常用目录:  
Android系统自带的apk文件都在out/target/product/generic/system/apk目录下;  
一些可执行文件(比如C编译的执行),放在out/target/product/generic/system/bin目录下;  
动态链接库放在out/target/product/generic/system/lib目录下;  
硬件抽象层文件都放在out/targer/product/generic/system/lib/hw目录下.  
  
  
### 3：SDK编译  
  
如果你需要自己编译SDK使用,很简单,只需要执行命令make sdk即可.  

## 三：烧录android系统
make完成后可以看到out目录下已经生成了对应的img文件，烧写android系统，进入img所在的目录，nexus5x的目录是out/target/product/bullhead/，可以使用如下命令进行烧写
```
adb reboot bootloader
fastboot flashall -w
```
提示Note：
>请确认adb devices和fastboot devices都能识别到设备

烧写完成后，手机正常开机，可见当前的build number就是我们预期的版本，如下图：


## 四：错误集合  
  
### 1.  如果是ubuntu系统，在安装的jdk时，可能遇到错误
```
**_You are attemping to build with the incorrect version_**
```
检查sdk，ubuntu，android系统版本，然后使用命令切换jdk版本，上面构建环境要求中已有说明，如果你认真看了，这个错误应该可以避免的
```
sudo update-alternative
```
### 2.  （heap size）， Out of memory error.具体错误如下:  
  
  
这个错误比较常见,尤其是在编译AOSP主线代码时,常常会因为JVM heap size太小而导致该错误.  
此时有两种解决方法:  
**方法一:**  
在编译命令之前,修改prebuilts/sdk/tools/jack-admin文件,找到文件中的这一行:  
JACK_SERVER_COMMAND="java -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -cp $LAUNCHER_JAR $LAUNCHER_NAME"  
然后在该行添加-Xmx4096m,如:  
JACK_SERVER_COMMAND="java -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -Xmx4096m -cp $LAUNCHER_JAR $LAUNCHER_NAME"  
然后再执行time make -8j  
  
**方法二:**  
在控制台执行以下命令:  
export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4096m"  
out/host/linux-x86/bin/jack-admin kill-server  
out/host/linux-x86/bin/jack-admin start-server  
  
执行完该命令后,再使用make命令继续编译.某些情况下,当你执行jack-admin kill-server时可能提示你命令不存在,此时去你去out/host/linux-x86/bin/目录下会发现不存在jack-admin文件.如果我是你,我就会重新repo sync下,然后从头来过.  
  
### 3.使用emulator时，虚拟机停在黑屏界面,点击无任何响应.
此时，可能是kerner内核问题,解决方法，执行如下命令:  
  ```
./out/host/linux-x86/bin/emulator -partition-size 1024 -kernel ./prebuilts/qemu-kernel/arm/kernel-qemu-armv7  
```
通过使用kernel-qemu-armv7内核 解决模拟器等待黑屏问题.而-partition-size 1024 则是解决警告: system partion siez adjusted to match image file (163 MB >66 MB)  
  
如果你一开始编译的版本是aosp_arm-eng,使用上述命令仍然不能解决等待黑屏问题时,不妨编译aosp_arm64-eng试试.  

---
以上便是android源代码下编译apk的全部内容，内容很少，但还算详细。

请尊重劳动成果，注意文中**版权声明**，以上未完成部份，**可以[点击关注我知乎](https://www.zhihu.com/people/qydq/activities)留意(这里求一波关注，你们的关注是我前进的东西)，**Android专栏不定时更新，也可以同时关注[人工智能专栏](https://zhuanlan.zhihu.com/sstai)，本内容作者**`sunst`**，有问题请沟通`qyddai@gmail.com`

> 作者：sunst 2019-08-12 11:32