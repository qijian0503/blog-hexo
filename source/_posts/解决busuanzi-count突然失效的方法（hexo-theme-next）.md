---
title: 解决busuanzi_count突然失效的方法（hexo-theme-next）
date: 2018-10-09 19:35:17
categories: hexo
tags: [hexo]
---

> 2018-09月份的时候，还正常的使用不蒜子统计功能（不蒜子统计功能配置文件已经配置好:themes\next\_config.yml）。过完国庆后，看自己的博客突然不蒜子统计失效了，没有统计数量了。
### 说明
我这里是使用的**hexo-theme-next**主题，主题版本为：5.1.4

<!-- more -->

### 原因分析
由于定位到是[不蒜子](http://busuanzi.ibruce.info/)统计功能突然有问题了，所以前往[不蒜子官网](http://busuanzi.ibruce.info/)进行查看，发现官网有一段很重要的提示：   
**“因七牛强制过期『dn-lbstatics.qbox.me』域名，与客服沟通无果，只能更换域名到『busuanzi.ibruce.info』！”**  
所以定位到问题，原来是不蒜子使用的七牛的域名被强制过期。  
需要把 dn-lbstatics.qbox.me 域名更换为 busuanzi.ibruce.info 

### 解决方案
hexo-theme-next主题中使用了dn-lbstatics.qbox.me域名的文件位置为：   
`themes\next\layout\_third-party\analytics\busuanzi-counter.swig`   

修改busuanzi-counter.swig
```
找到如下代码：
<script async src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
修改为：
<script async src="https://busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

重新预览，即可看到不蒜子统计功能已经生效    

参考链接：  
http://ibruce.info/2015/04/04/busuanzi/