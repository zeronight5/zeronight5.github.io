---
title: pk8和x509.pem转换成keystore
url: 365.html
id: 365
categories:
  - Android
date: 2017-04-12 20:33:11
tags:
---

一、 在github上下载工具 https://github.com/getfatday/keytool-importkeypair 将工具在Linux环境下解压或者解压后Copy到Linux下，运行如下命令

    keytool-importkeypair -k android/debug.keystore -p android -pk8 android/platform.pk8 -cert android/platform.x509.pem -alias androiddebugkey
    

二、 1 把pk8转换成pk12格式

    openssl pkcs8 -in platform.pk8 -inform DER -outform PEM -out platform.priv.pem -nocrypt
    

2 生成pk12的密钥问文件

    openssl pkcs12 -export -in platform.x509.pem -inkey platform.priv.pem -out platform.pk12 -name androiddebugkey
    

3 生成keystore

    keytool -importkeystore -deststorepass android -destkeypass android -destkeystore debug.keystore -srckeystore platform.pk12 -srcstoretype PKCS12 -srcstorepass android -alias androiddebugkey