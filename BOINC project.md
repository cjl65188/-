


> 

# Boinc Project
## Boinc Project

 - 1 创建Project
   ·1.1 使用make_project脚本创建project
   ·1.2 使用check_project脚本检查
   ·1.3 创建project之后的配置
   ·1.4 Project组成部分
   ·1.5 Project URL
   
 - 2 配置Project
 · 2.1 config
 ·2.2 daemons
 ·2.3 periodic tasks
 
 - 3 管理project
 

1创建project
1.1使用make_project脚本创建project
```
# 首先先切换用户为boincadm
 su - boincadm
#BOINC源码所在目录为$BOINC_dir， 例如我的源码在/home/chengjialong/dvp/boinc/。
 cd /home/chengjialong/dvp/boinc/tools
 ./make_project --test_app --db_passwd <db_passwd> <project_name> 
#尖括号内容分别问boincadm数据库密码和创建的项目名，输入指令时去掉尖括号。可通过--help查看使用说明
 ./make_project --help
```
1.2 使用check_project脚本检查
```
 cd /home/chengjialong/dvp/boinc/tools
 ./check_project -p ~/projects/hepalpha
```
![输入图片说明](/imgs/2023-10-23/RRjUNZy9flsRIfQy.png)
1.3 创建project之后的配置
```
# 在/etc/httpd/conf/httpd.conf增加如下行
 vim /etc/httpd/conf/httpd.conf
 Include /home/boincadm/projects/oak/oak.httpd.conf
# 增加crontab 内容
 crontab ~/projects/hepalpha/hepalpha.cronjob
# 为administrative web interface 增加 username 和 passwd,网页管理员可在日后通过访问url来监视和管理项目。
 htpasswd -cb ~/projects/hepalpha/html/ops/.htpasswd <username> <password>
```
1.4 Project构成
    ·  project由三部分组成
    

 1. 数据库：mysql/MariaDB
 2. project directory
 ![输入图片说明](/imgs/2023-10-23/BYiBC7WpbZTKsJo1.png)
 3. config.xml:用于配置project具体操作

1.5 Project URL
 ·使用make_project脚本创建Project后，project url格式为：
 ```
https://hostname/project/
#https://pvm088.ihep.ac.cn/hepalpha
```

## 2 配置Project
· Project使用配置文件 config.xml 来控制具体操作。包括以下三部分：

 1. config：用于配置scheduler，feeder，database等的具体行为参数
 2. daemons：可以理解为service，若无意外，一直运行
 3. tasks：周期性运行
 ```
<boinc>
  <config>
    [ configuration options ]
  </config>
  <daemons>
    [ list of daemons ]
  </daemons>
  <tasks>
    [ list of periodic tasks ]
  </tasks>
</boinc>
```
## 3管理project
3.1 管理数据库
3.1.1 add platform / applications
·增加app后，要运行bin/xadd，读取`project.xml`中的内容，用于add/update 数据库中的platform / applications
3.1.2 add app versions
在`$Project_home/apps`目录下建立app version目录, 并将相关文件放入该目录中
·目录结构为三层：app_name/app_version/platform/
例如：win/1/windows_intelx86/
·其中有三类文件：
 1. app files：app的code文件（可执行文件）
 2. signature files
 3. version.xml : app version 的说明文件
·使用`bin/update_versions` 增加app version


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MDQxNzY5MjldfQ==
-->