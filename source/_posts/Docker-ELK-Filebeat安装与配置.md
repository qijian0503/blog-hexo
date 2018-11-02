---
title: Docker ELK+Filebeat安装与配置
date: 2018-09-16 15:58:22
categories: elk
tags: [elk, docker]
---

### 环境说明
- linux  
CentOS Linux release 7.5.1804 (Core)
- docker  
Docker version 1.13.1
- elk  
sebp/elk latest
- filebeat  
filebeat-6.4.0
<!--more-->
> elk跟filebeat在同一台机器上

### 架构

- Elasticsearch  
一个近乎实时查询的全文搜索引擎。Elasticsearch 的设计目标就是要能够处理和搜索巨量的日志数据。

- Logstash  
读取原始日志，并对其进行分析和过滤，然后将其转发给其他组件（比如 Elasticsearch）进行索引或存储。Logstash 支持丰富的 Input 和 Output 类型，能够处理各种应用的日志。

- Kibana  
一个基于 JavaScript 的 Web 图形界面程序，专门用于可视化 Elasticsearch 的数据。Kibana 能够查询 Elasticsearch 并通过丰富的图表展示结果。用户可以创建 Dashboard 来监控系统的日志。

- Filebeat  
引入Filebeat作为日志搜集器，主要是为了解决Logstash开销大的问题。相比Logstash，Filebeat 所占系统的 CPU 和内存几乎可以忽略不计。

> 日志处理流程：  
> Filebeat将日志发送给Logstash进行分析和过滤，然后由Logstash转发给Elasticsearch，最后由Kibana可视化Elasticsearch 的数据

![elk流程图](https://upload-images.jianshu.io/upload_images/8760038-5ce325d59f634541.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 安装 ELK 套件
ELK 的部署方案可以非常灵活，在规模较大的生产系统中，ELK 有自己的集群，实现了高可用和负载均衡。我们的目标是在最短的时间内学习并实践 ELK，因此将采用最小部署方案：在容器中搭建 ELK。


- 运行ELK镜像需要vm.max_map_count至少需要262144内存  

```
切换到root用户修改配置sysctl.conf
vi /etc/sysctl.conf
在尾行添加以下内容   
vm.max_map_count=262144
并执行命令
sysctl -p
```
> elk启动的时候可能会提示如下错误:  
> max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]  
查看容器日志：docker logs 容器ID   
参考链接：https://blog.csdn.net/jiankunking/article/details/65448030


- 安装docker

```
在线安装吧，如果自定义安装请搜索下安装方法，这里就不再描述了
yum install docker   
启用服务
systemctl start docker
开机启动
systemctl enable docker
```


- 运行ELK镜像
```
sudo docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk sebp/elk
```
- 配置logstash  
```
查看容器信息
docker ps -a

进入容器
sudo docker exec -it elk /bin/bash
或
sudo docker exec -it 容器ID /bin/bash

修改02-beats-input.conf
cd /etc/logstash/conf.d/
vi 02-beats-input.conf
```
/etc/logstash/conf.d/02-beats-input.conf修改成如下图所示：  
> 这里vi命令使用有点问题，我是通过DEL键一行一行的删掉了那3行的

![image.png](https://upload-images.jianshu.io/upload_images/8760038-4a5ad23aff2691ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将以下三行删除掉。这三行的意思是是否使用证书，本例是不使用证书的，如果你需要使用证书，将logstash.crt拷贝到客户端，然后在filebeat.yml里面添加路径即可

```
ssl => true 
ssl_certificate => "/pki/tls/certs/logstash.crt"
ssl_key => "/pki/tls/private/logstash.key"
```
注意：sebp/elk docker是自建立了一个证书logstash.crt，默认使用*通配配符，如果你使用证书，filebeat.yml使用的服务器地址必须使用域名，不能使用IP地址，否则会报错

> 这里如果不去掉这三行配置的话，在后面启动filebeat时，会提示如下错误：
```
2018-09-12T10:01:29.770+0800	ERROR	logstash/async.go:252	Failed to publish events caused by: lumberjack protocol error
2018-09-12T10:01:29.775+0800	ERROR	logstash/async.go:252	Failed to publish events caused by: client is not connected
2018-09-12T10:01:30.775+0800	ERROR	pipeline/output.go:109	Failed to publish events: client is not connected
```


- 重启elk容器
```
docker restart 容器ID
```

- kibana可视化页面  

在浏览器输入：http://ip:5601 ，稍等一会即可看到kibana启动成功管理页面
![image.png](https://upload-images.jianshu.io/upload_images/8760038-a5cf52ea8fa0dc8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Elasticsearch的JSON接口：http://[Host IP]:9200/_search?pretty


### 安装Filebeat
> filebeat有多种安装方式，我这里采用rpm包的安装方式，可自动注册为systemd的服务

- 下载filebeat的rpm包
```
cd /opt/softwares
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.4.0-x86_64.rpm
```
或者到官网查看最新版本直接下载：https://www.elastic.co/downloads/beats/filebeat
![image.png](https://upload-images.jianshu.io/upload_images/8760038-0a2f8359c871ff4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 安装filebeat

```
rpm -ivh filebeat-6.4.0-x86_64.rpm
```

- 配置filebeat

```
cd /etc/filebeat
vi filebeat.yml
```
配置改成如下所示：  

```
#=========================== Filebeat inputs =============================

filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /opt/datas/logs/*/*.log
  tags: ["测试环境"]
  multiline:
    pattern: '^\s*(\d{4}|\d{2})\-(\d{2}|[a-zA-Z]{3})\-(\d{2}|\d{4})'
    # pattern: '^\s*("{)'
    negate: true
    match: after
    max_lines: 1000
    timeout: 30s
```
> enabled：filebeat 6.0后，enabled默认为关闭，必须要修改成true  
paths：为你想要抓取分析的日志所在路径  
> multiline：如果不进行该合并处理操作的话，那么当采集的日志很长或是像输出xml格式等日志，就会出现采集不全或是被分割成多条的情况  
> pattern：配置的正则表达式，指定匹配的表达式（匹配以 2017-11-15 08:04:23:889 时间格式开头的字符串），如果匹配不到的话，就进行合并行。  
[参考链接](https://www.cnblogs.com/xishuai/p/spring-boot-log4j2-and-elk-logstash-filebeat.html/)

![image.png](https://upload-images.jianshu.io/upload_images/8760038-d21dcc3717f47e10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 配置改为上图所示。  
注释掉Elasticsearch output，开启Logstash output。   
hosts：elk所在机器IP地址  
如果直接将日志发送到Elasticsearc，请编辑此行：Elasticsearch output  
如果直接将日志发送到Logstash，请编辑此行：Logstash output  
只能使用一行输出，其它的注掉即可 

- 启动filebeat服务

```
启动filebeat
systemctl start filebeat.service
查看filebeat状态
systemctl status filebeat.service
查看filebeat日志
tail -f /var/log/filebeat/filebeat
```
参考链接：https://www.jianshu.com/p/7ca38fa881ae


### kibana配置  

点击左上角的Discover按钮，如下图所示，提示创建“index pattern”： 
![image.png](https://upload-images.jianshu.io/upload_images/8760038-270f3c4268a3e1af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如下图，红框中输入filebeat-*，再点击Next step: 
![image.png](https://upload-images.jianshu.io/upload_images/8760038-88d0a8e129db4f05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如下图，下拉框中选择@timestamp，再点击Create index pattern
![image.png](https://upload-images.jianshu.io/upload_images/8760038-a38009e67b46926f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在弹出的页面上，再次点击左上角的Discover按钮，然后点击右上角的Last 15 minutes，如下图： 
![image.png](https://upload-images.jianshu.io/upload_images/8760038-61c7928fceea6c5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此时页面上会显示最近15分钟内的日志，如果最近15分钟内没有任何日志上报，您也可以点击下图红框中的Today按钮，展示今天的所有日志： 

![image.png](https://upload-images.jianshu.io/upload_images/8760038-39d5b4988c00d106.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


参考链接：  
https://blog.csdn.net/qq_39284787/article/details/78809538  
https://blog.csdn.net/boling_cavalry/article/details/79836171  
https://www.cnblogs.com/CloudMan6/p/7787870.html  
https://blog.csdn.net/boling_cavalry/article/details/79950677
