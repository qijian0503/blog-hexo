---
title: 解决docker容器存放目录磁盘空间满了问题
date: 2018-09-28 17:05:38
categories: docker
tags: [docker]
---
### 前言
> docker所在服务器，运行了一段时间后，发现服务器磁盘目录快不够用了。通过`du -h --max-depth=1 /` 逐级目录排查，发现/var/lib/docker目录文件过大。通过以下方法，解决该问题。

<!-- more -->

### 转移数据修改docker默认存储位置
> 有多种方式修改docker默认存储位置。  
最好是在docker安装完后，第一时间修改docker默认存储位置为其他大目录或者磁盘中。规避迁移数据过程中造成的风险。

- 停止docker服务
    ```
    systemctl stop docker
    ```
- 创建新的docker目录，执行命令df -h,找一个大的磁盘   
    我在 /home目录下面建了/home/modules/docker/lib目录
    ```
     mkdir -p /home/modules/docker/lib
    ```
- 迁移/var/lib/docker目录下面的文件到/home/modules/docker/lib  
    迁移后的完成docker路径：/home/modules/docker/lib/docker
    ```
    rsync -avz /var/lib/docker/ /home/modules/docker/lib/
    ```
- 配置 /etc/systemd/system/docker.service.d/devicemapper.conf  
    查看/etc/systemd/system/docker.service.d目录及devicemapper.conf是否存在。如果不存在，就新建
    ```
    mkdir -p /etc/systemd/system/docker.service.d/
    vi /etc/systemd/system/docker.service.d/devicemapper.conf
    ```
    devicemapper.conf添加如下内容：
    ```
    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd  --graph=/home/modules/docker/lib/docker
    ```
- 重启docker
    ```
    systemctl daemon-reload
    systemctl restart docker
    systemctl enable docker
    ```
- 确认Docker Root Dir修改是否已经生效
    ```
    [root@iZbp1jcwx7sfb1yrnvpg84Z docker]# docker info
    ...
    Docker Root Dir: /home/modules/docker/lib/docker
    Debug Mode (client): false
    Debug Mode (server): false
    Registry: https://index.docker.io/v1/
    ...
    ```
- 启动成功后，再确认之前的镜像是否还在
    ```
    [root@iZbp1jcwx7sfb1yrnvpg84Z docker]# docker images
    REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
    10.80.177.233/policy                2.1.2               64ac4e178cd2        2 hours ago         818 MB
    10.80.177.233/crm                   2.1.3               d7636fbb7a29        2 hours ago         762 MB
    ```
- 确定容器没问题后删除/var/lib/docker/目录中的文件

参考链接：https://blog.csdn.net/weixin_32820767/article/details/81196250