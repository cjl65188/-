


> 

# 一．配置BOINC server

 - 测试环境
 - 1 [操作系统](%E6%B5%8B%E8%AF%95%E7%8E%AF%E5%A2%83)
 - 2 BOINC相关用户创建及权限配置
 - 3 下载BOINC源码
 - 4 安装BOINC依赖软件
 - 5 配置BOINC依赖软件
 - 6 安装BOINC
 
 [配置BOINC server](#)
 

## 测试环境

 - test bed : pvm088
 - 操作系统：CentOS 7.8 x86_64
 - 硬件：24 CPU core/96GB RAM/500GB disk /10Gb Ethernet
 - 配置安装的主要软件版本
 
| 软件 | 版本 |
|--|--|
| MariaDB | Server version: 10.4.31-MariaDB MariaDB Server |
|Apache|Server version: Apache/2.4.6 (CentOS)          Server built:Mar 24 2022 14:57:57|
## 1 操作系统
 - 配置static ip address
 如果ip地址已经处于静态，无需配置！
 
 - 使用防火墙，开启端口80（http），443（https）
 `firewall-cmd --list-posts80/tcp 443/tcp`
## 2 BOINC相关用户创建及权限配置

2.1 **BOINC的用户**

 - 用户1   apache：scheduler, file uploader, web software
    ·使用`yum install httpd`安装Apache后，apache用户被创建，不需要手动建立
 - 用户2   boincadm : other BOINC programs, also the project owner
 ·不要使用root作为project owner，不利于project的管理和访问
 ·创建boincadm用户，作为project owner
 ```
useradd -U boincadm
```
2.2 **用户及组的权限设置**
 - 将apache加入boincadm组中，由web传回的文件可以写入同组的文件夹
 ```
usermod -a -G boincadm apache
```
## 下载BOINC源码

```
# https://github.com/BOINC/boinc/
git clone https://github.com/BOINC/boinc.git
# 假设BOINC源码所在目录：$BOINC_DIR
```
也可根据自己的选择下载稳定的版本
[BOINC各版本源码](https://github.com/BOINC/boinc)
![输入图片说明](/imgs/2023-10-23/5BVyl0W15A27gpH8.png)
## 4 安装BOINC依赖软件

 - 安装BOINC server所需的依赖软件
 - ``yum install git libtool gcc-c++ libstdc++-static MySQL-python php-mysql php-gd php-cli php-xml openssl-devel mysql-devel mysql-server httpd php autoconf automake libtool libcurl-devel``
 - 安装MariaDB
  `yum -y install mariadb-server mariadb-client mariadb   `
 - MariaDB 安装如下包
 MariaDB-common-10.1.46-1.el7.centos.x86_64
MariaDB-server-10.1.46-1.el7.centos.x86_64
MariaDB-devel-10.1.46-1.el7.centos.x86_64
MariaDB-client-10.1.46-1.el7.centos.x86_64
MariaDB-shared-10.1.46-1.el7.centos.x86_64
## 5 配置BOINC依赖软件
**5.1 MariaDB**
5.1.1 创建boincadm并授权
```
# 以root用户创建boincadm
mysql -u root -p
```
```
CREATE USER 'boincadm'@'localhost' IDENTIFIED BY '自设密码';
# 为boincadm授权，可根据需求细分授权
GRANT ALL ON *.* TO 'boincadm'@'localhost';
```
5.1.2 MariaDB的配置项
 - 关于字符编码的设置
 - 关于远程登录等配置
 - 确定环境变量PATH和LD_LIBRARY_PATH包含MariaDB的可执行文件及库文件路径，如果没有用export命令加入。
 ![输入图片说明](/imgs/2023-10-23/DyqtMd3oqeDgL9Tw.png)
 - 配置完毕记得重启数据库以更新配置
**5.2 Apache**
5.2.1 避免DOS（Denial-Of-Service）攻击, 去掉indexes
```
# 修改/etc/httpd/conf/httpd.conf 中 Options：去掉indexes
<Directory "/var/www/html">
    # 修改前
    #Options Indexes FollowSymLinks
    # 修改后
    Options FollowSymLinks
</Directory>
```
5.2.2 增加request size limit
```
# 在httpd.conf 中设置，单位是字节
# 若文件较大，可设置该项
LimitXMLRequestBody 134217728
LimitRequestBody 134217728
```
5.2.3 配置https
5.2.3.1 安装模块mod_ssl
```
$ yum install mod_ssl
```
5.2.3.2 获取CA
```
- 联系王丽<wangli320@ihep.ac.cn>获取用于Apache https的CA
- 通常为一个压缩包（内含两个文件）+ 一个操作说明文档
```
5.2.3.3 配置httpd.conf
```
# /etc/httpd/conf/httpd.conf
# 包含conf.d/ssl.conf, 通常Apache 2.4使用如下配置即可
IncludeOptional conf.d/*.conf
# loadmodule mod_ssl
LoadModule ssl_module modules/mod_ssl.so
```
5.3.3.4 配置ssl.conf
```
# /etc/httpd/conf.d/ssl.conf
# 确认如下行在配置文件中
<VirtualHost *:443>
# 网页根目录
DocumentRoot "/var/www/html"
# 服务器名称
ServerName ihepboinc.ihep.ac.cn
# SSL CA证书位置
SSLCertificateFile /usr/local/apache/conf/_.ihep.ac.cn_chain.crt
SSLCertificateKeyFile /usr/local/apache/conf/_.ihep.ac.cn_key.key
# SSL engine
SSLEngine on
# SSL Protocal
SSLProtocol all -SSLv2 -SSLv3
# SSL cypher
SSLCipherSuite RC4-SHA:AES128-SHA:HIGH:MEDIUM:!aNULL:!MD5
SSLHonorCipherOrder on
</VirtualHost>
```
![输入图片说明](/imgs/2023-10-23/qR1qS1iLAXmhKAjh.png)
5.3.3.5 重启httpd服务
```
$ systemctl restart httpd
```
## 6 安装BOINC
```
# 以boincadm身份安装
su - boincadm
# 在BOINC源码所在目录下$BOINC_DIR
cd $BOINC_DIR
./_autosetup
# 只编译server和API，更多选项可查看 ./configure -h
./configure --disable-client --disable-manager
make
make install
```
#如果报错就根据错误依次解决就可以成功安装
    

 

  





  
 
       
 



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxODM4MDIzOCw5OTE2MDc1NTldfQ==
-->