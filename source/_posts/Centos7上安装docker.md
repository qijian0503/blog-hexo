---
title: Centos7上安装docker
date: 2018-09-27 23:57:52
categories: docker
tags: [docker]
---

### docker安装
- 在线安装  
    ```
    yum install docker
    ```
- 启用服务
    ```
    systemctl start docker
    ```
- 开机启动  
    ```
    systemctl enable docker
    ```
    查看链接：  
    https://www.cnblogs.com/yufeng218/p/8370670.html

<!-- more -->

### 常见问题  
- docker运行镜像的时候，提示：docker-runc not installed on system
    ```
    [root@iZbp1jcwx7sfb1yrnvpg84Z docker]# docker run hello-world
    shell-init: error retrieving current directory: getcwd: cannot access parent directories: No such file or directory
    Unable to find image 'hello-world:latest' locally
    Trying to pull repository docker.io/library/hello-world ... 
    sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788: Pulling from docker.io/library/hello-world
    d1725b59e92d: Pull complete 
    Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
    Status: Downloaded newer image for docker.io/hello-world:latest
    /usr/bin/docker-current: Error response from daemon: shim error: docker-runc not installed on system.
    
    ```
    解决方案：  
    ```
    [root@iZbp1jcwx7sfb1yrnvpg84Z docker]# cd /usr/libexec/docker/
    [root@iZbp1jcwx7sfb1yrnvpg84Z docker]# ln -s docker-runc-current docker-runc
    ```
    参考链接：https://www.cnblogs.com/cxbhakim/p/8862758.html
    
- docker运行镜像的时候，提示：exec: "docker-proxy": executable file not found in $PATH   
    ```
    [root@iZbp1jcwx7sfb1yrnvpg84Z docker]# docker run -d -p 5000:5000 -v /home/datas/registry:/var/lib/registry registry
    acae4d30d40cdb209d86377505bab5215ef37c66a863033b16909719e1c76a53
    /usr/bin/docker-current: Error response from daemon: driver failed programming external connectivity on endpoint agitated_engelbart (7f225ce7048177e8ee4c5d77161bcf9d8bcc2597ea8307d1a10bcc3c9b20a4d3): exec: "docker-proxy": executable file not found in $PATH.
    
    ```
    解决方案：  
    ```
    [root@iZbp1jcwx7sfb1yrnvpg84Z bin]# cd /usr/libexec/docker/
    [root@iZbp1jcwx7sfb1yrnvpg84Z docker]# ln -s docker-proxy-current docker-proxy
    ```
    参考链接：https://blog.csdn.net/hellofyy/article/details/80091635

- docker运行镜像的时候，提示：Bind for 0.0.0.0:5000 failed: port is already allocated  
    > 调试这个问题花费了好长时间，因为无法通过netstat以及lsof看到究竟是什么应用占用了程序；后来我才发现原来是因为docker的原因；如果docker被run了两次
    
    ```
    [root@iZbp1jcwx7sfb1yrnvpg84Z docker]# docker run -d -p 5000:5000 -v /home/datas/registry:/var/lib/registry registry
    d6e57a3ed2bad6671df31acdc83cd32b9b6a136d2874ed34b77c99e37a858177
    /usr/bin/docker-current: Error response from daemon: driver failed programming external connectivity on endpoint ecstatic_goldstine (2521adff22666e908e773d6288ca03a30229aade2a1d68e8fcb250254fdc3353): Bind for 0.0.0.0:5000 failed: port is already allocated.
    ```
    第一次失败，那么这个端口将会被一直占用，即使docker容器并没有创建。  
    解决方案：  
    ```
    #重启docker服务
    service docker restart
    ```
    也有人说要sudo rm /var/lib/docker/network/files/local-kv.db，但是在我看来并不需要  
    参考链接：https://www.cnblogs.com/xiashiwendao/p/7859815.html
- 删除镜像时，提示：image is referenced in multiple repositories  
解决方案：  
删除时可以用repository和tag的方式来删除  
参考链接：https://blog.csdn.net/u013258415/article/details/80051082

