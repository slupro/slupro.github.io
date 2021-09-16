---
layout: post
title:  "在 WSL 和 CentOS 7 中编译openjdk-17-ga"
subtitle:   "build-openjdk-17-ga"
date:   2021-09-16 13:00:00 +10:00
author: "Steven Lu @slupro"
categories: [学习笔记]  # up to 2.
tags: [java]  # TAG names should always be lowercase, 0 to infinity.
---

昨天 openjdk 发布了 17GA，折腾了会儿在 Windows(WSL) 和 CentOS 7.9 上都编译出来了。记录一下。

### Windows(WSL)

Windows下可以用Cygwin或者WSL编译openjdk，因为我已经装了WSL了，所以就用WSL来编译。

下载完 openjdk-17-ga 的zip文件，解压，运行 wsl，并进入解压后的目录（比如我的是/mnt/d/GitHub_Repo/jdk-jdk-17-ga）。使用下面的步骤安装：

* 首先需要安装编译所需要的一些依赖文件，有些是我以前装的包，不确定是不是 openjdk 的编译也需要，我都包括进来了。

```
sudo apt install gcc autoconf make zip unzip build-essential libx11-dev libxext-dev libxrender-dev libxtst-dev libxt-dev libcups2-dev libfreetype6-dev libasound2-dev ccache gawk m4 libasound2-dev  libxrender-dev xorg-dev xutils-dev binutils libmotif-dev ant mercurial libc6-dev
```

* 安装 boot JDK。需要装一个比当前版本低的jdk，我这里装的是openjdk-16。
  
```
sudo apt install openjdk-16-jdk
```

* 运行 configure

```
chmod u+x configure
./configure  --build=x86_64-unknown-linux-gnu --host=x86_64-unknown-linux-gnu
```

这一步如果出错，可以查看详细的出错信息。成功后会出现：

```
====================================================
A new configuration has been successfully created in
/mnt/d/GitHub_Repo/jdk-jdk-17-ga/build/linux-x86_64-server-release
using configure arguments '--build=x86_64-unknown-linux-gnu --host=x86_64-unknown-linux-gnu'.

Configuration summary:
* Name:           linux-x86_64-server-release
* Debug level:    release
* HS debug level: product
* JVM variants:   server
* JVM features:   server: 'cds compiler1 compiler2 epsilongc g1gc jfr jni-check jvmci jvmti management nmt parallelgc serialgc services shenandoahgc vm-structs zgc'
* OpenJDK target: OS: linux, CPU architecture: x86, address length: 64
* Version string: 17-internal+0-adhoc.slupro.jdk-jdk-17-ga (17-internal)

Tools summary:
* Boot JDK:       openjdk version "16.0.1" 2021-04-20 OpenJDK Runtime Environment (build 16.0.1+9-Ubuntu-120.04) OpenJDK 64-Bit Server VM (build 16.0.1+9-Ubuntu-120.04, mixed mode, sharing) (at /usr/lib/jvm/java-16-openjdk-amd64)
* Toolchain:      gcc (GNU Compiler Collection)
* C Compiler:     Version 9.3.0 (at /usr/bin/gcc)
* C++ Compiler:   Version 9.3.0 (at /usr/bin/g++)

Build performance summary:
* Cores to use:   12
* Memory limit:   51360 MB

WARNING: Your build output directory is not on a local disk.
This will severely degrade build performance!
It is recommended that you create an output directory on a local disk,
and run the configure script again from that directory.
```

* 运行make

```
make images
```

成功后会出现```Finished building target 'images' in configuration 'linux-x86_64-server-release'```。

* 检查是否编译成功，可以看到openjdk的版本是17-ga了。

```
slupro@DESKTOP-96TK3JA:/mnt/d/GitHub_Repo/jdk-jdk-17-ga$ ./build/linux-x86_64-server-release/images/jdk/bin/java -version
openjdk version "17-internal" 2021-09-14
OpenJDK Runtime Environment (build 17-internal+0-adhoc.slupro.jdk-jdk-17-ga)
OpenJDK 64-Bit Server VM (build 17-internal+0-adhoc.slupro.jdk-jdk-17-ga, mixed mode, sharing)
```

### CentOS 7.9

CentOS在安装时由于组件和包都可选，每个人装的可能不一样，只列我的安装步骤了：

* 安装常用的开发组件和一些依赖文件

```
sudo yum groupinstall "Development Tools"
sudo yum install libXtst-devel libXt-devel libXrender-devel libXrandr-devel libXi-devel freetype-devel cups-devel alsa-lib-devel libffi-devel fontconfig-devel
```

* 安装boot JDK

```
sudo yum install java-16-openjdk-devel
```

* 运行 configure

```
chmod u+x configure
./configure  --build=x86_64-unknown-linux-gnu --host=x86_64-unknown-linux-gnu
```

这一步如果出错，可以查看详细的出错信息。成功后会出现：

```
====================================================
A new configuration has been successfully created in
/home/slupro/soft/jdk-jdk-17-ga/build/linux-x86_64-server-release
using default settings.

Configuration summary:
* Name:           linux-x86_64-server-release
* Debug level:    release
* HS debug level: product
* JVM variants:   server
* JVM features:   server: 'cds compiler1 compiler2 epsilongc g1gc jfr jni-check jvmci jvmti management nmt parallelgc serialgc services shenandoahgc vm-structs zgc' 
* OpenJDK target: OS: linux, CPU architecture: x86, address length: 64
* Version string: 17-internal+0-adhoc.slupro.jdk-jdk-17-ga (17-internal)

Tools summary:
* Boot JDK:       openjdk version "16.0.1" 2021-04-20 OpenJDK Runtime Environment 21.3 (build 16.0.1+9) OpenJDK 64-Bit Server VM 21.3 (build 16.0.1+9, mixed mode, sharing) (at /usr/lib/jvm/java-16-openjdk-16.0.1.0.9-3.rolling.el7.x86_64)
* Toolchain:      gcc (GNU Compiler Collection)
* C Compiler:     Version 4.8.5 (at /usr/bin/gcc)
* C++ Compiler:   Version 4.8.5 (at /usr/bin/g++)

Build performance summary:
* Cores to use:   4
* Memory limit:   15866 MB

```

* 运行make

```
make images
```

我在这里遇到了一个错误：

```
=== Output from failing command(s) repeated here ===
* For target hotspot_variant-server_libjvm_objs_precompiled_precompiled.hpp.gch:
gcc: error: unrecognized command line option '-std=c++14'
```

写了一个简单的C++文件，用 gcc 带参数 -std=c++14 运行，发现不支持这个参数。搜索了一下发现自带的gcc版本是4.8.5，不支持这个写法，于是把gcc升级到新的版本7.3.1。

```
sudo yum install centos-release-scl
sudo yum install devtoolset-7-gcc*
scl enable devtoolset-7 bash
gcc --version
```

注意升级完gcc后，需要重新运行```./configure```配置，然后再```make images```即可成功。