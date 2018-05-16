---
title: 如何用 Hexo + GitHub 搭建博客
date: 2018-8-15 18:11:45
---

搭完博客的第一篇文章，讲讲搭建的过程。

本文旨在让初学者快速上手，用 hexo 快速搭建自己的博客。😀
<!--more-->

### what is [hexo](https://github.com/hexojs/hexo)?
- blog framework
- based on Node.js
- backend free
- simple & powerful

### 创建 hexo 项目

1. 安装 hexo
    ```
    $ npm install -g hexo-cli 
    ```
2. 初始化项目&&安装依赖
 
    ```
    $ hexo init bolg（项目名称）
    $ cd blog
    $ npm install
    ```
    
 3. 完成上述步骤之后，可以看到项目目录如下:
    
    ```text
    ├── _config.yml // 网站的配置文件，可以在其中配置网站的大部分参数
    ├── package.json // 应用程序的信息
    ├── scaffolds // 模板文件夹，当新建文章的时候，Hexo会根据模板来建立文件
    ├── source // 资源文件夹，用来存放用户资源的地方
    |   ├── _drafts
    |   └── _posts
    └── themes // 主题文件夹，Hexo会根据主题来生成不同的静态页面
    ```
4. 执行以下命令预览博客雏形：

    ```text
    $ hexo g #生成静态页面，生成的内容在public文件夹下
    $ hexo s #启动本地服务，进行文章预览调试。hexo s --debug 命令可以用来调试
    ```
    
### theme

- 套用主题模板

    hexo 拥有很多主题，大家可以去各大网站寻找自己喜欢的主题，如：[https://hexo.io/themes/](https://hexo.io/themes/)等等。
    在 theme 文件夹下
    
    ```text
    $ git clone 你想要主题地址
    
    ```
     然后将 _config.yml 的 theme 配置成该主题的文件夹名称，执行
     
    ```text
    $ hexo g
    $ hexo s
    ```
    查看新主题的展示效果。
    
- 自定义主题
    如你找不到满意的主题，可以自己写一个。那么如何自定义主题呢？
    主题文件夹只要包含：
    ```text
    - html template // html 模板（ejs/jade等）
    - config // _config.yml 配置文件
    - assets // 存放静态文件js、css等
    ```
    推荐工具[slush](https://github.com/tcrowe/slush-hexo-theme)，在此先不展开叙述，可自行参照工具文档。
    
### 如何部署
 首先先了解一下[GitHub page](https://help.github.com/articles/what-is-github-pages/)
 了解之后我们知道了，部署的实质，就是把 generate 到 public 文件夹下的内容同步到你的 username.github.io 项目之下。
 
 - 笨方法
    ```text
      - cd ./public
      - git init
      - git config user.name "summerbaby"  #修改name
      - git config user.email "xxxx@qq.com"  #修改email
      - git add .
      - git commit -m "update"
      - git remote add origin git@github.com:summerbabybiu/summerbabybiu.github.io.git
      - git push -f origin master
    ```
    缺点： 每一次修改都要手动执行一遍上面的命令。
    
- 一劳永逸
    利用 travis-ci 实现自动化部署。。
    首先以 GitHub 账号登录 [https://travis-ci.org](https://travis-ci.org)，找到你想要部署的项目，开启并前往设置,如下图：
    ![](/images/test.png)
    要想让 travis-ci 去部署，就要将您的 GitHub token 添加到项目的设置之中。
    先前往 GitHub 生成你的 token， 如下图：
    ![](/images/token.png)
    然后将 token 配置到项目设置中， 如下图：
    ![](/images/varibale.png)
    
    以上步骤完成之后，就可以去写脚本啦~~
    在项目的根目录下创建 `.travis.yml` 文件， 内容大致如下：
    ```text
    language: node_js  #设置语言
    
    node_js: stable  #设置相应的版本
    
    install:
      - npm install  #安装hexo及插件
    
    script:
      - hexo cl  #清除
      - hexo g  #生成
    
    after_script:
      - cd ./public
      - git init
      - git config user.name "summerbaby"  #修改name
      - git config user.email "xxxx@qq.com"  #修改email
      - git add .
      - git commit -m "update"
      - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master  #GH_TOKEN是在Travis中配置token的名称
    
    branches:
      only:
        - master 
    env:
     global:
       - GH_REF: github.com/summerbabybiu/summerbabybiu.github.io.git  #设置GH_REF，注意更改yourname

    ```
    部署脚本写完之后，提交代码，就可以去 https://travis-ci.org 查看部署进度啦~~~
    
    部署完之后，前往你的 https://username.github.io 查看你的博客。
    
### END
撒花❀❀❀ 然后你就完成自己的博客啦啦啦啦😋