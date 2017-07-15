title: MacOSX 10.10.3下 android-goldfish-3.4 内核下载与编译
date: 2015-08-20 21:51:32
categories: Android
tags: [Tech,Android, Linux Kernel]
---

在上一篇文章中,我们已经完成了`android-5.1.1_r9`源码的编译和下载工作.但android源码里是不包含`Linux Kernel`内核源码的,因此我们还需要手动下载内核源码并编译.本文参照[在Ubuntu上下载、编译和安装Android最新内核源代码（Linux Kernel）](http://blog.csdn.net/luoshengyang/article/details/6564592)这篇文章在MacOSX环境下进行.

# 源码下载

首先进入之前下载好的android源码根目录,新建kernel目录并进去,然后从[谷歌官网](https://android.googlesource.com/)拖源码.

	$ cd /Volumes/android
	$ mkdir kernel
	$ cd kernel
	$ git clone http://android.googlesource.com/kernel/goldfish.git

下载完成后,只有一个`goldfish`文件夹,里边什么都没有.这时候通过`git`命令看下当前的分支情况:

	$ git branch
	$ git branch -a
	
第一条命令可以看到,当前处于`* master`分支,第二条命令可以看到所有的分支情况:

```
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/android-3.10
  remotes/origin/android-3.4
  remotes/origin/android-goldfish-2.6.29
  remotes/origin/android-goldfish-3.10
  remotes/origin/android-goldfish-3.10-m-dev
  remotes/origin/android-goldfish-3.4
  remotes/origin/linux-goldfish-3.0-wip
  remotes/origin/master
```


这里注意,远程分支有几个,内核版本有`2.6.29/3.0/3.4/3.10`几个,在切换分支前,**必须确认你下载的Android源码及编译出来的镜像对应的linux内核版本.**本着一切使用最新版本的原则,我在这里犯了错,用了`3.10`版本的内核,发现了大量的问题.便以失败的沮丧,在网上找了半天也没找到解决方案.最后打开`emulator`,发现内核版本是`3.4`的,恍然大悟!最后切换到了`remotes/origin/android-goldfish-3.4`分支,编译成功了.

	$ git checkout remotes/origin/android-goldfish-3.4

由于是在`emulator`上运行,因此需要切换到`goldfish`分支上.最后进入`goldfish`文件夹就可以看到内核源码了

	$ cd goldfish
	$ ls -a
	
# 源码编译

编译内核源码需要android源码中的交叉编译工具,因此首先把这个工具导入到`PATH`变量中

	$ export PATH=/Volumes/android/prebuilts/gcc/darwin-x86/arm/arm-eabi-4.8/bin/:$PATH
	
接下来就可以开始编译了,当前目录为`/Volumes/android/kernel/goldfish`,执行:

	$ export ARCH=arm
	$ export SUBARCH=arm
	$ export CROSS_COMPILE=arm-eabi-
	$ make goldfish_defconfig
	$ make
	
当然,如果你一次就成功了,那真的恭喜,因为我并没有成功,提示如下错误,缺少`elf.h`文件

```
scripts/kconfig/conf --silentoldconfig Kconfig
  CHK     include/linux/version.h
  CHK     include/generated/utsrelease.h
  UPD     include/generated/utsrelease.h
make[1]: `include/generated/mach-types.h' is up to date.
  CALL    scripts/checksyscalls.sh
  HOSTCC  scripts/mod/mk_elfconfig
scripts/mod/mk_elfconfig.c:4:10: fatal error: 'elf.h' file not found
#include <elf.h>
         ^
1 error generated.
make[2]: *** [scripts/mod/mk_elfconfig] Error 1
make[1]: *** [scripts/mod] Error 2
make: *** [scripts] Error 2
```
第一次,全盘搜了下`elf.h`这个文件,发现只有`linux`目录下有,索性试试看看行不行,放到`/usr/include/`目录需要`sudo`权限:

	$ sudo cp -a /Volumes/android//kernel/goldfish/include/linux/elf.h /usr/include/

未果,不起作用.最后继续找啊找,找到如下[解决方案](http://stackoverflow.com/questions/19346626/building-goldfish-kernel-goldfish-armv7-defconfig-not-found-at-arch-x86-conf):

	$ make ARCH=arm CROSS_COMPILE=arm-eabi- goldfish_armv7_defconfig
	
也就是用`goldfish_armv7_defconfig`的配置代替`goldfish_defconfig`进行初始化.经过了N(>=50)个问题后(如果不会,按照默认的配置一路按enter就行),接着又出现了上面的一幕,也就是说`elf.h`这个文件是必不可少的.按照找这个文件的思路,找到了[这篇文章](http://blog.csdn.net/iwantcomputer/article/details/23889585). 

**这个问题是由于os x的系统include路径里面少了一个elf.h这个文件，到 `http://www.rockbox.org/tracker/9006?getfile=16683` 这个地方，把里面的内容保存成elf.h，放到内核源码的scripts/mod/下面即可。**

顺便看到了作者遇到的第二个与`elf.h`有关的问题,

**elf.h之前是因为在系统include库里面，现在放到了scripts/mod下面，编译.c文件的时候定义elf.h的时候有问题，想必会c的人都知道#include <elf.h>和#include "elf.h" 的区别吧。把scripts/mod下面的mk_elfconfig.c，modpost.h这两个文件，开头前几行的#include <elf.h>改成#include "elf.h"。**

因此,需要把以下两个文件的`<elf.h>`改为`"elf.h"`

	/Volumes/android/kernel/goldfish/scripts/mod/mk_elfconfig
	/Volumes/android/kernel/goldfish/scripts/mod/modpost

照着作者的思路,继续兴奋地进行第二次编译,最后终于看到了期待已久的`ready`

	OBJCOPY arch/arm/boot/zImage
	Kernel: arch/arm/boot/zImage is ready

内核镜像编译成功后,执行下面的命令就可以看到你自己编译的内核运行在android模拟器上了

	emulator -kernel /Volumes/android/kernel/goldfish/arch/arm/boot/zImage &

如果提示`command not found: emulator`的话,需要重新初始化模拟器运行环境,

	$ cd /Volumes/android
	$ . ./build/envsetup.sh
	$ lunch full-eng