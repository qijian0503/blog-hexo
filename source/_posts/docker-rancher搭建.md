---
title: docker-rancher搭建
date: 2018-09-22 19:25:49
categories: rancher
tags: [docker, rancher]
---

### 环境说明
- linux：centos7
- docker  
`Docker version 1.13.1, build dded712/1.13.1`
- rancher：v1.6.18

> 本环境搭建需要先安装docker，docker安装这里不写了，大家自行百度吧。

<!-- more -->
### 创建MySQL容器
- 创建挂载目录  
    ```
    mkdir -p /opt/datas/mysql/{datadir,conf.d,logs}
    ```

- 创建mysql容器,设置密码123456  
    ```
    docker run --name mysqldb -p 3306:3306 \
      -v /opt/datas/mysql/datadir:/var/lib/mysql \
      -v /opt/datas/mysql/conf.d:/etc/mysql/conf.d \
      -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
    ```
- 查询测试  
    ```
    docker exec -it mysqldb mysql -p123456 -e "show databases;"
    ```
    返回如下信息说明mysql数据库初始化成功：
    ```
    mysql: [Warning] Using a password on the command line interface can be insecure.
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | cattle             |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    
    ```
- 创建库并授权(库,用户,密码都为cattle)   
    ```
    docker exec -it mysqldb mysql -p123456 -e "
      create database if not exists cattle collate = 'utf8_general_ci' character set = 'utf8';
      grant all on cattle.* to 'cattle'@'%' identified by 'cattle';
      grant all on cattle.* to 'cattle'@'localhost' identified by 'cattle';
      flush privileges;show databases;"
    ```
    运行mysql容器，提示如下错误：
    ![docker-mysql-error.png](https://upload-images.jianshu.io/upload_images/8760038-cec923fbc38dd752.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
    解决方法：关闭linux selinux  
    查看：https://blog.csdn.net/lijiqidong/article/details/78482908

### 创建rancher容器
- 运行rancher容器  
    mysql机器IP：mysql容器所在的机器IP  
    ```
    docker run -d --name rancher --link=mysqldb:db \
    --restart=unless-stopped -p 8080:8080 -p 9345:9345 rancher/server:latest \
    --db-host db --db-port 3306 --db-user cattle --db-pass cattle --db-name cattle \
    --advertise-address mysql机器IP
    ```
    等几分钟,当数据表创建超过100张时,可以打开浏览器访问rancher web管理页面了。  
    查询cattle数据库中表的数量，显示为109时安装完成  
    ```
    #查询cattle数据库中表的数量
    docker exec -it mysqldb mysql -u"cattle" -h localhost -p"cattle" -e "use cattle;show tables;" |wc -l
    ```
    ![cattle数据库](https://upload-images.jianshu.io/upload_images/8760038-71192bb9dae58c4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- rancher web管理页面  
    访问：http://主机IP:8080

### 添加主机(节点)
基础架构——主机——添加主机(保存)——复制第5部代码，在需要管理的docker机器节点执行。
![添加docker主机节点](https://upload-images.jianshu.io/upload_images/8760038-d31484741dfaf88d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 访问控制
系统管理--访问控制--开启访问控制
![开启访问控制](https://upload-images.jianshu.io/upload_images/8760038-ccf1f9125a4bad99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参考链接：  
https://www.cnblogs.com/elvi/p/8478551.html