---
layout: post
title:  "Android 编译问题&常见错误集"
date:   2018-12-06
desc: ""
keywords: "android,rom,jackserver,ubuntu"
categories: [Android]
tags: [android,rom,ubuntu]
icon: icon-mobile-device
---
# 搭建配置环境+下载android源码
[配置环境](https://source.android.com/setup/build/requirements)

注意事项，源码文件大概60个G，保持虚拟机有足够的磁盘空间，如果磁盘空间不够会报错，需要手动增加虚拟机内存，以VM上运行Ubuntu 为例，手动关闭虚拟机后，进入Settings-Hard Disk(SCSI) 调整虚拟机硬盘尺寸，调整完后开机并没有显示磁盘空间变大，是因为这些磁盘空间还未被分配,通过安装一个gparted 工具进行手动分配磁盘空间
```
sudo apt-get install gparted
```




# 常见错误集

### [错误一 jack server问题](https://forum.xda-developers.com/android/software/aosp-cm-los-how-to-fix-jack-server-t3575179)

```
[  0% 3/59149] Ensuring Jack server is installed and started
Jack server already installed in "/home/vanshu/.jack-server"
Launching Jack server java -XX:MaxJavaStackTraceDepth=-1 -Djava.io.tmpdir=/tmp -Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx2G -cp /home/vanshu/.jack-server/launcher.jar com.android.jack.launcher.ServerLauncher
[  0% 4/59149] //art/tools/cpp-define-generator:cpp-define-generator-data clang++ main.cc [linux]
ninja: build stopped: subcommand failed.
18:06:07 ninja failed with: exit status 1
```

### 错误二
#### 解决方法为在make -j4 命令之前执行

```sh
export LC_ALL=C
make -j4
```


```
[ 11% 7676/66826] //external/selinux/checkpolicy:checkpolicy lex policy_scan.l [linux]
FAILED: out/soong/.intermediates/external/selinux/checkpolicy/checkpolicy/linux_x86_64/gen/lex/external/selinux/checkpolicy/policy_scan.c 
prebuilts/misc/linux-x86/flex/flex-2.5.39 -oout/soong/.intermediates/external/selinux/checkpolicy/checkpolicy/linux_x86_64/gen/lex/external/selinux/checkpolicy/policy_scan.c external/selinux/checkpolicy/policy_scan.l
flex-2.5.39: loadlocale.c:130: _nl_intern_locale_data: Assertion `cnt < (sizeof (_nl_value_type_LC_TIME) / sizeof (_nl_value_type_LC_TIME[0]))' failed.
Aborted (core dumped)
[ 11% 7679/66826] //external/selinux/checkpolicy:checkpolicy clang policy_define.c [linux]
ninja: build stopped: subcommand failed.
03:50:56 ninja failed with: exit status 1

```


### [错误三 jackserver 内存不足到问题](https://stackoverflow.com/questions/35579646/android-source-code-compile-error-try-increasing-heap-size-with-java-option)

报错信息：
```
Android source code compile error: “Try increasing heap size with java option '-Xmx<size>'”
```

解决方法：
```
export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4g"
./prebuilts/sdk/tools/jack-admin kill-server
./prebuilts/sdk/tools/jack-admin start-server
```


### 问题4:如何查看当前源码下载的源码是什么版本代号
编译的时候从makefile的信息中确实可以看到，另外还可以从git(.repo/manifest.xml)中查询，或者到build/core/version_plaform.mk中去查询plaform_version的定义值

### 问题5:默认需要bash进行编译，如果是zsh会报错，从zsh切换回bash直接输入bash命令就可以切换

### 问题6:编译好后的版本使用fastboot flashall -w
编译好的img镜像会存储在out/target/product/bullhead
包括
```console
/out/target/product/bullhead$ ll -a *.img
-rw-rw-r-- 1 test test  12066816 Dec 26 01:54 boot.img
-rw-r--r-- 1 test test   5824660 Dec 26 01:49 cache.img
-rw-rw-r-- 1 test test   7013873 Dec 26 01:54 ramdisk-recovery.img
-rw-rw-r-- 1 test test   1234466 Dec 26 01:54 ramdisk.img
-rw-rw-r-- 1 test test  17846272 Dec 26 01:54 recovery.img
-rw-r--r-- 1 test test 936019604 Dec 26 02:00 system.img
-rw-r--r-- 1 test test 166260172 Dec 26 01:56 userdata.img
-rw-r--r-- 1 test test 189996644 Dec 26 01:49 vendor.img
```
输入： fastboot flashall -w
会报错需要设置,没有设置ANDROID_PRODUCT_OUT
```console
export ANDROID_PRODUCT_OUT=/home/test/android_8.1.0/out/target/product/bullhead
```


### 问题7:增加编译eng版本
* eng版本自带root版本
* user版本是product版本
* userdebug版本是默认没有root，但是可以通过adb开启root

aosp默认的lunch选项，多了一个aosp_angler-eng这个选项。 怎么增加lunch选项呢？因为我的是Nexus 5X 
```vim
vim /device/lge/bullhead/vendorsetup.sh
```
在其中加入下面这一行：
```
add_lunch_combo aosp_bullhead-eng
```
需要注意的是eng前面的不是下划线