---
title: docker常用命令
date: 2018-09-20 12:26:10
categories: docker
tags: [docker]
---

# docker常用命令
**启动docker**
```
systemctl start docker
```

**查看docker启动状态**

```
systemctl status docker
```
<!--more-->
**查看镜像**

```
docker images
```

**查看所有容器包括停止运行的**

```
docker ps -a
```

**启动所有容器**
```
docker start $(docker ps -a -q)
```

**停止容器**
```
docker stop gitlab-redis
```

**删除所有已经停止的容器**

```
docker rm $(docker ps -a -q)
```

**删除镜像**  

1. 需要先停止关联的容器  
2. 删除该容器
> docker rm 容器ID（CONTAINER ID）

3. 删除镜像  
> docker rmi 镜像ID（IMAGE ID）    
查看：https://blog.csdn.net/winy_lm/article/details/77980529 
 
4. 批量删除名字包含"none"关键字的镜像  
> docker rmi $(docker images | grep "none" | awk '{print $3}') 

**查看指定容器log**
```
docker logs 容器名称(NAMES)
```

---

# docker监控容器使用资源的情况
**查看容器使用的资源情况**  

docker stats 命令用来显示容器使用的系统资源。不带任何选项执行 docker stats 命令：
```
docker stats
```
![image.png](https://upload-images.jianshu.io/upload_images/8760038-6a6713dfbd47d6d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
默认情况下，stats 命令会每隔 1 秒钟刷新一次输出的内容直到你按下 ctrl + c。下面是输出的主要内容：  
[CONTAINER]：以短格式显示容器的 ID。  
[CPU %]：CPU 的使用情况。  
[MEM USAGE / LIMIT]：当前使用的内存和最大可以使用的内存。  
[MEM %]：以百分比的形式显示内存使用情况。  
[NET I/O]：网络 I/O 数据。  
[BLOCK I/O]：磁盘 I/O 数据。   
[PIDS]：PID 号。  

**只返回当前的状态**  

如果不想持续的监控容器使用资源的情况，可以通过 --no-stream 选项只输出当前的状态：  

```
docker stats --no-stream
```
![image.png](https://upload-images.jianshu.io/upload_images/8760038-aae5f414061cc549.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**只输出指定的容器**  
如果我们只想查看个别容器的资源使用情况，可以为 docker stats 命令显式的指定目标容器的名称或者是 ID：

```
docker stats --no-stream registry 1493
```
当有很多的容器在运行时，这样的结果看起来会清爽一些。这里的 registry 和 1493 分别是容器的名称和容器的 ID。注意，多个容器的名称或者是 ID 之间需要用空格进行分割

**显示容器名称**  

```
docker stats $(docker ps --format={{.Names}})
```
**显示容器名称，且只返回当前的状态**  

```
docker stats $(docker ps --format={{.Names}})
```
![image.png](https://upload-images.jianshu.io/upload_images/8760038-d82e7ae32352a1d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查看链接：https://www.cnblogs.com/sparkdev/p/7821376.html