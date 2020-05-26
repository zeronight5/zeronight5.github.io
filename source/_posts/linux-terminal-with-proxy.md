---
title: Linux下让指定程序走代理
date: 2017-03-31 09:33:33
tags:
---

1.proxychains-ng安装

```shell
git clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng/
./configure --prefix=/usr/local/proxychains --sysconfdir=/etc
sudo make 
sudo make install
sudo make install-config
```
2.proxychains-ng配置
```shell
sudo vim /etc/proxychains.conf
#将socks4 127.0.0.1 9095改为自己的代理端口
socks5  127.0.0.1 1080  //1080改为你自己的端口
sudo cp /usr/local/proxychains/bin/proxychains4 /usr/local/bin/proxy
```
3.proxychains-ng使用
```shell
proxy wget http://www.baidu.com
```