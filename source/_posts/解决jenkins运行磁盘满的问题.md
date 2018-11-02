---
title: 解决jenkins运行磁盘满的问题
date: 2018-09-28 16:26:23
categories: jenkins
tags: [jenkins]
---
### 前言
> jenkins服务器，运行了一段时间后，发现服务器磁盘目录快不够用了。通过`du -h --max-depth=1 /` 逐级目录排查，发现/var/lib/jenkins目录文件过大。通过以下两种方法，解决该问题。

<!-- more -->

### 优化方案
- **自动丢弃构建历史数据**  
    把以前构建过的过时历史数据自动清除掉，保留最近更新的天数和个数,根据个人需求保留。如下图
    ![丢弃旧的构建设置](https://upload-images.jianshu.io/upload_images/8760038-a8def1a40d13dea7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **更改jenkins默认目录**  
    > 我这里之前的jenkins已经配置好并运行了一段时间，并不想重装jenkins。所以需要转移jenkins目录，把默认目录/var/lib/jenkins，更改到其他大目录或者磁盘中。我这的环境是用的阿里云centos7，我把默认目录转移到/home/modules下
    
    - 把/var/lib/jenkins拷贝到/home/modules下
        ```
        cp -r /var/lib/jenkins /home/modules
        #因为是在root用户下操作的，所以需要更改目录所属用户为默认用户jenkins
        chown -R jenkins:jenkins /home/modules/jenkins
        ```
    - 修改/etc/init.d/jenkins的jenkins目录
        ```
        DAEMON_ARGS="--name=$NAME --inherit --env=JENKINS_HOME=/home/modules/jenkins --output=$JENKINS_LOG --pidfile=$PIDFILE"
        ```
    - 修改/etc/sysconfig/jenkins文件
        ```
        vi /etc/sysconfig/jenkins
        ```
        修改文件中的JENKINS_HOME，把JENKINS_HOME="/var/lib/jenkins"改成JENKINS_HOME="/home/modules/jenkins"  
        修改内容：  
        ```
        #JENKINS_HOME="/var/lib/jenkins"
        JENKINS_HOME="/home/modules/jenkins"
        ```
    - 修改/etc/passwd中的jenkins  
        把其中的Server:/var/lib/jenkins改成/home/modules/jenkins
        ```
        vi /etc/passwd
        修改后的内容如下：
        jenkins:x:994:991:Jenkins Automation Server:/home/modules/jenkins:/bin/false
        ```
    - 重启jenkins
        ```
        service jenkins restart
        ```
    - 可能出现的问题  
    如果jenkins安装的maven、gradle是用的自动安装的方式，需要手动在勾选下“自动安装”，在保存，让其进行重新安装。因为默认安装的目录是在/var/lib/jenkins下。  
    
    参考链接：https://blog.csdn.net/ling811/article/details/74991899