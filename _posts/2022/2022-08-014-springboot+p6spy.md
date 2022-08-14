---
:p6spylayout: post
title: mysql数据库逆向java pojo脚本
categories: 脚本
keywords: java spring 脚本
---

# spring-boot 使用p6spy 监控SQL语句
### 

1. 引入p6spy

```xml
        <dependency>
            <groupId>p6spy</groupId>
            <artifactId>p6spy</artifactId>
            <version>3.9.1</version>
        </dependency>
```

2. 修改application.yml 中的数据库驱动,**修改url**, 这点很容易被忽略

```yml
  datasource: #数据源相关配置节点
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver 
  # driver-class-name: com.mysql.jdbc.Driver  
    url: jdbc:p6spy:mysql://localhost:3306/
  # url: jdbc:mysql://localhost:3306
```

3. 在配置文件同级目录添加 spy.properties文件. (用yml文件应该也可以).我用了mybatis plus一些配置可能稍有不同. 关键点是 **driverlist=com.mysql.jdbc.Driver** 设置成自己原先的数据库驱动

```properties
modulelist=com.baomidou.mybatisplus.extension.p6spy.MybatisPlusLogFactory,com.p6spy.engine.outage.P6OutageFactory
# 自定义日志打印
logMessageFormat=com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger
#日志输出到控制台
appender=com.baomidou.mybatisplus.extension.p6spy.StdoutLogger
# 使用日志系统记录 sql
#appender=com.p6spy.engine.spy.appender.Slf4JLogger
# 设置 p6spy driver 代理
deregisterdrivers=true
# 取消JDBC URL前缀
useprefix=true
# 配置记录 Log 例外,可去掉的结果集有error,info,batch,debug,statement,commit,rollback,result,resultset.
excludecategories=info,debug,result,commit,resultset
# 日期格式
dateformat=yyyy-MM-dd HH:mm:ss
# 实际驱动可多个
driverlist=com.mysql.jdbc.Driver
# 是否开启慢SQL记录
outagedetection=true
# 慢SQL记录标准 2 秒
outagedetectioninterval=2
```

> 这时候p6spy集成内容以及完毕

4.SQL Profiler、IronStack SQL 图形化分析.
