---
title: 博客搭建
date: 2019-12-24 22:42:54
categories:
- 博客搭建
---



> 基于 hexo 在 github 上搭建属于自己的博客

## 一 : nodejs

* 官网 : https://nodejs.org/zh-cn/

* 下载地址 : https://nodejs.org/dist/v18.12.1/node-v18.12.1-x64.msi (长期维护版即可)

* 安装完成后 : 

  ```sh
  # 查看node版本
  $ node -v
  ```

## 二 : npm

* `node.js` 自带 `npm`

  ```sh
  # 查看npm版本
  $ npm -v
  ```

* `npm` 有时候不是特别好用, 可以使用 `cnpm`

  ```sh
  $ npm install -g cnpm --registry=https://registry.npm.taobao.org
  ```

* 安装过程中可能会出现无法加载文件的问题, 解决方案如下

  ```sh
  # 以管理员身份运行powerShell
  PS D:\Blog> set-ExecutionPolicy RemoteSigned
  
  执行策略更改
  执行策略可帮助你防止执行不信任的脚本。更改执行策略可能会产生安全风险，如 https:/go.microsoft.com/fwlink/?LinkID=135170
  中的 about_Execution_Policies 帮助主题所述。是否要更改执行策略?
  [Y] 是(Y)  [A] 全是(A)  [N] 否(N)  [L] 全否(L)  [S] 暂停(S)  [?] 帮助 (默认值为“N”): A
  PS D:\fore\jshERP-web> get-ExecutionPolicy
  RemoteSigned
  ```

* 查看版本信息

  ```sh
  $ cnpm -v
  ```

## 三 : hexo

* 使用 npm 安装 hexo

  ```sh
  # npm和cnpm哪个好使用哪个
  $ cnpm install -g hexo-cli
  ```

* 初始化

  ```sh
  # init : Create a new Hexo folder
  $ hexo init
  ```

* 生成工具栏

  ```sh
  $ hexo new page tags 		# 新增标签
  $ hexo new page categories	# 新增分类
  ```

* 新写文章

  ```shell
  # 这里
  $ hexo new "文章题目"
  ```

* 本地启动

  ```sh
  $ hexo server
  $ hexo s
  ```

* 生成静态文件

  ```sh
  $ hexo generate
  $ hexo g
  ```



## 四 : github

* 创建仓库, 这里只需要注意 `repository name` 的值为 : ` 用户名.github.io`

* 通过 `Settings` 查看 `Pages`

  ```sh
  Your site is live at https://info4z.github.io/		# 这就是个人博客的地址
  ```

* 编辑 `_config.yml`

  ```yaml
  deploy:
    type: git
    repository: git@github.com:info4z/info4z.github.io.git 	# 用ssh连接
    branch: main
  ```

* 安装 git 部署插件

  ```sh
  $ cnpm install hexo-deployer-git --save
  ```

* 执行如下指令

  ```sh
  # 清除缓存文件db.json和已生成的静态文件public
  $ hexo clean   
  # 生成网站静态文件到默认设置的public文件夹(generate)
  $ hexo g
  # 自动生成网站静态文件,并部署到设定的仓库(deploy)
  $ hexo d
  ```

