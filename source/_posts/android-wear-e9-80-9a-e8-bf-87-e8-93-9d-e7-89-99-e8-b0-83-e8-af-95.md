---
title: Android wear通过蓝牙调试
url: 307.html
id: 307
categories:
  - Android
date: 2016-09-14 22:44:57
tags:
---

adb forward tcp:6666 localabstract:/adb-hub; adb connect 127.0.0.1:6666