---
title: Trojan使用
date: 2020-05-25 19:10:33
tags:
---

### Trojan简介

**trojan**是近两年兴起的网络工具，项目官网[https://github.com/trojan-gfw](https://github.com/trojan-gfw) ，文档官网是 [https://trojan-gfw.github.io/trojan](https://trojan-gfw.github.io/trojan) 。与强调加密和混淆的SS/SSR等工具不同，trojan将通信流量伪装成互联网上最常见的https流量，从而有效防止流量被检测和干扰。

### 安装Trojan

终端输入如下命令：

```shell
sudo bash -c "$(curl -fsSL https://raw.githubusercontent.com/trojan-gfw/trojan-quickstart/master/trojan-quickstart.sh)"
```

该命令会下载最新版的trojan并安装。安装完毕后，trojan配置文件路径是 `/usr/local/etc/trojan/config.json`

重点关注以下属性：

1.  `local_port`：监听的端口，默认是443，下文中因为要将Trojan放在Nginx后，因此改为了别的端口；
2.  `remote_addr`和`remote_port`：非trojan协议时，将请求转发处理的地址和端口。可以是任意有效的ip/域名和端口号，默认是本机和80端口。一般是伪装的web站点服务；
3. `password`：密码。需要几个密码就填几行，最后一行结尾不能有逗号；
4. `cert`和`key`：域名的证书和密钥；
5. `key_password`：默认没有密码（如果证书文件有密码就要填上）

根据自己的需求修改配置文件，保存，然后设置开机启动：`systemctl enable trojan`，并启动trojan： `systemctl start trojan`。

### 配合nginx

Trojan 默认端口为443，接管了所有443端口的流量，其他请求则是通过Trojan转发到了其他服务，例如nginx。如果有需求是将Trojan放在nginx后面，由nginx统一接收流量再分发到Trojan和其他服务（web，v2ray），可以参考下面的配置。

要实现上面的需求，用到了nginx的stream_ssl_preread_module模块，具体资料可以参考 [http://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html](http://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html) 。
编译nginx时增加此模块。

1.将Trojan配置文件的`local_port`改为非`443`端口，例如`9443`。将nginx原有`443`端口的配置改为非`443`端口，例如`8443`。
2.`nginx.conf`中添加如下配置
```nginx
stream {
    map $ssl_preread_server_name $name {
        your.trojan.domain.address trojan;
        default nginx;
    }
    upstream trojan {
        server 127.0.0.1:9443;
    }
    upstream nginx {
        server 127.0.0.1:8443;
    }
    server {
        listen 443;
        listen [::]:443;
        proxy_pass $name;
        ssl_preread on;
    }
}
```
执行`nginx -t`测试配置文件，若提示`nginx: [emerg] unknown directive "stream" in XXX`，则在`nginx.conf`中的`events`区块前增加下面的语句
```nginx
load_module /usr/lib/nginx/modules/ngx_stream_module.so;
```
具体原因参考 [https://serverfault.com/questions/858067/unknown-directive-stream-in-etc-nginx-nginx-conf86](https://serverfault.com/questions/858067/unknown-directive-stream-in-etc-nginx-nginx-conf86) 。

以上就是Trojan放在Nginx后面的配置流程。

