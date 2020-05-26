---
title: Linux部署PPTP以及L2TP over IPsec
url: 204.html
id: 204
categories:
  - Linux
  - Ubuntu
date: 2014-12-10 23:22:06
tags:
---

操作系统 ubuntu 12.04 内核3.16.5-x86_64 Openswan U2.6.37 xl2tpd version: xl2tpd-1.3.1

PPTP部署
------

1.安装pptpd \[code\]apt-get install pptpd\[/code\] 2.配置文件 \[code\]vim /etc/ppp/options.pptpd\[/code\] 填写如下内容

    
    name pptpd
    refuse-pap
    refuse-chap
    refuse-mschap
    require-mschap-v2
    require-mppe-128
    proxyarp
    lock
    nobsdcomp
    novj
    novjccomp
    nologfd
    idle 2592000
    ms-dns 8.8.8.8
    ms-dns 8.8.4.4
    

说明：

    
    name pptpd : pptpd server 的名称。
    refuse-pap : 拒绝 pap 身份验证模式。
    refuse-chap : 拒绝 chap 身份验证模式。
    refuse-mschap : 拒绝 mschap 身份验证模式。
    require-mschap-v2 : 在端点进行连接握手时需要使用微软的 mschap-v2 进行自身验证。
    require-mppe-128 : MPPE 模块使用 128 位加密。
    ms-dns 8.8.8.8
    ms-dns 8.8.4.4 : ppp 为 Windows 客户端提供 DNS 服务器 IP 地址，第一个 ms-dns 为 DNS Master，第二个为 DNS Slave。
    proxyarp : 建立 ARP 代理键值。
    debug : 开启调试模式，相关信息同样记录在 /var/logs/message 中。
    lock : 锁定客户端 PTY 设备文件。
    nobsdcomp : 禁用 BSD 压缩模式。
    novj
    novjccomp : 禁用 Van Jacobson 压缩模式。
    nologfd : 禁止将错误信息记录到标准错误输出设备(stderr)。
    

\[code\]vim /etc/ppp/chap-secrets\[/code\] 输入以下内容

    myusername pptpd mypassword *

\[code\]vim /etc/pptpd.conf\[/code\] 填入以下内容

    
    option /etc/ppp/options.pptpd
    logwtmp
    localip 10.82.18.1
    remoteip 10.82.18.2-254
    

\[code\] sed -i 's/net.ipv4.ip\_forward = 0/net.ipv4.ip\_forward = 1/g' /etc/sysctl.conf sysctl -p \[/code\]

IPSec+L2TP部署
------------

1.安装openswan和xl2tpd \[code\]apt-get install openswan xl2tpd \[/code\] 安装过程中选择使用X.509，创建，自签名 2.配置文件 \[code\]vim /etc/ipsec.conf\[/code\] 填入以下内容

    
    version 2.0  
    config setup  
        nat_traversal=yes  
        virtual_private=%v4:10.0.0.0/8,%v4:192.168.0.0/16,%v4:172.16.0.0/12  
        oe=off  
        protostack=netkey  
    conn L2TP-PSK-NAT  
        dpddelay=40  
        dpdtimeout=130  
        dpdaction=clear  
        rightsubnet=vhost:%priv  
        also=L2TP-PSK-noNAT  
    conn L2TP-PSK-noNAT  
        authby=secret  
        pfs=no  
        auto=add  
        keyingtries=3  
        rekey=no  
        ikelifetime=8h  
        keylife=1h  
        type=transport  
        left=106.186.178.35  
        leftprotoport=17/1701  
        right=%any  
        rightprotoport=17/%any 
    

\[code\]vim /etc/ipsec.secrets \[/code\]

    YOUR.IP %any: PSK "password"

\[code\]vim /etc/xl2tpd/xl2tpd.conf\[/code\]

    
    [global]  
    ipsec saref = yes  
    [lns default]  
    ip range = 10.1.2.2-10.1.2.255  
    local ip = 10.1.2.1   
    refuse chap = yes  
    refuse pap = yes  
    require authentication = yes 
    name xl2tpd
    ppp debug = yes  
    pppoptfile = /etc/ppp/options.xl2tpd  
    length bit = yes
    

\[code\]vim /etc/ppp/options.xl2tpd \[/code\]

    
    require-mschap-v2  
    ms-dns 8.8.8.8  
    asyncmap 0  
    auth  
    crtscts  
    lock  
    hide-password  
    modem  
    debug  
    name xl2tpd  
    proxyarp  
    lcp-echo-interval 30  
    lcp-echo-failure 4  
    

\[code\]vim /etc/ppp/chap-secrets \[/code\]

    
    按说明填写,server写xl2tpd
    

\[code\]vim /etc/rc.local\[/code\]

    
    iptables --table nat --append POSTROUTING --jump MASQUERADE  
    echo 1 > /proc/sys/net/ipv4/ip_forward  
    for each in /proc/sys/net/ipv4/conf/*  
    do  
    echo 0 > $each/accept_redirects  
    echo 0 > $each/send_redirects  
    done  
    exit 0
    

\[code\] service ipsec restart service xl2tpd restart \[/code\]