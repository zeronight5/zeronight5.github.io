---
title: 怎样搭建本地AOSP Gerrit服务器
date: 2017-02-09 16:17:33
tags:
---
在这个教程里，我会介绍怎样搭建一个本地Android源码Gerrit服务器。

完成这个教程后，你会拥有一个完全可运行的AOSP镜像和本地Gerrit服务器。

首先我们需要一个Linux服务器。这里用Ubuntu 14.04

Gerrit 需求：

1.Java JDK > 1.7

2.Git

3.SSH server

4.DB

我选择mysql作为数据库服务，当然你也可以用其他的数据库软件。可以参考Gerrit[文档](https://gerrit.googlecode.com/svn/documentation/2.0/install.html#setting_up_the_database)

这里有一句命令可以让你得到你所需要的一切。

```shell
$ sudo apt-get install git openjdk-8-jdk openssh-server mysql-server gitweb
```

下载并安装Android repo

```shell
$ sudo curl https://storage.googleapis.com/git-repo-downloads/repo > /usr/bin/repo
$ sudo chmod a+x /usr/bin
```

现在我们需要设置一个本地Android镜像，这需要一段时间，你可以先去喝杯咖啡。

```shell
$ mkdir -p /usr/local/aosp/mirror
$ cd /usr/local/aosp/mirror
$ repo init -u https://android.googlesource.com/mirror/manifest --mirror
$ repo sync
```

下载[Gerrit](http://gerrit-releases.storage.googleapis.com/index.html)

创建一个Gerrit用户

```shell
$ sudo adduser --system --shell /bin/bash --gecos 'Gerrit Code Review User' --group --disabled-password --home /home/gerrit2 gerrit2
```

Gerrit需要一个数据库来工作，这里用的是mysql。打开终端输入以下命令：

```shell
$ mysql -u root -p
CREATE USER 'gerrit2'@'localhost' IDENTIFIED BY 'secret';
CREATE DATABASE reviewdb;
ALTER DATABASE reviewdb charset=latin1;
GRANT ALL ON reviewdb.* TO 'gerrit2'@'localhost';
FLUSH PRIVILEGES;
quit
```

上面这个命令在mysql中创建了一个gerrit用户，一个数据库和gerrit用户需要的权限集合。

用新的系统用户登陆：

```shell
$ sudo su gerrit2
```

复制上面下载的文件`gerrit-2.13.5.war`到gerrit2的home下，然后执行下面的命令：

```shell
$ java -jar ./gerrit-2.11.war init -d review_site
```

```shell
Using secure store: com.google.gerrit.server.securestore.DefaultSecureStore

*** Gerrit Code Review 2.11
***

Create ‘/home/gerrit2/review_site’ [Y/n]?

*** Git Repositories
***

Location of Git repositories [git]:

*** SQL Database
***

Database server type [h2]: MySQL

Gerrit Code Review is not shipped with MySQL Connector/J 5.1.21
** This library is required for your configuration. **
Download and install it now [Y/n]?
Downloading http://repo2.maven.org/maven2/mysql/mysql-connector-java/5.1.21/mysql-connector-java-5.1.21.jar … OK
Checksum mysql-connector-java-5.1.21.jar OK
Server hostname [localhost]:
Server port [(mysql default)]:
Database name [reviewdb]:
Database username [gerrit2]: gerrit2
gerrit’s password :
confirm password :

*** Index
***

Type [LUCENE/?]:

*** User Authentication
***

Authentication method [OPENID/?]:

*** Review Labels
***

Install Verified label [y/N]?

*** Email Delivery
***

SMTP server hostname [localhost]:
SMTP server port [(default)]:
SMTP encryption [NONE/?]:
SMTP username :

*** Container Process
***

Run as [gerrit2]:
Java runtime [/usr/lib/jvm/java-8-openjdk-amd64/jre]:
Copy gerrit-2.11.war to /home/gerrit2/review_site/bin/gerrit.war [Y/n]?
Copying gerrit-2.11.war to /home/gerrit2/review_site/bin/gerrit.war
*** SSH Daemon
***

Listen on address [*]:
Listen on port [29418]:

Gerrit Code Review is not shipped with Bouncy Castle Crypto SSL v151
If available, Gerrit can take advantage of features
in the library, but will also function without it.
Download and install it now [Y/n]?
Downloading http://www.bouncycastle.org/download/bcpkix-jdk15on-151.jar … OK
Checksum bcpkix-jdk15on-151.jar OK
Generating SSH host key … rsa… dsa… done

*** HTTP Daemon
***

Behind reverse proxy [y/N]?
Use SSL (https://) [y/N]?
Listen on address [*]:
Listen on port [8080]:
Canonical URL [http://ubuntu:8080/]: http://10.0.0.9:8080

*** Plugins
***

Installing plugins.
Install plugin download-commands version v2.11 [y/N]?
Install plugin reviewnotes version v2.11 [y/N]?
Install plugin singleusergroup version v2.11 [y/N]?
Install plugin replication version v2.11 [y/N]?
Install plugin commit-message-length-validator version v2.11 [y/N]?
Initializing plugins.
No plugins found with init steps.

Initialized /home/gerrit2/review_site
Executing /home/gerrit2/review_site/bin/gerrit.sh start
Starting Gerrit Code Review: OK
Waiting for server on ubuntu:8080 … OK
Opening http://10.0.0.9:8080/#/admin/projects/ …FAILED
Open Gerrit with a JavaScript capable browser:
http://10.0.0.9:8080/#/admin/projects/
```

现在Gerrit已经开始运行了。

需要注意的是，第一个登陆到gerrit的用户将会成为管理员。

Gerrit能通过多种认证方式登陆。上面选择的OpenID。

现在让我们打开浏览器并打开Gerrit网站。

注册并登陆后需要设置SSH keys，这样我们就能够在命令行下工作。

下面我们在Ubuntu系统的另一个用户下执行以下命令：

```shell
$ ssh-keygen <ENTER>
Generating public/private rsa key pair.
Enter file in which to save the key (/home/serveradmin/.ssh/id_rsa):  <ENTER>
Enter passphrase (empty for no passphrase): <ENTER>
Enter same passphrase again: <ENTER>
Your identification has been saved in /home/serveradmin/.ssh/id_rsa.
Your public key has been saved in /home/serveradmin/.ssh/id_rsa.pub.
The key fingerprint is:
d5:7b:51:d8:22:0e:95:63:f9:0e:a2:22:1c:97:76:40 serveradmin@ubuntu
The key's randomart image is:
+---[RSA 2048]----+
| .E ..o o.|
| . ..* o..|
| o .+.+.. |
| . + ... o... |
| . + .S. ..o. |
| o . . .. |
| . . |
| |
| |
+-----------------+

$ cat ~/.ssh/id_rsa.pub  <ENTER>
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDobCUbb8ExP9ci48ZCzCUE3P+IjoH6zrv/l88+4NOTf9FJWDAl4SHCXI+mrZoFEeZo9
uieKTuvHqUFQNpnVA9vfNY6bhaTucAAt0fX9Q1M1ZtNj2IxaQd7u9PnxjSGEig0BUtjqDEu4CMhMShXGWsGAwL7ju/qu7G7RF
iK/Wtcye6wUSjziXseCusb1DUZZ6dsOpxrPYEM3kwXpItQtAq1oEIQWsxEgj2nrOqRXm1UWdqIQU1X45XHQtg5iqi44PfLVNU
3alu453MeWn5PrpSS5dFw/7AkBW4KMPwrOvVnu8gb9xLA0TWtPf+sQ9ROEQC5SBeO+4Q9XRNf5YaswpLj 
```

在网站的右上角点击你的用户名然后点击Settings，在左侧的菜单中选择“SSH Public Keys”，复制上面的内容到里面，然后点Add。

下面让我们测试一下通过SSH访问Gerrit：

```shell
ssh -p 29418 admin@localhost <ENTER> <Change "admin" to your username in gerrit>

**** Welcome to Gerrit Code Review ****

Hi Administrator, you have successfully connected over SSH.

Unfortunately, interactive shells are disabled.
To clone a hosted Git repository, use:

git clone ssh://ramon@10.0.0.9:29418/REPOSITORY_NAME.git
Connection to localhost closed.
```

出现以上信息说明登陆成功，下面创建两个group：“android-admin” and “android”

前者用来管理，包含review，merge，delete等权限，后者只有view和change-set的权限。

在网页中点击People -> Create New Group，输入名称“android-admin”。执行同样的操作创建“android”。

现在我们为AOSP代码树创建一个父项目。所有为这个项目的设置都会继承到子项目。如果跳过这步，AOSP项目会继承“All-Projects”项目，而这个项目太通用了。

在网站中点击Project -> Create New Project，填入以下内容：

```
Project Name: Android
Rights Inherit From: All-Projects
```

勾选 “Only Serve As Parent For Other Projects”，然后点击“Create Project”按钮。

项目已经创建完，下面来为这个项目设置权限控制：

点击Projects -> List，然后选择“Android”项目，点击“Access”然后点击“Edit”，修改成如下图。图中还包含了其他的群组，可自行设置。

![Gerrit权限控制](https://crazystar.net/images/gerrit.png)

现在我们可以把所有的Android项目推送到Gerrit了。

第一条命令会在Gerrit上创建所有的Android项目；第二条命令会将刚刚创建的项目的父项目修改为“Android”；第三条命令会将代码推送到Gerrit，时间会比较长。

进入下载Android镜像的目录执行以下命令：

```shell
$ repo forall -c 'echo $REPO_PATH; ssh -p 29418 admin@localhost gerrit create-project --name android/$REPO_PATH --owner android;'

accessories/manifest
device/asus/deb
device/asus/flo
device/asus/flo-kernel
device/asus/fugu
device/asus/fugu-kernel
…

$ repo forall -c 'echo $REPO_PATH; ssh -p 29418 admin@localhost gerrit set-project-parent --parent Android android/$REPO_PATH;'

accessories/manifest
device/asus/deb
device/asus/flo
device/asus/flo-kernel
…

$ repo forall -c 'echo $REPO_PATH; git push ssh://ramon@localhost:29418/android/$REPO_PATH +refs/heads/* +refs/tags/*;'
```

大功告成！



创建Gerrit服务自启：

```shell
$ sudo echo "GERRIT_SITE=/opt/gerrit-review" >> /etc/default/gerritcodereview
$ sudo ln -snf /opt/gerrit-review/bin/gerrit.sh /etc/init.d/gerrit
$ sudo update-rc.d gerrit defaults
```

下面贴一个gerrit配置文件(postgre数据库，ldap认证，提交信息增加jira链接)

```
[gerrit]
	basePath = git
	canonicalWebUrl = http://localhost:8080/
[database]
	type = postgresql
	hostname = localhost
	database = reviewdb
	username = gerrit2
[index]
	type = LUCENE
[auth]
	type = LDAP
[ldap]
	server = ldap://localhost:389
	username = cn=admin,dc=darkerthanblack,dc=org
	accountBase = ou=People,dc=darkerthanblack,dc=org
	groupBase = ou=group,dc=darkerthanblack,dc=org
[receive]
	enableSignedPush = false
[sendemail]
	smtpServer = smtp.exmail.qq.com
	smtpServerPort = 465
	smtpEncryption = SSL
	smtpUser = example@example.com
        smtpPass = your_password
        from = example@example.com
[container]
	user = gerrit2
	javaHome = /usr/lib/jvm/java-8-openjdk-amd64/jre
[sshd]
	listenAddress = *:29418
	maxConnectionsPerUser = 0
[httpd]
	listenUrl = http://*:8080/
[cache]
	directory = cache
[gitweb]
	cgi = /usr/lib/cgi-bin/gitweb.cgi
[commentlink "jira"]
	match = ([A-Z]+-[0-9]+)
	link = http://localhost:8081/browse/$1

```

