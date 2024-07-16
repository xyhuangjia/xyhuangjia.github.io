---
title: Hexo相关零碎
date: 2018-08-04 15:48:19
tags:
---

**题外话**

我胡汉三又回来！！！

博客继续每月更新。。。

奉水贴一篇。

 <!--more--> 

## 命令

### 安装与更新

```javascript
npm install hexo -g #安装  
npm update hexo -g #更新
```

### 新建文章

 &emsp;&emsp; 当你胸中有大竹棍时候。你可以使用如下两条命令来操作新建一份可以直接在pc或者移动端预览的文稿。

```shell
hexo new "title" #新建文章
hexo n "title" #简化命令
```

###  新建草稿

推荐使用这种方式来新建文稿，然后处理下再发布出去。其实主要是个人水平太弱，找到课题也不能用目前知识解答出来，写出的东西一般漏洞百出。

```shell
hexo new draft "new draft"
```

 ### 预览

```shell
hexo server --drafts # 预览草稿
hexo s
hexo publish #发布
hexo p  
hexo generate#生成
hexo g 

```

### 部署

```shell
hexo deploy#部署
hexo d  
hexo generate --deploy
hexo deploy --generate
或
hexo deploy -g
hexo server -g
```

## 主题设置

&emsp;  其实个人觉得这个没什么好说的，都是一些很简单的东西。一般是下载主题和.yml文件参数配置。其实这个过程也不会出现什么影响流程的问题。最多git clone 慢，修改下hosts 文件就好，推荐个[工具](https://github.com/HostsTools/macOS)。

个人使用next模版

```yaml
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title:  #博客标题
subtitle: #博客子标题
description: #描述
author: #作者
language: zh-Hans #汉语
timezone: # 时区

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: #博客地址
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
post_asset_folder: true
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

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
theme: next
# scheme: Pisces
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: # 部署地址
  branch: master

```

##  常见问题








