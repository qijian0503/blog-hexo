---
title: hexo+github从0到1搭建免费个人博客
date: 2018-09-19 23:48:49
categories: hexo
tags: [hexo]
---
本教程详细记录了使用hexo+github从0到1搭建免费个人博客  
简书地址：https://www.jianshu.com/u/cfd6f3a8ae30
<!--more-->

### 基础环境搭建

- 安装node.js  
node版本：
```
C:\Users\Administrator>node -v
v8.11.1
```

- 安装git  
git版本：
```
C:\Users\Administrator>git --version
git version 2.9.0.windows.1
```

- 选装cnpm  
淘宝cnpm官网。由于npm国内下载速度经常抽风，所以建议安装淘宝的这个镜像；使用方法就是在命令中把npm换成cnpm即可。
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```


### 安装Hexo及其相关插件

- 全局安装Hexo

```
cnpm install hexo-cli -g
```
装完成后输入hexo -v，出现版本信息则表示安装成功。

- 在项目中安装Hexo  
在自定义目录（D:\workspace），新建hexo文件夹，然后输入cd hexo（进入该文件夹），在依次执行如下操作
```
hexo init   #这个时间也会比较长，也有可能要等几分钟，有显示 WARN 也不用管
cnpm install #有显示 WARN 也不用管
```
安装完成之后，D:\workspace\hexo 目录结构是这样的：
![安装 hexo 后的目录结构](https://upload-images.jianshu.io/upload_images/8760038-05ad280306a14530.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 安装hexo的插件
```
cnpm install hexo-server --save     #搭建本地服务器所需插件
cnpm install hexo-deployer-git --save   #使用git方式进行部署博客所需插件
```

### 在本地生成博客静态页面并预览

- 在本地生成静态页面  

```
hexo generate
```
会生成一个存放静态文件的文件夹public，其简写形式为hexo g；  

- 启动本地服务器
```
hexo server
```
其简写形式为hexo s；  
这条指令运行完成后可在本地启动服务器并预览博客，默认网址为：http://localhost:4000/  
如果以上步骤都不出意外的话，你就会看到一个Hexo博客初始化的页面。
![image.png](https://upload-images.jianshu.io/upload_images/8760038-caf38fde02b58852.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 创建Github pages并SSH授权
- 创建仓库  
首先要有自己的Github账号，没有的可以到[GitHub官网](https://github.com/)注册账号，注册完后，我们来下一步，在我们的GitHub上面右上角的New repository来创建一个仓库。 
![image.png](https://upload-images.jianshu.io/upload_images/8760038-0a2df790c7810d21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 仓库名必须遵守相应格式：github_username.github.io， 这样子在访问主页的时候直接用：github_username.github.io就能访问。 

![image.png](https://upload-images.jianshu.io/upload_images/8760038-e416c380ace548c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我这里因为是已经创建了一个仓库了，所以会有提示，然后点Create repository确定创建仓库。

- 本地生成ssh密钥  
创建好仓库之后，要本地生成 SSH 秘钥，方便电脑上的 git 软件好提交内容到 Github上

1. git bash命令行下输入`ssh-keygen -t rsa -C ‘你的邮箱地址’`，然后回车，回车，再回车，一共 3 次回车。  
2. 此时，生成密钥后，在你电脑目录：C:\Users\你的计算机用户名\.ssh 下，会生成两个文件：  
私钥：id_rsa  
公钥：id_rsa.pub  
3. 现在用记事本打开 id_rsa.pub，复制文件中的所有内容
访问：https://github.com/settings/ssh ，点击添加新秘钥(New SSH key)，效果如下图  
Title：自己随便取  
Key：把刚刚复制的都粘贴进来

![image.png](https://upload-images.jianshu.io/upload_images/8760038-d841920bfa16ead0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 本地博客同步到GitHub上
这一步其实就是把本地生成的博客内容（静态页面）放到GitHub新建成的仓库qijian0503.github.io中。

- 编辑博客配置文件: _config.yml  
在hexo根目录（也就是D:\workspace\hexo文件夹）下找到_config.yml文件，把其中的deploy参数（没有的话就按如下格式新建，注意冒号后面一定要有一个空格），修改为（你需要认真看的是含有中文注释的内容）：
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site，这一块区域主要是设置博客的主要说明，需要注意的是：每个冒号后面都是有一个空格，然后再书写自己的内容的
title: qijian技术栈
subtitle: 记录所有技术相关
description: 技术栈
keywords:
author: qijian
language: zh-Hans
timezone:

# URL，这一块一般可以设置的是 url 这个参数，比如我要设置绑定域名的，这里就需要填写我的域名信息
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://geekstar.site
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:
  
# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date
  
# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape

# Deployment
## 这里是重点，这里是修改发布地址，因为我们前面已经加了 SSH 密钥信息在 Github 设置里面了，所以只要我们电脑里面持有那两个密钥文件就可以无需密码地跟 Github 做同步。
## 需要注意的是这里的 repo 采用的是 ssh 的地址，而不是 https 的。分支我们默认采用 master 分支，以后你翅膀硬了要换其他也无所谓。
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:qijian0503/qijian0503.github.io.git
  branch: master

```
- 重新部署  
在博客根目录下打开Git Bash，依次执行如下Hexo命令：

```
hexo clean    #会清除缓存文件db.json及之前生成的静态文件夹public；
hexo g     #会重新生成静态文件夹public；
hexo deploy    #因为之前已经安装了插件并且在博客配置文件中也配置好了，所以这个命令会在博客根目录下生成一个.deploy_git的文件夹，并 把本地生成的静态文件部署到qijian0503.github.io这个仓库中的master分支上；简写形式为hexo d；
```
> hexo g 和 hexo d可以合并在一起写：hexo g -d

执行：hexo deploy，有弹出下面提示框，请输入：yes
![image.png](https://upload-images.jianshu.io/upload_images/8760038-c6e551f5bead1376.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
提交成功效果如下：
![image.png](https://upload-images.jianshu.io/upload_images/8760038-16db68c3ac2775a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 在浏览器中访问博客  
在浏览器中输入 qijian0503.github.io（可能你已经发现了，这个就是之前新建仓库的名字，同时也是你博客的域名），即可以再次看到那个熟悉又亲切的博客页面了。

> 至此，我们已经通过Hexo创建了一个最原始的博客，并且通过把博客静态文件放到GitHub的仓库中，实现了从网上以GitHub的默认域名访问这个博客。接下来要做的就是要锦上添花了：换个好看的主题；自定义博客的域名；操作及优化博客。

### 更换主题
更换主题主要是两步，先下载主题然后放到博客中的themes文件夹（专门用来存放主题）下，再修改主题的配置文件_config.yml中相关参数，启用themes文件夹下新增的主题。这里用Next主题做示例。
1. 下载Next主题。  
进入 hexoBlog/themes 文件夹中，打开Git Bash面板，输入：

```
git clone https://github.com/iissnan/hexo-theme-next themes/next
```
把主题包克隆到themes文件夹中即可。
2. 启用主题  
与所有 Hexo 主题启用的模式一样。 当 克隆/下载 完成后，打开themes下的主题配置文件_config.yml， 找到 theme 字段，并将其值更改为 next（注意冒号后面要留一个 空格）。
3. 验证主题  
清除并重新生成hexo静态文件，启动本地服务器，然后通过 http://localhost:4000/ 预览博客：
```
hexo clean      #清除静态文件
hexo g          #重新生成静态文件
hexo s      #启动服务器
```
> 如果网络没问题，通过域名访问你的博客也可以看到刚换的新主题了。
> 关于更换Next主题的详细介绍，也可访问Next中文官网

### 配置自定义域名  
- 注册域名  
域名注册商可选择阿里云等平台进行注册，.site|.top后缀域名都很便宜。这里就不记录了，大家自信去注册吧。
![image.png](https://upload-images.jianshu.io/upload_images/8760038-9787b2b10823484d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 新建CNAME文件  
> 新建一个CNAME文件（文件名叫 CNAME，没有文件后缀的），把该文件放在 D:\workspace\hexo\source 目录下  
CNAME 上的内容需要写你具体要绑定的域名信息

![image.png](https://upload-images.jianshu.io/upload_images/8760038-96202cb6156174e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
依次执行
```
hexo clean 
hexo g 
hexo d
```
- 域名解析设置  
查看github空间服务IP：`ping www.qijian0503.github.io`
![image.png](https://upload-images.jianshu.io/upload_images/8760038-390a27d8031aa99a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
登录阿里云【域名-点击域名列表中的域名-域名解析】进入域名的解析后台，添加如下两条解析记录：  
![image.png](https://upload-images.jianshu.io/upload_images/8760038-dda3280b2a7ec4ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
设置好之后，等过几分钟域名解析好之后，我们访问：http://www.geekstar.site ，即可通过域名访问到你的博客了。

### 博客操作
- 新建文章  

```
hexo new post 背影
```

执行上面命令会新建一篇名为‘背影’的文章，源文件会自动生成到hexoBlog/source/_post 路径下，后缀为点md ，直接打开编辑就可以了。编辑完保存，然后再依次执行
```
hexo clean 
hexo g 
hexo d
```
在博客就可以看到你的文章了（有时候网络问题生成会比较慢，需要等几分钟才可以看到）。

- 新建页面  
新建标签、分类、关于我等各种页面，并在博客的菜单栏中显示。这里以新建‘标签’页面来做示范。
1. 创建页面

```
hexo new page 'tags'
```
会在hexoBlog/source路径下自动生成一个名为tags的文件夹，里面包含一个index.md的文件，在这个文件中添加对应的页面类型type: tags：
```
---
title: tags
date: 2018-09-16 16:20:05
type: "tags"
---
```
2. 把页面路径添加到菜单中。  
编辑主题配置文件（themes/_config.yml）,找到munu字段，添加tag: /tags（格式为item_name: link），如下：
```
menu:
    home: /
    tag: /tags    #‘标签’’页面的路径
```

参考链接：  
https://blog.csdn.net/weixin_39345384/article/details/80787998  
https://blog.csdn.net/qq_32454537/article/details/79482896  
https://www.jianshu.com/p/21c94eb7bcd1  
https://blog.csdn.net/linshuhe1/article/details/52424573  
https://www.jianshu.com/p/1c98aed8d92e  
https://www.jianshu.com/p/ea5ac6162a96  
https://blog.csdn.net/u011976726/article/details/78217467  
https://www.cnblogs.com/wanghuaijun/p/7073296.html  
https://www.cnblogs.com/zhangqie/p/7978394.html