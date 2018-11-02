---
title: hexo theme-next主题设置
date: 2018-09-20 13:49:52
categories: hexo
tags: [hexo]
---

### 前言
> 由于hexo已经搭建好，并且是用的next主体。这里主要介绍下，next主题相关的一些优化配置。

读者可以在 https://hexo.io/themes/ 可以查看你喜欢的主题。 这里主要介绍NexT主题的相关配置。其他主题可以多看看官方文档。
<!--more-->

### 安装主题
安装的过程就一行代码，你需要在博客根目录出打开命令行输入以下命令：

```
git clone https://github.com/theme-next/hexo-theme-next themes / next
```
### 启用主题
修改站点配置文件_config.yml（D:\workspace\hexo\_config.yml）
```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```


**选择 Scheme：**   
Scheme 是 NexT 提供的一种特性，借助于 Scheme，NexT 为你提供多种不同的外观。同时，几乎所有的配置都可以 在 Scheme 之间共用。目前 NexT 支持三种 Scheme：  
- **Muse** - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白  
- **Mist** - Muse 的紧凑版本，整洁有序的单栏外观  
- **Pisces** - 双栏 Scheme，小家碧玉似的清新  
- **Gemini** - 左侧网站信息及目录，块+片段结构布局      

修改站点配置文件_config.yml（D:\\workspace\\hexo\\themes\\next\\_config.yml）   
scheme的切换通过更改主题配置文件，搜索scheme关键字。你会看到有四行scheme   的配置，将你需用启用的scheme 前面注释 # 去除即可。 
```
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------

# Schemes
#scheme: Muse
scheme: Mist
#scheme: Pisces
#scheme: Gemini
```

### 设置语言
编辑站点配置文件， 将 language 设置成你所需要的语言。建议明确设置你所需要的语言，例如选用简体中文，配置如下：

```
language: zh-Hans
```

### 主页文章加阴影
打开\themes\next\source\css\_custom\custom.styl,向里面加入：

```
// 主页文章添加阴影效果
.post {
margin-top: 60px;
margin-bottom: 60px;
padding: 25px;
-webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
-moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
}
```
### 设置网站图标
默认的网站图标是一个N，当然是需要制定一个图了，在网上找到图后，将其放在/themes/next/source/images里面，然后在主题配置文件中修改下图所示图片位置

```
favicon:
  #small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next-docker.jpg
  #apple_touch_icon: /images/apple-touch-icon-next.png
  #safari_pinned_tab: /images/logo.svg
  #android_manifest: /images/manifest.json
  #ms_browserconfig: /images/browserconfig.xml
```

### 设置侧边栏头像
编辑主题的 \themes\next\_config.yml，新增字段 avatar， 值设置成头像的链接地址。

```
avatar: /images/java.jpg
```
### 设定代码高亮主题
NexT使用Tomorrow Theme作为代码高亮，共有5款主题供你选择:
normal | night | night eighties | night blue | night bright,默认使用的是白色的normal  
编辑站点的_config.yml：
```
highlight:
  enable: true
  line_number: true
  auto_detect: true
  tab_replace:
```
编辑主题的\themes\next\_config.yml：
```
highlight_theme: night
```
### 页面访客统计
> 在使用该配置之前，你需要先确保自己使用的Hexo博客的NexT主题。旧版的NexT主题可能不支持改配置，在进行进一步的操作之前，确保自己使用的NexT版本支持对应功能。在这里，我使用的版本为5.1.4，你可以通过查看/theme/next/_config.yml文件搜索“version”来确认自己的NexT版本。

- 页面底部总访客|总访问量配置    
打开/theme/next/_config.yml，找到如下的配置项：
```
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  site_uv_header: <i class="fa fa-user"></i>总访客
  site_uv_footer: 人
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: <i class="fa fa-eye"></i>总访问量
  site_pv_footer: 次
  # custom pv span for one page only
  page_pv: true
  page_pv_header: <i class="fa fa-file-o"></i>
  page_pv_footer:
```
将enable的值由false修改为true后，重新部署即可看到效果。我这里添加了：总访客、总访问量对应的汉字描述。  
在你完成部署后，本地预览可能你会发现，自己网站的PV数和UV数都非常大，如下所以：
![访问量](https://upload-images.jianshu.io/upload_images/8760038-1068b5c5e79142e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这是正常情况，因为使用不蒜子统计的用户都使用同一个存储空间，如果你的URL和别人重复，就会出现数据量异常。这样的情况一般出现在你使用localhost:4000访问自己在本地部署的网页的时候。`hexo d`部署后，通过域名访问既不会出现这样的情况。  
参考链接：https://lfwen.site/2016/11/13/next-busuanzi-vistor-count

- 百度统计  
> 配置好后，到百度统计管理系统中不会立刻有效果，需要等待一会，才会看到如下效果

![百度统计](https://upload-images.jianshu.io/upload_images/8760038-c8924e05194c0039.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


参考链接：https://blog.csdn.net/u013066244/article/details/71056834

### 阅读次数
使用[leancloud](https://leancloud.cn/)实现，效果如下：  
![阅读次数](https://upload-images.jianshu.io/upload_images/8760038-2d94f66c344f52a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参考链接：https://lfwen.site/2016/05/31/add-count-for-hexo-next/


