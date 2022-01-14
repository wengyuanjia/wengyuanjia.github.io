---
layout: post
title: Docker lnmp
categories: 经验
keywords:  Docker lnmp
---
> 在linux上安装php集成环境,可以使用宝塔面板.在Mac上自己单独配置过 apache,php,mysql.
> 但是年代久远,有些细节都忘记了,而且系统升级,可能会有新坑.想找一个简单的方式搭建php环境.
> MAMP PRO 应该挺好用,但是费用有点贵.当时刚升级big sur 破解版的运行不起来.最终考虑使用
> docker.

>docker 安装PHP环境 用lnmp最方便,pull官方 php, 还需要安装一些扩展比较麻烦,选择了星比较高德       
>2233466866/lnmp 的docker[仓库连接](https://hub.docker.com/r/2233466866/lnmp)
```shell
# 主流版本
docker pull 2233466866/lnmp
#9000需要开吗?宿主机的nginx 需要访问容器的9000端口
# 端口映射,建议多开几个! 以备后用
#挂载wwwroot, 数据库,nginx配置目录! 
docker run -dit \
-p 80:80 \
-p 443:443 \
-p 3306:3306 \
-p 9000:9000 \
-v /Users/wengyuanjia/Docker/lnmp/www:/www \
-v /Users/wengyuanjia/Docker/lnmp/mysql:/data/mysql \
-v /Users/wengyuanjia/Docker/lnmp/nginx:/usr/local/nginx \
--privileged=true \
--name=lnmp \
2233466866/lnmp
```

修改数据库密码
```shell
docker exec -it lnmp /bin/bash
cat /var/log/mysqld.log|grep 'A temporary password'
mysql -h localhost -u root -p
```
```mysql
alter user user() identified by "Wengyuanjia1!";
use mysql;
select user,host from user;
update user set host='%' where user='root';
flush privileges;
#密码要求
show variables like 'validate_password%';
#更改密码强度
set global validate_password_policy=0;
set global validate_password_length=4;
```
意外删除容器后,直接挂在之前的文件夹 数据库和PHP代码没有丢失,只用配置一下没有挂载的nginx配置文件,就可以恢复!!
