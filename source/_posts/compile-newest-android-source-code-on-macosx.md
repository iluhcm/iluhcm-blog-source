title: Mac OS X 10.10.3下android-5.1.1_r9 源码下载与编译
date: 2015-08-18 10:36:21
categories: Android
tags: [Tech,Android]
---
# Thinking

最近刚买了Mac,趁着新鲜感还在,多学习点东西,对自己的职业发展是有好处的.在公司里实习的这段时间,几乎每周都有经验分享,大多数都涉及*Android Framework*层的知识,鉴于编写*Android Appilication*也有一段时间了,是时候开始着手从源码上提高自己的水平了.于是有了这篇文章.

# Preparing

首先还是从[android官网](https://source.android.com/source/initializing.html#setting-up-a-mac-os-x-build-environment)入手,看看准备工作有哪些.


## 创建磁盘文件

由于Mac OS X系统在默认安装时,文件类型是大小写保留的(但不是大小写敏感),这种类型可能会造成git工作不正常,因此最好创建一块新磁盘,并将文件类型设置为大小写敏感的.

创建磁盘有两种方式:

LaunchPad->Disk Utility->New Image,如图所示.

![Disk Utility Screenshot](http://7xl6ic.com1.z0.glb.clouddn.com/blog_disk_utility.jpeg)

打开Terminal,输入如下命令:

	$ hdiutil create -type SPARSE -fs 'Case-sensitive Journaled HFS+' -size 50g ~/android.dmg


需要注意的地方:

* 如果编译的是和我一样的版本,则磁盘空间最好超过50GB,因为我编译完成后的总大小是41GB(不包括`.repo`文件夹);
* 文件各种一定要选择Case-sensitive的;
* 如果在编译过程中遇到磁盘不够用的情况,可以先将android镜像umount,然后通过下面的命令行更改磁盘大小,最后再mount上.

	`$ hdiutil resize -size <new-size-you-want>g ~/android.dmg`


* android官方提供有**mount**和**umount**的function,将它们paste到'**~/.bash_profile**'(没有的话就touch一个),如下:

```
# mount the android file image
function mountAndroid() { hdiutil attach ~/android.dmg -mountpoint /Volumes/android; }

# unmount the android file image
function umountAndroid() { hdiutil detach /Volumes/android; }
```

最后再Terminal里输入以下命令更新启动文件.

	$ source ~/.bash_profile

## 安装JDK

android的编译不支持JDK8.0的编译环境,而我的机子装的都是最新版的JDK,因此只能再去官网下载JDK7.0并安装.官方提供的版本是[jdk-7u71-macosx-x64.dmg](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html#jdk-7u71-oth-JPR).

安装完成后,可以到`/Library/Java/JavaVirtualMachines`目录下查看目前安装的所有JDK版本.默认安装完成后,并不会自动地切换版本,需要手动进行.我参考了[How to switch JDK version on Mac OS X](http://www.jayway.com/2014/01/15/how-to-switch-jdk-version-on-mac-os-x-maverick/)这篇文章,在启动文件里设置了function,需要切换的时候直接输入下面的命令就可以了.

	$ setjdk 1.* (*代表版本号,6/7/8,前提是你安装了相应的版本)

## 安装Xcode

前面说了,我机器上的所有软件都是官方最新的版本,Xcode也不例外,默认安装的Xcode版本是6.4,其中Command Line Tools的版本是6.4, Mac SDK版本是10.10/10.9.第一次编译Android的过程中,出现了如下问题:

```
...
including ./system/extras/Android.mk ...
including ./system/keymaster/Android.mk ...
including ./system/media/audio_route/Android.mk ...
including ./system/media/audio_utils/Android.mk ...
including ./system/media/camera/src/Android.mk ...
including ./system/media/camera/tests/Android.mk ...
including ./system/netd/client/Android.mk ...
including ./system/netd/server/Android.mk ...
including ./system/security/keystore-engine/Android.mk ...
including ./system/security/keystore/Android.mk ...
including ./system/security/softkeymaster/Android.mk ...
including ./system/vold/Android.mk ...
including ./tools/external/fat32lib/Android.mk ...
host C++: libnativehelper_32 <= libnativehelper/JNIHelp.cpp
libnativehelper/JNIHelp.cpp:28:10: fatal error: 'string' file not found
#include <string>
         ^
1 error generated.
make: *** [out/host/darwin-x86/obj32/SHARED_LIBRARIES/libnativehelper_intermediates/JNIHelp.o] Error 1

#### make failed to build some targets (08:25 (mm:ss)) ####
```

Google了一番,发现遇到这个问题的人少得可怜...不过在GoogleGroup上有人讨论了这个问题:[libnativehelper/JNIHelp.cpp:28:10: fatal error: 'string' file not found](https://groups.google.com/forum/#!topic/android-building/tauCHs4QJJE),解决方案就是把Xcode降级到5.1.1,使用Mac SDK 10.8及Command Line Tools 5.1.1来编译.

随后就默默地去苹果的开发者论坛下载了[Xcode_5.1.1.dmg](http://adcdownload.apple.com/Developer_Tools/xcode_5.1.1/xcode_5.1.1.dmg),下载需要登陆开发者账号(不需要付费).下载并安装完成后,把Command Line Tools设置成了Xcode 5.1.1 (5B1008).

还有一种简便的做法是直接下载Mac SDK 10.8,然后将这个包放到`/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs`.

## 安装MacPorts
MacPorts和apt-get,yum,homebrew等软件一样,用于快捷安装应用程序.
访问[官方网站](http://www.macports.org/install.php)，这里提供有dmg安装和源码安装两种方式，dmg就多说了，下载[MacPorts-2.3.3-10.10-Yosemite.pkg](https://distfiles.macports.org/MacPorts/MacPorts-2.3.3-10.10-Yosemite.pkg)，下一步下一步安装即可。安装MacPorts时间比较长,可能和国内的网络有一定关系...耐心等待即可.

安装完成后,将其添加到PATH路径里及启动文件里,打开`~/.bash_profile`,添加:

	$ export PATH=/opt/local/bin:$PATH

最后使启动生效:

	$ source ~/.bash_profile

一切准备就绪后,通过MacPorts安装make,git和GPG:

	$ POSIXLY_CORRECT=1 sudo port install gmake libsdl git gnupg

## 解除文件限制

Mac系统下默认只能同时打开1024个文件,而在进行Android源码编译时有可能会超出这个限制,因此需要解除这个限制.方法很简单,在`~/.bash_profile`里添加一行:

	# set the number of open files to be 1024  
	ulimit -S -n 1024


# Getting the Source Code

以上一切准备就绪后,就可以开始源码的下载工作了.下载源码之前,需要同步repo,同时获取最新的android源码分支版本.

## 安装repo

Repo是一个辅助于Git管理Android版本及分支的工具.在安装repo前,需要新建一个文件夹`~/bin(名字可随意定)`并把这个文件夹放到PATH环境变量里,然后我们就可以把repo下载到这个文件夹里.

	$ mkdir ~/bin
	$ PATH=~/bin:$PATH
	$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo (执行此条命令需要FQ~)
	$ chmod a+x ~/bin/repo

repo这个工具很小(24KB),但却是下载整个源码不可缺少的工具.下载完成后,建立一个放置android源码的目录,

	$ mkdir WORKING_DIRECTORY
	$ cd WORKING_DIRECTORY

然后通过`repo init`将分支信息拉到本地:

	$ repo init -u https://android.googlesource.com/platform/manifest

当然如果有其他国内的镜像的话可以用国内镜像代替,可参考[同步、更新、下载Android Source & SDK from 国内镜像站](http://www.cnblogs.com/act262/p/4179093.html)这篇文章.

初始化的分支目录大小约为1.87MiB,之后会得到类似下面这样的目录:

```
...
 * [new tag]         android-5.0.1_r1 -> android-5.0.1_r1
 * [new tag]         android-5.0.2_r1 -> android-5.0.2_r1
 * [new tag]         android-5.0.2_r3 -> android-5.0.2_r3
 * [new tag]         android-5.1.0_r1 -> android-5.1.0_r1
 * [new tag]         android-5.1.0_r3 -> android-5.1.0_r3
 * [new tag]         android-5.1.0_r4 -> android-5.1.0_r4
 * [new tag]         android-5.1.0_r5 -> android-5.1.0_r5
 * [new tag]         android-5.1.1_r1 -> android-5.1.1_r1
 * [new tag]         android-5.1.1_r10 -> android-5.1.1_r10
 * [new tag]         android-5.1.1_r12 -> android-5.1.1_r12
 * [new tag]         android-5.1.1_r13 -> android-5.1.1_r13
 * [new tag]         android-5.1.1_r2 -> android-5.1.1_r2
 * [new tag]         android-5.1.1_r3 -> android-5.1.1_r3
 * [new tag]         android-5.1.1_r4 -> android-5.1.1_r4
 * [new tag]         android-5.1.1_r5 -> android-5.1.1_r5
 * [new tag]         android-5.1.1_r6 -> android-5.1.1_r6
 * [new tag]         android-5.1.1_r7 -> android-5.1.1_r7
 * [new tag]         android-5.1.1_r8 -> android-5.1.1_r8
 * [new tag]         android-5.1.1_r9 -> android-5.1.1_r9
... 
```

选择你想下载的android源码版本(我选择了android-5.1.1_r9),然后输入下面的命令进一步初始化该分支信息

	$ repo init -u https://android.googlesource.com/platform/manifest -b android-5.1.1_r9

初始化工作完成后,下一步就是下载源码了.

## 下载Android源码

为了将Android源码下载到本地,通过代码同步当前的repo:

	$ repo sync
	
下载源码的时间特别漫长,中途可能会发生断开连接的现象,不过并不要紧,此同步支持断点继传,但麻烦的是不知道什么时候会断开,为了解决这个问题,可以使用下面的shell来开启同步:

```
#!/bin/bash 
   #FileName  get_android.sh
   PATH=~/bin:$PATH 
   repo init -u https://android.googlesource.com/platform/manifest -b android-5.1.1_r9
   repo sync 
   while [ $? = 1 ]; do 
   echo "================sync failed, re-sync again =====" 
   sleep 3 
   repo sync 
   done
```

将上面的代码保存成`get_android.sh`,放在下载的源码的根目录下,开始下载:

	$ ./get_android.sh 
	
下载完成android5.1.1-r9版本的repo后,会发现占用最大空间的不是源码,而是`.repo`文件夹...`.repo`文件夹占用17GB,源码占用10GB.然后源码编译跟`.repo`文件夹并没有什么关系,在空间有限的Mac系统里,不得不珍惜每一寸土地,因此可以大方地

	$ rm -rf .repo/
	
最后保留10G的源码即可.接下来,挂载刚才创建的磁盘,并将源码移到磁盘目录下,

	$ cd ~/bin/WORKING_DIRECTORY/
	$ mountAndroid
	$ mv ~/bin/WORKING_DIRECTORY/* /Volumes/android/

# Compiling the Source Code

源码下载完成后,还需要做一些编译源码前的准备工作.我的编译环境如下表所示:

| Mac OS X系统| Mac OS X SDK	| git 		| GnuPG | GNU Make |  JDK   | Command Line Tools |
|:----------:	|:-------------:	| :------:|:-----:|:--------:|:------:| :------:| 
| 10.10.4   	| 		10.8 		| 2.5.0 	|1.4.19 | 3.81     |1.7.0_79|  5.1.1(5B1008)

**编译环境过新很有可能导致编译失败.**例如,我之前用的编译环境是MacOSX 10.10, JDK 1.8, Command Line Tools 6.4, 会出现上面提到的`fatal error: 'string' file not found`的错误.为了修正这个错误,需要手动去调整Mac SDK版本.首先进入下面的目录查看是否安装了`MacOSX10.8.sdk`

	/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs

如果已经安装了,那就找到下面的文件,

	/Volumes/android/build/core/combo/HOST_darwin-x86_64.mk

手动更改`-DMACOSX_DEPLOYMENT_TARGET=`**10.8**,然后保存退出.

## 初始化

编译源码前需要初始化编译环境,进入源码根目录`/Volumes/android/`,执行
	
	$ source build/envsetup.sh
	
得到下面的提示:

```
including device/asus/deb/vendorsetup.sh
including device/asus/flo/vendorsetup.sh
including device/asus/fugu/vendorsetup.sh
including device/asus/grouper/vendorsetup.sh
including device/asus/tilapia/vendorsetup.sh
including device/generic/mini-emulator-arm64/vendorsetup.sh
including device/generic/mini-emulator-armv7-a-neon/vendorsetup.sh
including device/generic/mini-emulator-mips/vendorsetup.sh
including device/generic/mini-emulator-x86/vendorsetup.sh
including device/generic/mini-emulator-x86_64/vendorsetup.sh
including device/htc/flounder/vendorsetup.sh
including device/lge/hammerhead/vendorsetup.sh
including device/lge/mako/vendorsetup.sh
including device/moto/shamu/vendorsetup.sh
including device/samsung/manta/vendorsetup.sh
including sdk/bash_completion/adb.bash
```
接下来选择编译目标,其中`full`这个名字可以随意定,后面的`-eng`表示附带了调试工具的开发者配置模式:

	$ lunch full-eng
	
出现以下类似信息:

```
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=5.1.1
TARGET_PRODUCT=full
TARGET_BUILD_VARIANT=eng
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
TARGET_ARCH_VARIANT=armv7-a
TARGET_CPU_VARIANT=generic
TARGET_2ND_ARCH=
TARGET_2ND_ARCH_VARIANT=
TARGET_2ND_CPU_VARIANT=
HOST_ARCH=x86_64
HOST_OS=darwin
HOST_OS_EXTRA=Darwin-14.3.0-x86_64-i386-64bit
HOST_BUILD_TYPE=release
BUILD_ID=LMY48I
OUT_DIR=out
============================================
```
接下来就可以编译了:

	$ make -jN

其中N代表同时进行的任务数.官方建议任务数设置为线程数的`1~2`倍,比如我的机器是单CPU,四核,8线程,则最快的构建任务数是`8~16`.

编译时间根据机器的性能不同而存在很大差异,在我的机子`MacBook Pro (2.2 GHz Intel Core i7/16 GB 1600 MHz DDR3)`上编译了一个半小时.如果第一次编译失败,程序会接着在编译失败的地方继续编译,而不会从头再来.我第一次编译的时候硬盘空间不够,后来重新调整了容量再继续编译,只花了几分钟就编译完了.如果出现:

	**make completed successfully (xx:xx (mm:ss))**

那么恭喜,编译成功了.

# Running

由于没有真机,只能使用模拟器调试,因此输入下面的命令会自动打开模拟器:

	$ emulator
	
至此整个android源码编译过程就完成了.

