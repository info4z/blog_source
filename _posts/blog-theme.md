---
title: 博客美化
date: 2019-12-24 23:16:00
categories:
- 博客搭建
---



> hexo 提供了博客的骨架, 通过 kaze 使其丰满起来吧





## 一 : 主题

* 博客搭建完毕后, 可以对其进行一定的美化, 可以使用 kaze 主题

  ```sh
  $ git clone git@github.com:theme-kaze/hexo-theme-kaze.git themes/kaze
  ```

## 二 : 站点配置

* 编辑博客根目录下的 `_config.yml`

  ```yaml
  # Site
  title: 冰清阁		#标题
  subtitle: ''
  description: '差不多得了, 玩什么命呀...'	#简介或者格言
  keywords:
  author: 清月明风		#作者
  language: zh-CN		#主题语言,查看themes\next\languages下面的具体名字
  timezone: Asia/Shanghai		#中国的时区
  
  # URL
  url: https://info4z.github.io
  
  # 代码高亮
  highlight:
    enable: true
    line_number: false
    auto_detect: false
    tab_replace: ''
    wrap: true
    hljs: true
  prismjs:
    enable: false
  
  # Extensions
  theme: kaze
  ```

## 三 : 主题配置

* 修改 kaze下的 `_config.yml`

  ```yaml
  # Header config
  title: 冰清阁
  author: 清月明风
  logo_img: https://img.songhn.com/img/Y67gdd.png
  author_img: https://img.songhn.com/img/Y67gdd.png
  author_description: 差不多得了, 玩什么命呀...
  ```
  
* 目录

  ```yaml
  # Navbar config
  menus:
    home: /
    tags: /tags
    categories: /categories
    archive: /archives
    about: /about
    friends: /friends
  ```

* 当然了, 有些目录是不存在的, 需要手动创建

  ```sh
  $ hexo new page tags
  $ hexo new page categories
  $ hexo new page about
  $ hexo new page friends
  ```

* 搜索

  ```yaml
  search:
    enable: true
    path: search.json
    field: posts
    searchContent: true
  ```

