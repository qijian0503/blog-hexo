---
title: kubernetes集群的安装异常汇总
date: 2018-10-12 00:17:29
categories: kubernetes
tags: [kubernetes]
---
> kubernetes集群二进制文件安装方式过程中，出现的异常汇总

<!-- more -->

### 异常【kubelet cgroup driver：cgroupfs跟docker cgroup driver：systemd不一致】
- 异常描述
    >  error: failed to run Kubelet: failed to create kubelet: misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"
    
    启动kubelet时
    ```
    #启动kubelet
    service kubelet start
    #查看kubelet日志
    journalctl -f -u kubelet
    ```
    提示如下错误
    ```
    10月 11 20:05:18 server03 kubelet[15984]: error: failed to run Kubelet: failed to create kubelet: misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"
    10月 11 20:05:18 server03 systemd[1]: kubelet.service: main process exited, code=exited, status=1/FAILURE
    10月 11 20:05:18 server03 systemd[1]: Unit kubelet.service entered failed state.
    10月 11 20:05:18 server03 systemd[1]: kubelet.service failed.
    
    ```
- 原因分析  
    kubelet文件驱动默认cgroupfs, 而我们安装的docker使用的文件驱动是systemd, 造成不一致, 导致镜像无法启动。  
    现在有两种方式, 一种是修改docker, 另一种是修改kubelet。
    我这里采用修改docker的方式  
    ==注意==：  
    网上大部分教程都是说直接修改`daemon.json`
    ```
    #修改daemon.json
    vi /etc/docker/daemon.json
    #添加如下属性
    "exec-opts": [
        "native.cgroupdriver=systemd"
    ]
    ```
    这样会导致修改后，docker无法启动成功，提示`daemon.json`与`/lib/systemd/system/docker.service`中`native.cgroupdriver=systemd`重复存在。

- 解决方案（修改docker）
    ```
    # 修改前查看docker Cgroup Driver
    [root@server02 ~]# docker info
    ...
    Server Version: 1.13.1
    Storage Driver: overlay2
     Backing Filesystem: xfs
     Supports d_type: true
     Native Overlay Diff: true
    Logging Driver: journald
    Cgroup Driver: systemd
    ...
    ```
    ```
    # 修改docker.service
    vi /lib/systemd/system/docker.service
    ```
    ```
    找到
    --exec-opt native.cgroupdriver=systemd \
    修改为：
    --exec-opt native.cgroupdriver=cgroupfs \
    ```
    ```
    # 重启docker
    systemctl daemon-reload
    systemctl restart docker
    ```
    ```
    # 修改后查看docker Cgroup Driver
    [root@server03 sysconfig]# docker info
    ...
    Server Version: 1.13.1
    Storage Driver: overlay2
     Backing Filesystem: xfs
     Supports d_type: true
     Native Overlay Diff: true
    Logging Driver: journald
    Cgroup Driver: cgroupfs
    ...
    
    ```
    参考链接：
    http://www.cnblogs.com/hongdada/p/9771857.html

### 异常【Failed to get system container stats for kubelet.service】
- 异常描述  
    > failed to get container info for "/system.slice/kubelet.service": unknown container "/system.slice/kubelet.service"
    
    启动kubelet时
    ```
    service kubelet start
    #查看kubelet日志
    journalctl -f -u kubelet
    ```
    提示如下错误
    ```
    10月 11 19:37:46 server01 kubelet[64872]: E1011 19:37:46.150198   64872 summary.go:92] Failed to get system container stats for "/system.slice/kubelet.service": failed to get cgroup stats for "/system.slice/kubelet.service": failed to get container info for "/system.slice/kubelet.service": unknown container "/system.slice/kubelet.service"
    
    ```
- 解决方案

    ```
    # 修改kubelet.service
    vi /lib/systemd/system/kubelet.service
    ```
    ```
    #在ExecStart位置最后面，添加如下配置
    --runtime-cgroups=/systemd/system.slice \
    --kubelet-cgroups=/systemd/system.slice
    ```
    修改后的/lib/systemd/system/kubelet.service
    ```
    [Unit]
    Description=Kubernetes Kubelet
    Documentation=https://github.com/GoogleCloudPlatform/kubernetes
    After=docker.service
    Requires=docker.service
    
    [Service]
    WorkingDirectory=/var/lib/kubelet
    ExecStart=/opt/modules/kubernetes-bins/kubelet \
      --address=192.168.1.188 \
      --hostname-override=192.168.1.188 \
      --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/imooc/pause-amd64:3.0 \
      --kubeconfig=/etc/kubernetes/kubelet.kubeconfig \
      --network-plugin=cni \
      --cni-conf-dir=/etc/cni/net.d \
      --cni-bin-dir=/opt/modules/kubernetes-bins \
      --cluster-dns=10.68.0.2 \
      --cluster-domain=cluster.local. \
      --allow-privileged=true \
      --fail-swap-on=false \
      --logtostderr=true \
      --v=2 \
      --runtime-cgroups=/systemd/system.slice \
      --kubelet-cgroups=/systemd/system.slice
    #kubelet cAdvisor 默认在所有接口监听 4194 端口的请求, 以下iptables限制内网访问
    ExecStartPost=/sbin/iptables -A INPUT -s 10.0.0.0/8 -p tcp --dport 4194 -j ACCEPT
    ExecStartPost=/sbin/iptables -A INPUT -s 172.16.0.0/12 -p tcp --dport 4194 -j ACCEPT
    ExecStartPost=/sbin/iptables -A INPUT -s 192.168.0.0/16 -p tcp --dport 4194 -j ACCEPT
    ExecStartPost=/sbin/iptables -A INPUT -p tcp --dport 4194 -j DROP
    Restart=on-failure
    RestartSec=5
    
    [Install]
    WantedBy=multi-user.target
    ```
    参考链接：https://www.cnblogs.com/devilwind/p/8862069.html








