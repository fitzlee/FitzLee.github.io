---
title: Hexo Next主题
urlname: tools_hexo_new
date: 2013-09-05 22:07:43
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
#photos:
#  - >-
#   https://ss0.bdstatic.com/5aV1bjqh_Q23odCf/static/superplus/img/logo_white_ee663702.png
categories: 22tools
tags:
    - hexo
    - 博客
---


这里简要做介绍，深入了解可以参考:[官方文档](http://hexo.io/docs/)


## hexo next 安装
1. 首先安装node npm
2. 然后安装hexo
   npm install -g hexo-cli
   -g一般用在安装可以直接全局执行的程序。
   其他依赖，直接放在package.json运行npm install安装在本地即可，npm install xxx --save可以把xxx这个软件save到package.json得dependency里边，下次直接npm install就可以安装所有软件。
3. 第2部已经安装好hexo
4. 初始化hexo程序，运行一下命令
5. >hexo init
6. >npm install
7. >hexo generate
8. >hexo server -p 5555 既可以直接看到localhost:5555生成得博客
9. 博客默认是landscape主题，可以在themes/landscape看到，也可以在_config.yml中看到theme: landscape配置项得默认值。
10. 克隆next主题到themes/next, 根目录运行
11. >git clone --branch v5.1.2 https://github.com/iissnan/hexo-theme-next themes/next
12. 修改根目录配置文件_config.yml中theme: landscape为theme: next
13. >hexo generate
14. >hexo s -p 5555 可以观察到next已经生效
15. 建议删除themes/next中得.git文件夹，否则需要使用submodules git来处理比较麻烦，正常来说配置好后很长时间不会改变，即使有改变也可以通过beyondcompare来比对就可以了。

## hexo next 个性化
修改主要涉及根目录_config.yml, themes/next/_config.yml配置文件，还有themes/next/sourec/images文件夹，其他地方一般不需要修改。

### 自定义博客名字 _config.yml
```
title: Fitz.Lee
subtitle:
description: Security & Android & Java
keywords:
author: Fitz.Lee

url: https://fitzlee.github.io

deploy:
  type: git
  repo: https://github.com/fitzlee/FitzLee.github.io
  branch: master
```

### 自定义博客avatar和favicon
* 准备一个图标, 使用在线工具，转换成32*32的ico文件作为网站logo
* 替换images/favicon.ico这个文件
* 修改next/_config.yml中favicon: images/favicon.ico
* 替换images/avatar.gif这个文件

### 自定义菜单
* 修改next/_config.yml文件中
```
menu:
  home: /
  categories: /categories/
  #about: /about/
  archives: /archives/
  tags: /tags/
  about: /resume/
```
* 运行hexo new page "categories"
* 运行hexo new page "tags"
* 修改文件夹根目录source/categories/index.md, 添加type: "categories"
* 修改文件夹根目录source/tags/index.md, 添加type: "tags"
* about: /resume/ 说明about/直接运行到resume.git这个仓得gh-pages静态页面，源码可以直接参考resume.git的gh-pages分支。

### 自定义社交图标
```
social:
  #LinkLabel: Link
  GitHub: https://github.com/fitzlee
  E-Mail: mailto:fitz.lee@outlook.com
  JianShu: https://www.jianshu.com/u/c7757daadf27

social_icons:
  enable: true
  icons_only: false
  transition: false
  # Icon Mappings.
  # KeyMapsToSocialItemKey: NameOfTheIconFromFontAwesome
  GitHub: github
  E-Mail: envelope
  JianShu: book
```

### 自定义友情链接或有意思得link，如下
```
# Blog rolls
links_title: Links
#links_layout: block
links_layout: inline
links:
  Anquanke: https://www.anquanke.com/
  Gityuan: https://www.gityuan.com/
```

### 自定义固化文章链接
默认是使用md文章得title来生成链接，如类似2017/01/02/0x23-0x43等，甚至还包含中文字符，那么会导致标题变化后，地址也变化，不利于google/baidu的索引，所以固定链接很重要。
如下修改根目录_config.yml，如下添加urlname作为文章唯一链接。
```
permalink: posts/:urlname.html
permalink_defaults:
```
修改每一篇文章为如下样式
```
---
title: JVM和ART经典书籍推荐
urlname: jvm_art_book_recommand
date: 2016-06-05
comments: true
top: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 03JVM&ART
tags:
    - 书籍推荐
    - JVM
    - ART
---
```

### 运行查看样式hexo generate
### hexo s -p 5555查看运行结果。


## 置顶文章hexo-generator-index-pin-top

### 安装hexo-generator-index-pin-top
>npm uninstall hexo-generator-index
>npm install hexo-generator-index-pin-top --save
>给文章添加top: true的标签即可，但是多个top的话，没法自主选择优先级

## 安装search搜索页面
> npm install hexo-generator-searchdb --save
> 直接在themes/next/_config.yml修改
```
local_search:
  enable: true
```
如果安装后搜索的链接不一致，那么请修改permalink: posts/:urlname.htmlw为这个样子，不是permalink: /posts/:urlname.html这样。

运行hexo g/hexo s -p 5555即可观察到效果。

## 添加travis ci支持
https://www.jianshu.com/p/e22c13d85659

## 一些配置文章参考

### hexo 置顶文章
https://blog.csdn.net/qwerty200696/article/details/79010629


### hexo环境配置
https://blog.csdn.net/lijiajun95/article/details/53862528


## 个性化About页面简历
建议直接跳转到另外一个gh-pages
具体建立可以参考该页模版: https://gitee.com/cool-resume

## 显示更多摘要，不现实全文
最好对置顶得几篇文章，进行精确控制，其他不控制即可。
精确控制需要在md文章中，添加<!--more--> 即可。

## 其他 Quick Start

### 创建新文章

``` bash
$ hexo new "My New Post"
```

### 打开source\_posts\xxx.md,开始写文章

#### 主要用法
``` cpp
title: Hexo写文章
layout: post
date: 2014-03-03 19:07:43
comments: true
categories: Blog
#tags: [Hexo]
tags:
    - Hexo
    - Blog
keywords: Hexo, Blog
description: 生命在于折腾
photos:
    - https://ss0.bdstatic.com/5aV1bjqh_Q23odCf/static/superplus/img/logo_white_ee663702.png
```

#### 显示摘要
``` cpp
以上是文章摘要 
<!--more--> 
以下是余下全文 
```

#### MarkDown  [Markdown-Chinese-Demo](https://github.com/guoyunsky/Markdown-Chinese-Demo)
``` cpp
# 标题1
## 标题2
### 标题3
- 列表1

测试

- 列表2
- 列表3
水平线
—————————-
1. 列表1
2. 列表2
3. 列表3
5. 顺序错了不用担心
3. 写错的列表，会自动纠正
如果文字后面紧跟着水平线，看看是什么效果
———————
*我是斜体*
测试文字里面，**我是粗体**，很简单
__我是粗体__
————————————————————————————
保存后，就可以看到渲染后的效果：
```

More info: [Writing](http://hexo.io/docs/writing.html)

### 本地查看

``` bash
$ hexo server
```
More info: [Server](http://hexo.io/docs/server.html)

### 部署到git



``` bash
$ hexo d -g
```
More info: [Generating](http://hexo.io/docs/generating.html)
More info: [Deployment](http://hexo.io/docs/deployment.html)
More info: [Markdown](http://dillinger.io/)


### hexo travis
https://www.jianshu.com/p/e22c13d85659

一些命令
npm -v 
node -v
