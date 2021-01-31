+++
title = "通过 Hexo + Git Page 搭建静态博客"
date = 2015-12-30
tags = ["blog"]
categories = ["折腾"]
draft = false
+++

介绍如何通过 Hexo + Git Page 搭建静态博客。
<!--more-->


## 环境 {#环境}

本地环境：WIN8
Cygwin64,
Git: 2.5.3
Hexo: Latest
服务器环境：Github Page


## 过程 {#过程}

1.  安装 nodejs
    <https://nodejs.org/en/>
    貌似被墙了，真是悲剧。
2.  安装 hexo
    <https://hexo.io/>

    ```sh
            npm install hexo-cli -g
            hexo init blog
            cd blog
            npm install
            hexo server
    ```

    到目前位置，已经可以运行了,浏览器打开：<http://localhost:4000>
3.  设置主题
    <https://hexo.io/themes/>
    我选的是这个主题：[maupassant](https://github.com/tufu9441/maupassant-hexo)， 可以在[这里](https://www.haomwei.com/) 预览。
    如何配置，在主题的 github 页面已经说的很清楚了。
    配置完毕，可以通过 hexo server 来预览效果。
4.  注册 github page 的 repository
    官方教程：<https://pages.github.com/>
    这个应该非常简单，只要注意一点注册的代码仓库（repository）必须是<githubaccount>.github.io，
    比如我的账户是 qianmarv，那么我注册的名字一定要叫 **qianmarv.github.io** ，其他就没问题了
5.  如何部署
    [官方说明](https://hexo.io/docs/deployment.html) ，简单的说一下的话就是两步：
    1.  安装插件[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)
    2.  修改设定\_config.yml

        > deploy:
        > type: git
        > repo: <repository url>
        > branch: [branch]
        > message: [message]

        可能存在的问题，如果是用 ssh 的方式应该不会有太大问题，如果是用 https 的方式，在我这边就发现了一个问题：
        hexo deploy 提交部署提示输入用户名，回车后就卡在那里了，不知道别人会不会也有同样的问题，总之就是密码提示出不来。

        最后通过 git 的 store 方式明文存放用户名和密码来解决，当然这个文件是通过文件权限来控制安全的，如果电脑只是本人使用应该还是安全的。
        具体来说两步：

        1.  执行：git config credential.helper 'store [options]'
        2.  在命令行手动 push 一次，命令行提示用户名和密码没有问题，输入后，用户名和密码就被记录下来了，下次就不需要在输了

        参考：[Git - How to avoid typing your password repeatedly](https://blog.sleeplessbeastie.eu/2012/08/12/git-how-to-avoid-typing-your-password-repeatedly/)
        [Git credential-store](https://git-scm.com/docs/git-credential-store)


## 剩下的问题 {#剩下的问题}


### <span class="org-todo todo TODO">TODO</span> 博客从 ORG 到 HEXO 的流程 {#博客从-org-到-hexo-的流程}


### <span class="org-todo todo TODO">TODO</span> 博客的迁移 {#博客的迁移}


### <span class="org-todo todo TODO">TODO</span> 主题的定制化 {#主题的定制化}


### <span class="org-todo todo TODO">TODO</span> 域名设置 {#域名设置}

<http://wiki.jikexueyuan.com/project/github-pages-basics/cname-file.html>
