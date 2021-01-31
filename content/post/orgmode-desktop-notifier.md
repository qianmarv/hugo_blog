+++
title = "Setup Desktop Notifier For Org-Mode"
date = 2018-01-10
tags = ["emacs"]
categories = ["效率"]
draft = false
+++

介绍如何为Emacs中特别是Orgmode中的定时任务建立桌面提醒。
<!--more-->


## 背景介绍 {#背景介绍}

`orgmode` 中我们可以为某个TODO设置具体的计划开始时间，然后通过Emacs的appt包设
置提醒，比如提前20分钟提醒，然后可以隔5分钟提醒一次等等，但是问题在于这个提醒
是在Emacs中生成一个Buffer，如果这个时候你并不是在Emacs中做事情，那么很可能你就
看不到这个提醒了，所以想法是可以把这个提醒和外部的提醒程序结合起来，这样只要你
是在电脑前工作，你就一定可以收到提醒。

对于不同的操作系统平台，提醒的程序也是不一样的，目前暂时提供对WIN10的支持，调
用WIN10的Action Center来发送提醒。


## 基础设置 {#基础设置}


### 启用APPT并将ORGMODE中SCHEDULE到时分秒的TODO加入 {#启用appt并将orgmode中schedule到时分秒的todo加入}

首先你的TODO当然得设定在时分秒的粒度上，下面是一个例子：
我们创建一个TODO项，写本文的时间是2018-01-10，然后计划在17:00开始做这件事情。
那么我希望有个桌面提醒可以在这个任务开始之前5分钟提醒我。

```org
     * TODO Test Desktop Reminder
       SCHEDULED: <2018-01-10 Wed 17:00>
```

TODO有了，下面需要加入到APPT中去，执行命令：
`M-x org-agenda-to-appt`

我的工作流是每天早上会安排当天的TODO，其中会议等TODO会在schedule的时候设定固
定时间，然后执行这个命令加入到appt中。

到目前为止，appt可以用Emacs的内置提醒在制定时间弹出警告了。
你可以调整一下默认的设置如下：

```emacs-lisp
     (setq appt-message-warning-time 5) ; Show notification 5 minutes before event
     (setq appt-display-interval appt-message-warning-time) ; Disable multiple reminders
     (setq appt-display-mode-line nil)
```


### 增加桌面提醒功能 {#增加桌面提醒功能}

首先我们需要一个客户端工具可以设置提醒，最好是通过windows的action center来发
送提醒，这里有不少的option，我经过尝试，选择使用一个叫做 **BurntToast** 的
`Powershell` 下的脚本工具，具体安装可以参见：
<https://github.com/Windos/BurntToast>

安装完成之后，你可以用windows的cmd来调用下面命令测试：
`powershell New-BurntToastNotification -Text 'Test' -Sound 'Alarm2' -SnoozeAndDismiss`

现在我们需要写一个elisp function来调用这个程序:

```emacs-lisp
     (defun qianmarv-org/show-alarm (min-to-app new-time message)
       (cond ((string-equal system-type "windows-nt")
              (call-process "powershell"
                            nil
                            t
                            nil
                            (format " New-BurntToastNotification -Text '%s' -Sound 'Alarm2' -SnoozeAndDismiss" message)
                            ))))
```

接下来我们要告诉 `appt` 要发送桌面提醒，设置如下：

```emacs-lisp
     (setq appt-disp-window-function 'qianmarv-org/show-alarm)
```


## 在org-pomodoro中集成桌面提醒功能 {#在org-pomodoro中集成桌面提醒功能}

如果你是一个番茄工作法的爱好者，你使用 **org-pomodoro** 来工作，应该会碰到之前同
样的困扰，就是一个番茄时间满了之后，如果这是你不是在电脑前面的，你就不会看到
Emacs的提醒信息，导致接下来的工作并没有计入到番茄里面去，同样休息时间也不能够
好好的利用，而如果可以有桌面提醒就不会有这个问题了。你可以通过hook的方式将
org-pomodoro的提醒改为桌面提醒。

设置如下：

```emacs-lisp
    ;; Refer to https://gist.github.com/jstewart/7664823
    (add-hook 'org-pomodoro-finished-hook
              (lambda ()
                (qianmarv-org/show-alarm 0 0 "Pomodoro completed! - Time for a break.")))

    (add-hook 'org-pomodoro-break-finished-hook
              (lambda ()
                (qianmarv-org/show-alarm 0 0 "Pomodoro Short Break Finished - Ready for Another?")))

    (add-hook 'org-pomodoro-long-break-finished-hook
              (lambda ()
                (qianmarv-org/show-alarm 0 0 "Pomodoro Long Break Finished - Ready for Another?")))

    (add-hook 'org-pomodoro-killed-hook
              (lambda ()
                (qianmarv-org/show-alarm 0 0 "Pomodoro Killed - One does not simply kill a pomodoro!")))
```
