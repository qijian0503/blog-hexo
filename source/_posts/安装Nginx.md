---
title: 安装Nginx
date: 2018-09-20 13:17:40
categories: nginx
tags: [nginx]
---

安装前一定要注意当前centos系统的版本，只有版本一一对应才能安装成功
简书地址：https://www.jianshu.com/u/cfd6f3a8ae30
<!--more-->

### 下载对应当前系统版本的nginx包(package)
```
wget http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

### 建立nginx的yum仓库

```
rpm -ivh nginx-release-centos-7-0.el7.ngx.noarch.rpm
```


### 下载并安装nginx

```
yum install nginx
```

### 启动nginx服务

```
systemctl start nginx
```
![clipboard.png](https://upload-images.jianshu.io/upload_images/8760038-778894b7cd166229.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



该错误是提示，当前80端口已经被占用，不用重复绑定该端口。
这里可能是因为之前tomcat已经启动了，占用了80端口了，需要关闭之前的80端口。
在重新启动nginx

### 查看nginx状态

```
systemctl status nginx.service
```
![查看nginx状态](https://upload-images.jianshu.io/upload_images/8760038-da26a355b5e9d939.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 配置
默认的配置文件在 /etc/nginx 路径下，使用该配置已经可以正确地运行nginx；如需要自定义，修改其下的 nginx.conf 等文件即可。

### 更新conf文件后直接reload，不用重启

```
nginx -s reload
```

### 测试
在浏览器地址栏中输入部署nginx环境的机器的IP，如果一切正常，应该能看到如下字样的内容。
![QQ图片20160419185608.jpg](https://upload-images.jianshu.io/upload_images/8760038-cc765baf135ad1bc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

访问如果页面出现：An error occurred
则进入 /var/log/nginx/ 查看error.log

