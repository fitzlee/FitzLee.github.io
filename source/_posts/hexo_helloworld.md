title: Hexo 写文章方法

#title: 使用Hexo搭建个人博客
#layout: post
date: 2013-09-05 22:07:43
comments: true
categories: Blog
#tags: [Hexo]
keywords: Hexo, Blog
description: 生命在于折腾
photos:
    - https://ss0.bdstatic.com/5aV1bjqh_Q23odCf/static/superplus/img/logo_white_ee663702.png

tags:
    - Hexo
    - Blog


---
这里简要做介绍，深入了解可以参考:[官方文档](http://hexo.io/docs/)

## Quick Start

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