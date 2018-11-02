---
title: hexo博客添加搜索功能
date: 2018-09-29 17:24:01
categories: hexo
tags: [hexo]
---

### 安装插件
在自己博客根目录下（我的目录：D:\workspace\hexo），执行如下命令
```
cnpm install hexo-generator-searchdb --save
```

<!-- more -->

### 修改站点配置文件
修改根目录下的_config.yml（我的目录：D:\workspace\hexo\_config.yml），在最底部添加如下配置
```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```
![_config.yml](https://upload-images.jianshu.io/upload_images/8760038-6b393fa5d79c1e3c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 修改主题配置文件
修改主体下的themes\next\_config.yml配置文件（我的目录：D:\workspace\hexo\themes\next\_config.yml），搜索local_search，修改enable为true
```
local_search:
  enable: true
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1

```

### 预览效果
开启本地server
```
hexo clean
hexo g
hexo s
```
访问：http://localhost:4000/ ，即可看到想要的搜索功能了  
![搜索功能](https://upload-images.jianshu.io/upload_images/8760038-f806bc2c515c2cd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![搜索弹窗](https://upload-images.jianshu.io/upload_images/8760038-e2e3adc32911b9c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

