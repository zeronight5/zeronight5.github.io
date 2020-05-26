---
title: 关于Android CPU架构产生的应用运行异常问题提示
url: 376.html
id: 376
categories:
  - Android
  - 小提示
date: 2017-05-08 11:11:48
tags:
---

Android开发中，不可避免的可能会引用到外部so文件，设置一个项目中可能需要引用多个不同的外部so文件。因为不同的引入库中so文件的目录可能不同，导致打包后生成的项目lib目录中的目录结构是不同的外部so文件目录的合集。可能会出现armeabi/armeabi-v7a/arm64-v8a/x86/mips等，一般情况下，armeabi应该是有的，当此三个目录下的文件可能不同时，在某些特定机型下很可能会出现**UnsatisfiedLinkError**。

原因在于不同的机型CPU结构不同导致搜寻不同的目录下面的包，而由于外部库不同的so文件目录可能armeabi下还有a、b so文件，而x86下可能只含有a。例如一个应用armeabi下有`liba.so`、`libb.so`，arm64-v8a下有`liba.so`这样的应用装在arm64的设备上，当代码掉到加载libb.so时就会有上面的异常出现，即便是armeabi下有此so，系统也不会去寻找，原因是这个apk中已经有arm64的目录了。

此时解决方案如下：

在build.gradle中添加如下内容

        android {
            defaultConfig {
                ndk {
                    abiFilters 'armeabi'
                }
            }
        }
    

这样生成的apk中就只包含armeabi下的so文件，而不包含其他架构的目录，这样系统会采取兼容模式，加载armeabi下的so文件。