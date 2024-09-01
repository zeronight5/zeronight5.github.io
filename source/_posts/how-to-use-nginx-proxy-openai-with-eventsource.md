---
title: 如何用Nginx反向代理openAI接口
date: 2023-03-05 10:19:00
tags: [Nginx, OpenAI, EventSource]
---

# 如何用Nginx反向代理openAI接口

最近国内很多地方都无法直接访问openAI的接口，从而无法接入ChatGPT，反向代理是一种解决方案。

反向代理是一种常见的服务器配置，通过它可以将客户端的请求转发给后端的服务。在这个教程中，我们将学习如何使用Nginx反向代理来访问OpenAI API。

## 步骤1：安装Nginx

首先，我们需要安装Nginx。在Ubuntu上，可以使用以下命令安装：

```
sudo apt-get update
sudo apt-get install nginx
```

如果有特殊的module需要添加（例如stream），可以自行编译安装，这里不做介绍。

在安装完成后，可以使用以下命令启动Nginx服务：

```
sudo systemctl start nginx
```

## 步骤2：配置Nginx

接下来，我们需要配置Nginx来反向代理OpenAI API。在Nginx的配置文件中添加以下内容：

```
server {
    listen 80;
    server_name {your_domain_name};
    location / {
        proxy_pass  https://api.openai.com/;
        proxy_set_header Host api.openai.com;
        proxy_set_header Connection '';
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

将`{your_domain_name}`替换为您自己的域名。其中以下几项是为了保证使用stream参数请求时，EventSource类型响应的流畅输出。

```
proxy_set_header Connection '';
proxy_http_version 1.1;
chunked_transfer_encoding off;
proxy_buffering off;
proxy_cache off;

```

## 步骤3：测试反向代理

现在，我们可以测试反向代理是否正常工作。使用以下命令重新加载Nginx配置：

```
sudo systemctl reload nginx

```

然后，可以使用以下命令测试反向代理：

```
curl http://{your_domain_name}/v1/api/completions?prompt=Hello%2C%20my%20name%20is%20John%20and%20I%20am

```

如果一切正常，您应该能够收到来自OpenAI的响应。

如果nginx出现形如`[error] 27648#27648: *49512 SSL_do_handshake() failed (SSL: error:14094410:SSL routines:ssl3_read_bytes:sslv3 alert handshake failure:SSL alert number 40) while SSL handshaking to upstream`的错误日志，可以在代理配置添加如下配置：
```
proxy_ssl_server_name on;
```

完整的https配置如下：
```
server {
    listen 443 ssl;
    server_name {your_domain_name};
    ssl_certificate {your_cert_path};
    ssl_certificate_key {your_cert_key_path};
    ssl_session_cache shared:le_nginx_SSL:1m;
    ssl_session_timeout 1440m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers TLS13-AES-256-GCM-SHA384:TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-128-CCM-8-SHA256:TLS13-AES-128-CCM-SHA256:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+ECDSA+AES128:EECDH+aRSA+AES128:RSA+AES128:EECDH+ECDSA+AES256:EECDH+aRSA+AES256:RSA+AES256:EECDH+ECDSA+3DES:EECDH+aRSA+3DES:RSA+3DES:!MD5;
    location / {
        proxy_pass  https://api.openai.com/;
        proxy_ssl_server_name on;
        proxy_set_header Host api.openai.com;
        proxy_set_header Connection '';
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

## 结论

在本教程中，我们学习了如何使用Nginx反向代理来访问OpenAI API。主要是保留EventSource类型响应的流畅输出功能，以提供良好的用户使用体验。
