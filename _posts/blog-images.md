---
title: 博客图床
date: 2019-12-25 23:42:54
categories:
- 博客搭建
---



> 要写博客, 免不了要图文结合, 图片怎么办 ?



## 一 : typora

* 下载地址 : https://download2.typoraio.cn/windows/typora-setup-x64.exe
* 傻瓜式安装即可

## 二 : github

* 创建一个公开仓库 : blog_images
* 在 settings => Developer settings => Personal access tokens 里生成一个 token

## 三 : picgo

* 文件 => 偏好设置 => 图像, 做如下修改

  ![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230114155825153.png)

* 点击下载或更新, 下载 PicGo-Core(command-line)

* 下载完毕后, 点击打开配置文件, 或者手动打开 `C:/Users/{用户名}/.picgo/config.json`, 参考下面文件进行配置

  ```json
  {
      "picBed": {
  		"current": "github",
          "github": {
              "repo": "info4z/blog_images",
              "branch": "main",
              "token":"刚刚生成的 Personal access tokens",  
              "path": "images",
              "customUrl": "https://cdn.jsdelivr.net/gh/info4z/blog_images@main"
          }
      },
      "picgoPlugins": {}
  }
  ```

