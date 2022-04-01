---
title: Python3 requests增加自签名证书
date: 2022-04-01 10:10:47
tags: 
- python
- python3
- requests
---
 
对python3 requests包的网络请求抓包时，https请求会报错，错误如下：
```
[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed
```
这时需要增加抓包工具用到的自签名证书，步骤如下：
1. 终端执行命令`python3 -m certifi`，查看CA证书存放位置
2. 将抓包工具生成的自签名证书内容（pem格式）复制到上面提示的文件中
