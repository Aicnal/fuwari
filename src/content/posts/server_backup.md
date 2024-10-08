---
title: 服务器备份方案
published: 2024-10-08
description: '基于Duplicati+Alist+阿里云盘的备份方案'
image: ''
tags: [Tech, Server]
category: 'Tech'
draft: false 
lang: ''
---
# 服务器备份方案

## 前言

在我们实际的生产环境中，由于云服务存在很多的不稳定性，因此对服务器进行定时备份就很有必要了。

虽然部分服务器厂商提供了快照备份的功能，但是有些是要去收费的，对于我们这些个人用户并不是十分友好，在参考了众多论坛大佬的方案后，我总结出了一套基于**Duplicati+Alist+阿里云盘**的备份方案

## 安装

### Alist

Alist官方提供了多种安装方式，在这里我直接使用[一键脚本](https://alist.nn.ci/zh/guide/install/script.html)进行安装

```bash
https://alist.nn.ci/zh/guide/install/script.html
```

安装完成之后进入到Alist到web界面，我们开始挂载阿里云盘

阿里云盘官方对Alist对支持非常好，具体挂载方式可以参考：

[阿里云盘 Open](https://alist.nn.ci/zh/guide/drivers/aliyundrive_open.html)

值得注意的是，如果你使用的也是北京阿里云的ecs，你可以在Alist的编辑界面中开启**内部上传**，这样的话走的就是阿里云的内网流量，带宽会快一点

![](https://s2.loli.net/2024/10/08/n7SNs8I5UQ1grZo.png)

之后我们需要确定webdev的挂载路径：

```bash
mkdir /mnt/webdev
```

之后对于具体的挂载方式，请参考：

[Ubuntu系统挂载Alist网盘 WebDav服务完全指南](https://github.com/alist-org/alist/discussions/5470)

### Duplicati

Duplicati的安装支持使用Docker，在这里我们直接使用docker-compose进行部署

```docker-compose
version: '3.8'

services:
  backup:
    image: lscr.io/linuxserver/duplicati:latest
    environment:
      - PUID=0
      - PGID=0
      - TZ=Asia/Shanghai
    ports:
      - 8200:8200
    volumes:
      - /root:/source/root  # 挂载整个 /root 目录
      - /etc:/source/etc    # 挂载整个 /etc 目录
      - ./duplicati/config:/config  # 挂载配置文件目录
      - ./duplicati/backups:/backups  # 挂载备份文件目录
      - /mnt/webdav:/backups_webdav
    restart: unless-stopped
```

在`/source/`这里你可以更改你自己所需要备份的路径

之后启动：

```docker-compose
docker compose up -d
```

等待安装完成，进入web界面：`http://IP:8200`

## 备份设置

进入web之后我们需要做的第一件事情就是设定密码！！！

首先进入到设置，然后输入密码，点左边的小方块，最后划到最下面进行确认，之后会要求你进行重新登陆，输入密码

![](https://s2.loli.net/2024/10/08/ZuoDTqnhSkxswia.png)

之后我们就可以正式开始备份了：

![](https://s2.loli.net/2024/10/08/AB31mZ68c4oURsq.png)

选择一个名字，这里我推荐你选择一个密码，请牢记

![](https://s2.loli.net/2024/10/08/gKdSj4plLnTVfbv.png)

这里的路径选择之前我们在docker-compose中挂载的路径：`backups_webdev`

![](https://s2.loli.net/2024/10/08/xrc32IpE9fUPKZi.png)

之后点击“下一步”，我们选择“计算机”，之后选择“source”

![](https://s2.loli.net/2024/10/08/cvbRleIWtsSKfMi.png)

![](https://s2.loli.net/2024/10/08/LXAlsViqp6JvySN.png)

选择“下一步”，之后选择同步周期：

![](https://s2.loli.net/2024/10/08/439G27JTZmwuLjs.png)

之后再点击“下一步”，在这里我建议你使用“智能备份保留策略”，这样的话比较古早的备份就会自动被删除，减少云盘的占用

![](https://s2.loli.net/2024/10/08/UZWhKXbLPVMIgaT.png)

点击“保持”即可结束配置，回到主页，点击“立即允许”，之后即可查看是否备份成功

![](https://s2.loli.net/2024/10/08/DEQAiTb6H7wgq1n.png)

备份成功之后在Alist中的对应文件夹中也可以找到文件的身影

![](https://s2.loli.net/2024/10/08/EH2gwOnbFljTaWp.png)

## 总结

就这样，一个增量式服务器备份就水灵灵得的部署好了

当然你也可以使用其他的存储云盘，比如说onedrive，google drive等等（当然前提是你的网络环境允许你这样做