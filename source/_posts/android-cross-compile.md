---
title: Android交叉编译curl
date: 2017-03-31 09:27:33
tags:
---

操作系统：ubuntu 14.04
一.安装需要工具

```shell
sudo apt-get install autoconf libtool libsysfs-dev
```
二.制作交叉编译工具链
1.下载NDK，并配置NDK环境变量为NDK的安装路径
2.根据NDK里docs文档里的standalone-toolchain.html来抽取交叉编译的环境。
3.配置SYSROOT环境变量： 
```shell
SYSROOT=$NDK/platforms/android-19/arch-arm //android-19是你的android开发版本所定
```
4.然后运行命令：
```shell
$NDK/build/tools/make-standalone-toolchain.sh --platform=android-19 --install-dir=/tmp/my-android-toolchain
```
/tmp/my-android-toolchain是你交叉编译环境的复制路径。最好别放在tmp目录里，因为重启机子就立即消失了。这个新生成的文件 夹即是你的交叉编译环境
5.配置PAHT和CC环境变量：
```shell
export PATH=/tmp/my-android-toolchain/bin:$PATH
export CC=arm-linux-androideabi-gcc
```
三.下载curl并编译
```shell
git clone https://github.com/curl/curl.git
cd curl
mkdir build
autoreconf -i
./configure --host=arm-linux --enable-cross-compile --enable-threaded-resolver --disable-shared --disable-ftp --disable-file --disable-ldap --disable-ldaps --disable-rtsp --disable-dict --disable-telnet --disable-tftp --disable-pop3 --disable-imap --disable-smtp --disable-gopher --disable-manual --enable-proxy --enable-ipv6 --enable-cookies --enable-symbol-hiding --disable-versioned-symbols --disable-soname-bump --disable-sspi --disable-ntlm-wb --prefix=`pwd`/build
make 
make install
```