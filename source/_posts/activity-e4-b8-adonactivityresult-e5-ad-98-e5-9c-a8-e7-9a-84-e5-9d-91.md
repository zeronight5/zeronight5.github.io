---
title: Activity中onActivityResult存在的坑
url: 363.html
id: 363
categories:
  - Android
date: 2017-04-12 10:40:16
tags:
---

所有需要传递或接收的 Activity 不允许设置singleInstance，或只能设为标准模式，否则在部分手机上系统将在 startActivityForResult()后直接返回Activity.RESULT_CANCELED，但在某些手机是正常的。