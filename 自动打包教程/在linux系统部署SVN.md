# 教程

1）https://www.jb51.net/article/175835.htm

## **步骤一：**安装subversion

1) 连接xshell，在xshell输入命令**：**yum install subversion

2) 查看安装svn服务的版本：svnserve --version

![image-20201106142531786](img\Snipaste_2020-11-06_14-25-24.png)

## **步骤二：创建版本库**

1）连接Xftp，在var目录下创建svn目录

mkdir svn

2）在xshell依次输入以下命令：

cd /var/svn   //先进入svn目录

svnadmin create /var/svn/proname 

 //用svn管理员创建proname库***（只能有一个库），可以在这分目录放置不同的项目代码***

cd  proname  //进入库  

ls //查看库中的文件

![Snipaste_2020-11-06_14-32-50](img\Snipaste_2020-11-06_14-32-50.png)

## **步骤三：版本库权限配置**

authz文件是权限控制文件

passwd是帐号密码文件

svnserve.conf SVN服务配置文件

1）在xshell输入命令：cd /var/svn/proname/conf/  进入权限配置目录下

2）配置svnserve.conf 文件。切到conf目录输入命令：vi svnserve.conf

输入i，再修改以下内容：注意去掉#，去掉空格顶格显示

![1668514-20190429172737596-531990292](img\1668514-20190429172737596-531990292.png)

realm = /var/svn/proname

3）配置passwd文件：切到conf目录输入命令：vi  passwd

输入i，输入用户和密码信息：

![Snipaste_2020-11-06_14-43-55](img\Snipaste_2020-11-06_14-43-55.png)

4）配置authz文件。切到conf目录输入命令：vi authz

输入i，输入用户权限信息：
liwenlang = liwenlang
[/]
@liwenlang = rw

*=

## **步骤四：启动svn版本库**

svnserve -d -r /var/svn/proname --listen-port=3690

重启：

 先杀掉服务：killall svnserve

查看服务是否开启：ps -ef |grep svn

再重启服务：svnserve -d -r /var/svn/proname --listen-port=3690



## **步骤五：文件的导入导出**

checkout 

**[group]不能重复**

需要使用svn://而不要使用http://协议否则会提示以下错误信息:

svn://121.196.121.18/MyProject2

# **配置Apache支持HTTP访问**

https://blog.csdn.net/u011781521/article/details/80200583?utm_medium=distribute.pc_relevant.none-task-blog-title-10&spm=1001.2101.3001.4242



## **1.安装httpd mod_dav_svn**

yum install  httpd 

yum install  mod_dav_svn

统一安装，以防冲突

yum install -y httpd mod_dav_svn

出现错误：

Error: Missing Dependency: httpd-mmn =

20051115 is needed by package

mod_dav_svn

```
$find / -name yum.conf
$nano /etc/yum.conf

从此行删除httpd *：

    exclude = apache * bind-chroot courier * dovecot * exim * filesystem httpd * mod_ssl * mydns * mysql * nsd * perl * php * proftpd * pure-ftpd * ruby​​ * spamassassin * squirrelmail *

保存并关闭yum.conf,安装mod_dav_svn

    yum install mod_dav_svn
```

## 2.检查Apache,mod_dav_svn是否安装成功

httpd -v

find / -name mod_dav_svn.so

find / -name mod_authz_svn.so



通过查看文件/usr/lib/systemd/system/svnserve.service, 了解到svnserver的配置文件是/etc/sysconfig/svnserve
修改/etc/sysconfig/svnserve

```
[root@localhost ~]# vim /etc/sysconfig/svnserve
OPTIONS="-r /var/svn"     
======> OPTIONS="-r /var/www/svn"　
```

## 3.修改配置文件/etc/httpd/conf.d/subversion.conf(没有则新建),內容为：

vim /etc/httpd/conf.d/subversion.conf

```
LoadModule dav_svn_module modules/mod_dav_svn.so
LoadModule authz_svn_module modules/mod_authz_svn.so
<Location /svn>
    DAV svn
    SVNPath /var/svn/proname
    SVNParentPath  /var/svn
    AuthType Basic
    AuthName "Authorization SVN"
    AuthzSVNAccessFile /var/svn/proname/authz
    AuthUserFile /var/svn/proname/passwd
    Require valid-user
</Location>
```

```
LoadModule dav_svn_module     modules/mod_dav_svn.so
LoadModule authz_svn_module   modules/mod_authz_svn.so
 
<Location /svn>
    DAV svn
    SVNListParentPath on
    SVNPath /home/root/svn
    AuthType Basic
    Satisfy Any
    AuthName "Subversion repos"
    AuthUserFile /home/root/svn/conf/accesspwd
    AuthzSVNAccessFile home/root/svn/conf/authz
    Require valid-user
</Location>
```



## 4.创建用户文件passwd

touch /var/svn/passwd

htpasswd /var/svn/passwd admin  123456

\#查看用户名密码  cat /var/svn/passwd

##### htpasswd命令是Apache的Web服务器内置工具，用于创建和更新储存用户名、域和用户基本认证的密码文件。

## 5.创建权限文件authz

cp /var/svn/proname/conf/authz /var/svn/authz 拷贝

## 6.配置apache对SVN目录权限

chown -R apache:apache /var/svn/proname

## 7.配置httpd

1)vim /etc/httpd/conf/httpd.conf

修改

AllowOverride None改为AllowOverride All

2)启动Apache

```
service httpd restart
```

svn checkout --username=admin --password=123456 file:///var/svn/proname/ /www/wwwroot/svn/

## 8.使用http访问

http://121.196.121.18:3690/svn

问题：The XML response contains invalid XML

加上端口号

## 9.增加防火墙规则