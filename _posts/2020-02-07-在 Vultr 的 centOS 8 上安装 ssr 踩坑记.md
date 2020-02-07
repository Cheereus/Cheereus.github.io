---
layout:     post
title:      在 Vultr 的 centOS 8 上安装 ssr 踩坑记
subtitle:   ssr 服务器端安装总结
date:       2020-02-07
author:     Cheereus
header-img: img/post-bg-debug.png
catalog: true
tags:
    - Linux
    - SSR
---

在 <https://vultr.com> 买好新的服务器之后，先去 <https://www.vps234.com/ipchecker/> 测试一下 ip 有没有被墙
如果已经被墙，可以销毁后重新部署一台。

### 先安装 wget

> yum -y install wget

实际上在 Vultr 上的 centOS 8 中已经安装了 wget，上一步可以略过。

### 然后安装ssr一键脚本管理

> wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh && chmod +x ssr.sh && bash ssr.sh

然后进行参数配置，配置完成启动的时候可能会失败，这时参考下面的问题。
需要注意的几个问题是：

### Python 导致的启动失败

Vultr 上买的新服务器(centOS 8)虽然已经安装了 python36(若服务器没有安装 python 则需要先安装 python)，但没有配置为默认，因此 ssr 客户端会启动失败。

因此要先配置 Python36 为默认 Python：

> alternatives --set python /usr/bin/python3

至此 ssr 客户端就可以正常启动了。

### firewall 导致的无法连接

第二：服务器默认开启了 firewall 防火墙，且默认没有开放任何端口，因此要将所配置的 ssr 端口(以443为例)添加到白名单并重启防火墙

> firewall-cmd --add-port=443/tcp --permanent
> firewall-cmd --add-port=443/udp --permanent
> firewall-cmd --reload

然后就可以正常连接和使用 ssr 进行科学上网活动了。

### 附本人所使用的配置

> 加密：aes-256-cfb
> 协议：origin
> 混淆：http_post

设备数限制和线程数限制可自行斟酌，不需要可以直接按回车跳过。具体关于协议和混淆的配置请参考文末的参考文章。

要启动 ssr 一键脚本管理，可以运行

> bash ssr.sh

来管理和配置 ssr 服务端。

### 参考内容

本文参考的原详细教程 <https://github.com/Alvin9999/new-pac/wiki/%E8%87%AA%E5%BB%BAss%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%95%99%E7%A8%8B>

python 安装和配置默认 <https://ywnz.com/linuxjc/6033.html>

firewall 防火墙相关命令 <https://www.cnblogs.com/Raodi/p/11625487.html>
