---
layout: post
title:  "RHEL 5.4 设置为字符界面启动"
date:   2014-02-03 17:32:00 +0800
categories: linux
---

前一段时间由于公司某项目的需要，安装了一个与生产环境相同的RHEL 5.4，安装以及运行的最大感受，就是RHEL 5看上去要比RHEL 6来得Hacker一些。嗯，RHEL 6已经已经够Hacker了，RHEL 5更加Hacker。

跟RHEL6.4不同，RHEL 5.4没有最小安装，默认安装时就带有X11，所以默认的启动方式就是进入图形界面，不过按下Alt+F1之类的切换到其他tty，可以进入字符界面。

但是我个人是命令爱好者，更愿意一启动就进入字符界面，以前在Fedora下这么干过（还把系统搞崩过= =），想必RHEL应该差不多。

首先，这些配置应该放在/etc/inittab这个配置文件下，打印一下这个文件，可以看到如下内容：

    # Default runlevel. The runlevels used by RHS are:
    #   0 - halt (Do NOT set initdefault to this)
    #   1 - Single user mode
    #   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
    #   3 - Full multiuser mode
    #   4 - unused
    #   5 - X11
    #   6 - reboot (Do NOT set initdefault to this)
    #
    id:5:initdefault:

"id:"和":initdefault:"之间的数字就说明初始化时使用哪个runlevel，总共有7个runlevel：

0：halt（停止，请勿将默认启动设置成这个）  
1：单用户模式  
2：不带NFS的多用户模式（没有连接网络时可以用这个）  
**3：多用户模式**  
4：未使用（也不要设置成这个）  
**5：X11，就是图形用户界面**  
6：reboot（重启，也不要设置成这个）  
从上面的信息看来，比较实用的就3和5两种了。

然后，可以用vi把"id:5:initdefault:"改为"id:3:initdefault:"，保存，退出，重启，然后就可以看到文字版的登陆界面了。

这时，再想进入X11图形用户界面，可以输入startx命令。

以上。
