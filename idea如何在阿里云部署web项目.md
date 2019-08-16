# idea如何在阿里云部署web项目 #
之前一直在跟着做一个ssm的项目，看到在阿里云部署就先搁置了，现在开始一步一步在阿里云部署java网站。

## 一、申请阿里云服务器 ##

（1）因为是学生，所以直接搜索云翼计划2018，支付宝登录，通过学信网学生认证之后，就能撸羊毛了。一年114，与原价相比基本是一折优惠，果断撸一年的。我选择的是centos7.3。
![](http://note.youdao.com/noteshare?id=2be6635b3668f03779bfcb3acf1d0059&sub=F52F28180B2741E9AFC68B4DC32C598C)

（2）开通成功后，服务器会启动并运行，同时会自动分配一个公网IP，外网就可以通过这个公网IP访问服务器。
![](http://note.youdao.com/noteshare?id=a55abf53391a2b6998712272ff713979&sub=743731BAFF724758A66A3494E9E90DEF)





## 二、搭建程序运行的环境 ##

1.下载软件到本地

（1）JDK

我选择的版本是jdk-8u65-linux-x64.rpm

（2）Tomcat

我选择的版本是apache-tomcat-8.5.41.tar.gz

（3）Mysql

我选择的是Mysql5.7repo源，后通过centos自带的yum安装，下载的地址为

https://dev.mysql.com/downloads/repo/yum/

这里选择mysql57-community-release-el7-11.noarch.rpm

将上述软件下载到本地，然后上传的服务器，我使用的是Xshell和Xftp，还是非常方便的

2.安装软件

（1）安装JDK

Java程序需要运行在JRE里边，因此咱们需要安装JDK，通过Xshell远程连接阿里云服务器，执行命令

//添加可执行权限

chmod +x jdk-8u144-linux-x64.rpm

//安装RPM软件包

rpm -ivh jdk-8u144-linux-x64.rpm

//查看java的版本信息，若出现版本信息则成功

java -version

（2）安装Mysql

安装用来配置mysql的yum源的rpm包

rpm -Uvh mysql57-community-release-el7-11.noarch.rpm

安装Mysql

yum install mysql-community-server

开启mysql服务

service mysqld start

mysql安装成功后创建的超级用户 'root'@'localhost'的密码会被存储在/var/log/mysqld.log，可以使用如下命令查看密码

grep 'temporary password' /var/log/mysqld.log

----------
使用mysql生成的'root'@'localhost'用户和密码登录数据库，并修改其密码，具体命令

mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED BY '修改后的密码';

开启远程连接

通过阿里云控制台开放3306端口，在阿里云控制台实例页面下面选择安全组->配置规则
进入到规则配置页面之后，可以看到目前只有22端口和3389端口支持远程访问，额外开通80端口3306端口(mysql)

配置一个支持远程登录的帐号，这里配置一个work账号

mysql -u root -p

use mysql;

grant SELECT,UPDATE,INSERT,DELETE on *.* to 'work'@'%' identified by '修改后的密码';//创建work帐号并授权，同时设置密码

flush privileges;//生效配置

之后便能在我们本地通过调用mysql指令远程登录阿里云服务器上的mysql server中，

mysql -uwork -P3306 -h公网IP地址 -p //本机远程登录mysql指令

（3）安装tomcat 8

解压tomcat压缩包

tar -zxvf apache-tomcat-8.0.46.tar.gz

启动tomcat

./apache-tomcat-8.0.46/bin/startup.sh

我的tomcat总是会有一个问题，可以启动无数次，但是关闭它时会报错。如果你也是这样，按如下步骤做即可。

例如，我的jdk路径是“/usr/java/jdk1.8.0_65”。

cd /usr/java/jdk1.8.0_65/jre/lib/security/

找到名为“java.security”的文件，

vi java.security

找到“securerandom.source = file：/ dev / random”。

修改“securerandom.source = file：/ dev /./ urandom”。

然后，转到Tomcat/bin目录，执行./start.sh并./shutdown.sh发现一切正常。

修改自己本地的网站的配置

即把项目里的mysql配置，修改为阿里云服务器对应的配置(即ip，端口，密码等配置修改成服务器里安装好的mysql的对应的配置)

## 三、在服务器上发布并运行自己的web项目 ##

通过idea将项目以war包的形式打出

图

点击apply，确定即可

war包的输出位置就在项目的原文件out文件夹中，通过Xftp将war包复制到阿里云服务器tomcat中的webapps下即可。上传成功后，没过几秒tomcat便会在webapps目录下自动从项目war包中解析出项目工程目录来。

我是通过本地的sqlyog登录阿里云服务器中的数据库，将本地数据库中的数据复制到阿里云中的，操作也很简单，复制数据库时可能会出现权限不够的问题，只要输入命令

grant all on xxx.* to 'root'@'%' identified by 'password' with grant option;

其中：xxx代表创建的数据库; password为用户密码，在此为root的密码。

之后，就可以复制了。

最后通过ip+请求路径的形式便可访问到自己的项目。













