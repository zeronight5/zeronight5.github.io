---
title: 如何将非Android Studio项目用gradle编译
date: 2017-02-10 16:27:33
tags:
---

示例`build.gradle`文件如下。

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.1'
    }
}
apply plugin: 'com.android.application'

allprojects {
    repositories {
        //maven { url "https://jitpack.io" }
    }
}

dependencies {
    compile fileTree(include: '*.jar', dir: 'libs')
    //compile(name: 'MPChartLib-release', ext: 'aar')
    compile 'com.android.support:support-annotations:24.2.1'
}

repositories{
    flatDir {
        dirs 'libs'
    }
}

android {
    compileSdkVersion 23
    buildToolsVersion "24.0.0"

    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
        }

        // Move the tests to tests/java, tests/res, etc...
        instrumentTest.setRoot('tests')

        // Move the build types to build-types/<type>
        // For instance, build-types/debug/java, build-types/debug/AndroidManifest.xml, ...
        // This moves them out of them default location under src/<type>/... which would
        // conflict with src/ being used by the main source set.
        // Adding new build types or product flavors should be accompanied
        // by a similar customization.
        debug.setRoot('build-types/debug')
        release.setRoot('build-types/release')
    }
}
```

