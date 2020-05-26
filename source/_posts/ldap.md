---
title: 搭建ldap服务器并认证登陆linux系统
date: 2017-02-10 16:27:33
tags:
---

1.搭建ldap服务器

在终端输入下面的命令：

```shell
sunny@server1:~$ sudo apt-get install slapd ldap-utils
```

填入相应的信息。如果相重新填写执行`sudo dpkg-reconfigure slapd`

安装管理工具：

```shell
sunny@server1:~$ sudo apt-get install phpldapadmin
```

修改配置文件

```shell
sunny@server1:~$ sudo vim /etc/phpldapadmin/config.php
```

在`$servers->setValue('server','base',array('dc=example,dc=com'));`以及`$servers->setValue('login','bind_id','cn=admin,dc=example,dc=com');`位置将`dc=example,dc=com`修改为上面配置slapd时填入的域名。

登入管理页面`http://localhost/phpldapadmin`

创建People和group对象，并建立群组和用户实例。

2.ldap认证登陆

```shell
sunny@server2:~$ sudo apt-get install ldap-auth-client nscd
```

```shell
LDAP server Uniform Resource Identifier: ldap://LDAP-server-IP-Address
将默认的"ldapi:///"修改为"ldap://"
Distinguished name of the search base:填入上面"dc=example,dc=com"部分
LDAP version to use: 3
Make local root Database admin: No
Does the LDAP database require login? No
LDAP account for root:填入上面"cn=admin,dc=example,dc=com"部分
LDAP root account password: 你的ldap管理密码
```

如果想重新配置执行`sunny@server2:~$ sudo dpkg-reconfigure ldap-auth-config`

编辑文件`nsswitch.conf`

```shell
sudo vim /etc/nsswitch.conf
```

```shell
passwd: files ldap
group: files ldap
shadow: files ldap
```

上面三部分需要包含ldap

编辑文件

```shell
sunny@server2:~$ sudo vim /etc/pam.d/common-session
```

在行尾填入如下内容：

```
session required    pam_mkhomedir.so skel=/etc/skel umask=0022
```

重启服务：

```shell
sudo /etc/init.d/nscd restart
```

这样这台服务器就可以通过server1的ldap中配置的用户登陆了。
