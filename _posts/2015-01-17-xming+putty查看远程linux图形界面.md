---
layout: article
title: xming+putty查看远程linux图形界面
---

最近想查看一下linux服务器上运行的tomcat的情况，本来想借助vnc+jvisualvm的，但是服务器没有配置vnc server。在网上搜了一圈，发现使用xming+putty是不错的方法，因为ssh原本就是配置好的。

下面是一些配置步奏，具体原理可以在网络上查询。

### linux服务器端配置 ###

    #vi /etc/ssh/sshd_config

取消这一行的注释——如果没有这一行则手动添加之：

    X11Forwrding yes

这样配置的作用是允许SSH的X转发。

    #yum install xauth -y

有些服务器没有安装xauth包，导致打不开图形界面，所以也需要先执行这个命令。

### xming安装 ###

xming可以到sourceforge上面下载，下载路径：http://sourceforge.net/projects/xming/

安装基本是next就行，如有问题，请自行查询。

![install xming](/img/xming-install-config.jpg)

### putty配置使用 ###

putty下载路径：http://www.putty.org/，putty无需安装，解压即可使用。

打开putty：Connection → SSH → X11

勾选**Enable X11 forwarding**，

![putty X11 config](/img/putty-X11-config.png)

注：使用上面配置好putty连接远程linux服务器之前，请先打开Xming程序

接下来我们运行jvisualvm命令，即可在本地看到jvisualvm的图形界面。

![putty jvisualvm](/img/putty-jvisualvm.png)

![remote linux jvisualvm gui](/img/remote-linux-jvisualvm-gui.png)