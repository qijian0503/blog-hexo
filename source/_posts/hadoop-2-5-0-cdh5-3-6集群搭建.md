---
title: hadoop-2.5.0-cdh5.3.6集群搭建
date: 2018-09-22 17:53:02
categories: [hadoop]
tags: [hadoop,hdfs,yarn]
---
### 安装环境说明
- 操作系统：CentOS 7  
- hadoop版本：hadoop-2.5.0-cdh5.3.6.tar.gz  
- jdk版本：jdk 1.7
- 安装用户：root  
- 相关软件下载  
    https://pan.baidu.com/s/1drI1TO

<!-- more -->

### 机器与服务规划
- 机器规划  

hostname | sparkproject1 | sparkproject2 | sparkproject3
---|---|---|---|
内存 | 	32G | 16G | 16G
- 服务规划  

hostname | sparkproject1 | sparkproject2 | sparkproject3
---|---|---|---|
HDFS | 	NameNode、SecondaryNameNode	 | DataNode  | DataNode 
YARN | 	ResourceManager	 | NodeManager   | NodeManager 

### 安装hadoop包
- 下载[hadoop-2.5.0-cdh5.3.6.tar.gz](http://archive.cloudera.com/cdh5/cdh/5/)  
安装目录为：/usr/local  
将下载的hadoop-2.5.0-cdh5.3.6.tar.gz，上传到/usr/local目录下。

- 将hadoop包进行解压缩
    ```
    tar -zxvf hadoop-2.5.0-cdh5.3.6.tar.gz
    ```

- 对hadoop目录进行重命名
    ```
    mv hadoop-2.5.0-cdh5.3.6 hadoop
    ```

- 配置hadoop相关环境变量
    ```
    #配置环境变量
    vi ~/.bashrc
    #添加hadoop环境变量
    export HADOOP_HOME=/usr/local/hadoop
    export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
    #使配置的环境变量生效
    source ~/.bashrc
    ```
- 创建/usr/local/data目录
    ```
    mkdir /usr/local/data
    ```
- 测试是否配置成功
    ```
    hadoop version
    yarn version
    ```
    ![hadoop版本](https://upload-images.jianshu.io/upload_images/8760038-f022e337b2148a50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 修改配置文件
> 以下配置文件在：/usr/local/hadoop/etc/hadoop/目录下

- 修改core-site.xml
    ```
    <property>
      <name>fs.default.name</name>
      <value>hdfs://sparkproject1:9000</value>
    </property>
    ```
    ![core-site.xml](https://upload-images.jianshu.io/upload_images/8760038-9fa6a77b5aae51ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    > 属性说明：  
    fs.default.name：配置hdfs地址

- 修改hdfs-site.xml
    ```
    <property>
      <name>dfs.name.dir</name>
      <value>/usr/local/data/namenode</value>
    </property>
    <property>
      <name>dfs.data.dir</name>
      <value>/usr/local/data/datanode</value>
    </property>
    <property>
      <name>dfs.tmp.dir</name>
      <value>/usr/local/data/tmp</value>
    </property>
    <property>
      <name>dfs.replication</name>
      <value>2</value>
    </property>
    ```
    ![hdfs-site.xml](https://upload-images.jianshu.io/upload_images/8760038-abb3ed27712ddd73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    > 属性说明：  
    dfs.replication：hdfs副本数。  
    总共3个节点，1个master，2个slave。所以设置成2个block副本

- 修改mapred-site.xml  
    重命名mapred-site.xml.template为mapred-site.xml
    ```
    <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property>
    ```
    ![mapred-site.xml](https://upload-images.jianshu.io/upload_images/8760038-d0976165d7cfdf25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 修改yarn-site.xml
    ```
    <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>sparkproject1</value>
    </property>
    <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
    </property>
    ```
    ![yarn-site.xml](https://upload-images.jianshu.io/upload_images/8760038-070899cc3fe50714.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 修改slaves文件
    ```
    sparkproject2
    sparkproject3
    ```
    ![slaves](https://upload-images.jianshu.io/upload_images/8760038-b0a6ac3e1f857904.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 在另外两台机器上搭建hadoop
> 使用如上sparkproject1上配置hadoop，在另外两台机器上搭建hadoop。可以使用scp命令将sparkproject1上面的hadoop安装包和~/.bashrc配置文件都拷贝到sparkproject2、sparkproject3。

- 将sparkproject1上的hadoop复制到sparkproject2  
  - 在sparkproject1上执行
    ```
    cd /usr/local
    scp -r hadoop root@sparkproject2:/usr/local
    scp ~/.bashrc root@sparkproject2:~/
    ```
    复制成功后sparkproject2上的hadoop：  
    ![sparkproject2上的hadoop](https://upload-images.jianshu.io/upload_images/8760038-2f3afce424febfd0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  - 在sparkproject2上执行
    
    ```
    #对.bashrc文件进行source，以让它生效。
    source ~/.bashrc
    #创建data目录。
    mkdir /usr/local/data
    ```


- 将sparkproject1上的hadoop复制到sparkproject3  
    按照上面同样的步骤，同样的方式将sparkproject1上面的hadoop安装包和~/.bashrc配置文件都拷贝到sparkproject3。


- 测试sparkproject2、sparkproject3是否配置成功  
    在sparkproject2、sparkproject3分别执行如下命令：
    ```
    hadoop version
    yarn version
    ```
    ![hadoop、yarn版本查看](https://upload-images.jianshu.io/upload_images/8760038-715bbd75016534b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 启动hdfs集群
- 格式化namenode  
    在sparkproject1上执行以下命令
    ```
    hdfs namenode -format
    ```

- 启动hdfs集群
    ```
    start-dfs.sh
    ```
    ![启动hdfs集群.png](https://upload-images.jianshu.io/upload_images/8760038-a9910c5c75d83ef6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 验证启动是否成功  
    sparkproject1：namenode、secondarynamenode  
    ![sparkproject1-jps.png](https://upload-images.jianshu.io/upload_images/8760038-2c0000b401b1ecf4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
    sparkproject2：datanode    
    ![sparkproject2-jps.png](https://upload-images.jianshu.io/upload_images/8760038-284eac1b07848611.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
    sparkproject3：datanode  
    ![sparkproject3-jps.png](https://upload-images.jianshu.io/upload_images/8760038-9d1ac859b6faf575.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- hdfs管理界面：  
    http://sparkproject1:50070
    ![hdfs管理界面.png](https://upload-images.jianshu.io/upload_images/8760038-75aa4b4f5ab9ae1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 测试hdfs  
    ```
    hdfs dfs -put hello.txt /hello.txt
    ```

### 启动yarn集群
- 启动yarn集群
    ```
    start-yarn.sh
    ```
    ![启动yarn集群.png](https://upload-images.jianshu.io/upload_images/8760038-8b053e93821aa127.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 验证启动是否成功  
    sparkproject1：resourcemanager  
    ![sparkproject1-yarn-jps.png](https://upload-images.jianshu.io/upload_images/8760038-b7635193ea5a8587.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
    sparkproject2：nodemanager  
    ![sparkproject2-yarn-jps.png](https://upload-images.jianshu.io/upload_images/8760038-b5c30945a766cdd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
    sparkproject3：nodemanager  
    ![sparkproject3-yarn-jps.png](https://upload-images.jianshu.io/upload_images/8760038-af9cfaaccdfea60c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- yarn管理界面  
    http://sparkproject1:8088/
    ![yarn管理界面.png](https://upload-images.jianshu.io/upload_images/8760038-2d631aca4e7f311c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

