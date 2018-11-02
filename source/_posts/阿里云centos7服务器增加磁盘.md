---
title: 阿里云centos7服务器增加磁盘
date: 2018-09-28 17:38:50
categories: linux
tags: [linux]
---
### 前言
> 阿里云云服务器现在基本已成主流的云服务器了。我使用的是阿里云centos7。在使用过程中，难免会遇到初始化的磁盘空间不够用，这时候比较好的方案就是扩容服务器磁盘空间创建云盘了。因为如果重新格式化系统盘在重装，那代价还是挺大的。

<!-- more -->

### 步骤

- 创建云盘  
    > 创建的云盘，只能采用按量付费方式计费，而且只能作数据盘用。 
    
    具体创建云盘的步骤，这里就不记录了，挺简单。  
    具体操作步骤，查看官方教程：https://help.aliyun.com/document_detail/25445.html?spm=a2c4g.11186623.6.679.46671846M2ayt0

- 挂载云盘
    > 将单独创建的云盘（作数据盘用）挂载到ECS实例上。您可以选择从实例管理页面挂载云盘，也可以从云盘管理页面挂载云盘。云盘只能挂载到同一地域下同一可用区内的实例上，不能跨可用区挂载。

    具体操作步骤，查看官方教程：https://help.aliyun.com/document_detail/25446.html?spm=a2c4g.11186623.6.681.9f343029XcxOt4
- 格式化和挂载数据盘
    > 这里注意，按照官方教程格式化数据盘后，挂载文件系统时，挂载的目录指定成需要自定义挂载的目录。  
    因为默认是挂载到/mnt目录的 `mount /dev/vdb1 /mnt`

    我这里挂载的目录是/home
    ```
    [root@iZbp1jcwx7sfb1yrnvpg84Z ~]# df -h
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda1        99G   42G   53G  45% /
    devtmpfs        7.8G     0  7.8G   0% /dev
    tmpfs           7.8G     0  7.8G   0% /dev/shm
    tmpfs           7.8G  2.8M  7.8G   1% /run
    tmpfs           7.8G     0  7.8G   0% /sys/fs/cgroup
    tmpfs           1.6G     0  1.6G   0% /run/user/0
    /dev/vdb1       197G   27G  161G  15% /home
    
    ```
    具体操作步骤，查看官方教程：https://help.aliyun.com/document_detail/25426.html?spm=a2c4g.11186623.2.19.65391846Wl8rWe#concept_jl1_qzd_wdb

