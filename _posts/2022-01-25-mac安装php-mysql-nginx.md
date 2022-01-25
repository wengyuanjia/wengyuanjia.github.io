---
layout: post
title: Typora的简单使用技巧
categories: markdown
keywords: typora markdown

---

#  mac安装php-mysql-nginx

>docker-lnmp环境改端口,nginx设置比较麻烦.然后也体验了MAMP(破解需要关闭SIP),MxSrvs等集成环境,用着也不习惯.还是单独安装吧,这样方便设置.这里记录一下遇到的坑,大部分是"墙"造成的.

##### nginx `brew install nginx`

##### 安装Php-fpm

```bash
brew install php@7.3
#Error: php@7.3 has been disabled because it is a versioned formula!
brew tap shivammathur/php
brew install shivammathur/php/php@7.3
#太慢需要使用代理
ALL_PROXY=socks5://127.0.0.1:7890 brew install shivammathur/php/php@7.3
php-fpm -t  #类似nginx 查看配置是是否正确,也可以查看配置路径
cd /private/etc/
#复制默认配置文件 php-fpm.conf.default , 当前没有配置文件复制一份
sudo cp php-fpm.conf.default php-fpm.conf
open  php-fpm.conf 
error_log = /Users/wengyuanjia/phplog/php-fpm.log
#端口在这个文件设置 www.conf.default
sudo cp /private/etc/php-fpm.d/www.conf.default /private/etc/php-fpm.d/www.conf
#[pool www] 'user' directive is ignored when FPM is not running as root
 #启动
sudo php-fpm
brew services restart shivammathur/php/php@7.3
brew services stop shivammathur/php/php@7.3
#有时候需要用这个命令 
sudo php-fpm -D  
 #关闭
sudo  killall  php-fpm 
```



##### 安装Mysql

```bash
brew install mysql@5.7
#注意看提示 , 加入环境变量
echo 'export PATH="/usr/local/opt/mysql@5.7/bin:$PATH"' >> ~/.zshrc
#启动 关闭,可能需要权限
sudo mysql.server start
sudo mysql.server restart
sudo mysql.server stop
#To restart mysql@5.7 after an upgrade:
#/etc/my.cnf /etc/mysql/my.cnf /usr/local/etc/my.cnf ~/.my.cnf 
brew services restart mysql@5.7
#MySQL 的配置启动文件列表：
mysqld --help --verbose | less
#配置文件在
open /usr/local/etc/my.cnf
```

```sql
#修改密码
alter user 'root'@'localhost' identified by '123456';
```

```bash
[root@test etc]# vi my.cnf
[mysqld]
port=3506
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
 
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

