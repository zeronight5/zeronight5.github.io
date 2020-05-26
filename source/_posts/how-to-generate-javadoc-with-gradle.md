---
title: 如何用gradle生成javadoc
date: 2017-03-31 09:27:33
tags:
---

在模块的build.gradle中填入以下内容

```groovy
task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

javadoc {
    options {
        encoding "UTF-8"
        charSet 'UTF-8'
        author true
        version true
        splitIndex true
        noDeprecated true
        setMemberLevel(JavadocMemberLevel.PUBLIC)
        links "http://docs.oracle.com/javase/8/docs/api"
    }
}
```

下面还有一个可能更好使
```groovy
android.libraryVariants.all { variant ->
    task("generate${variant.name}Javadoc", type: Javadoc) {
        title = "$name $version API"
        source = variant.javaCompile.source
        ext.androidJar = "${android.sdkDirectory}/platforms/${android.compileSdkVersion}/android.jar"
        classpath = files(variant.javaCompile.classpath.files, ext.androidJar)
        options {
            links("http://docs.oracle.com/javase/8/docs/api/");
            linksOffline("http://d.android.com/reference", "${android.sdkDirectory}/docs/reference");
            encoding "UTF-8"
            charSet 'UTF-8'
            author true
            version true
            splitIndex true
            noDeprecated true
            setMemberLevel(JavadocMemberLevel.PUBLIC)
        }
        failOnError false
        exclude '**/BuildConfig.java'
        exclude '**/R.java'
    }
}
```