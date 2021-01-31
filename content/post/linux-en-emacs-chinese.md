+++
title = "Linux英文环境下在Emacs中使用中文输入法"
date = 2021-01-31
tags = ["emacs"]
categories = ["日志"]
draft = false
+++

记录在linux英文环境下设置在Emacs中通过fctix来切换中文输入法。
<!--more-->

相关索引

-   <https://github.com/hick/emacs-chinese>
-   <https://github.com/tumashu/pyim>

系统环境是Debian10，emacs27.1，语言环境为英文。

通过fcitx来切换中文和英文输入，设置快捷键为WIN+SPACE, 其他应用均无问题可以正常切换中文。

但是在emacs中，win+space会报错说为定义按键，无法切换中文，而内置的拼音输入法相对而言不那么好用。

经过一番探索和尝试，终于解决。最主要的线索在于emacs需要 `LC_CTYPE=zh_CN.UTF-8` 才能够正常调用fcitx，参照一些帖子内容将这个变量设置到本地的profile不能生效(`/.profile`)。可能放到 `/etc/profile` 中可以生效吧，没有尝试。

最终我采用的方式按照如下[帖子](https://codertw.com/%e8%bb%9f%e9%ab%94%e9%96%8b%e7%99%bc%e5%b7%a5%e5%85%b7/23416/)中介绍的方式通过在加载emacs之前手动设置 `LC_CTYPE`
基本步骤为：
在 `/usr/local/bin/` 中创建文件为 `emacs` ，由于我是自己编译的emacs27.1, 在 `sudo make install` 之后，在该目录下会创建 `emacs27-1` 的binary以及 `emacs` 的链接文件，我将链接文件 emacs删除，并创建一个命令文件并添加如下内容：

```sh
#! /bin/bash
export LC_CTYPE=zh_CN.UTF-8;
/usr/bin/local/emacs27-1 "$@"
```

完成之后通过 `sudo chmod +x emacs` 来设置执行权限即可。
