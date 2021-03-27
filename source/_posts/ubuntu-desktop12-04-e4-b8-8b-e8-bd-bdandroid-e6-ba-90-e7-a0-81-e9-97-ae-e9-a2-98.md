---
title: ubuntu desktop12.04 下载android 源码问题
url: 127.html
id: 127
categories:
  - Ubuntu
date: 2014-05-12 10:10:47
tags:
---

在VMware中安装了ubuntu desktop 12.04进行下载android源码。后来出现卡屏以及重启后进不去桌面问题。 1.卡屏问题原因出在VMware设置里，将显示器的3d加速选项去掉。 2.进不去桌面问题解决方案：在出现ubuntu图样之前按Ctrl+Alt+F2进入tty，重装xinit，然后输入startx命令