---
layout:     post
title:      v2ray客户端结合Chrome的使用教程
subtitle:   windows上的安装使用说明
date:       2020-02-07
author:     Cheereus
header-img: img/post-bg-debug.png
catalog: true
tags:
    - v2ray
    - 原创
---

### 压缩包内容简介

![压缩包内容](/img/post/2020031301.jpg)

#### SwithyOmega_Chromium 目录

为 Chrome 扩展包，用于在 Chrome 中安装 SwitchyOmega 插件从而可以连接 v2ray 的系统代理。

#### v2ray.exe

为 v2ray 的显示命令行启动程序，双击后 v2ray 会以命令行窗口的形式运行，当关闭命令行窗口时， v2ray 也会随之关闭。

#### wv2ray.exe

为 v2ray 的隐藏式启动程序，双击后 v2ray 会自动进入后台运行，因此肉眼看不到任何启动效果，此种方式启动可以通过资源管理器来关闭进程。

### 安装及使用流程

#### v2ray 系统代理的启动

按以上所说，酌情选择 v2ray.exe 或者 wv2ray.exe 双击运行即可

#### 在 Chrome 中安装 SwitchyOmega 插件

安装插件本来可以通过 Chrome 的应用商店直接操作，但这需要本来就已经能够访问谷歌，因此无需再赘述。

压缩包中准备了直接安装的文件，具体操作如下：

##### 点击 Chrome 右上角的三点按钮，选择 *更多工具* 然后点击 *扩展程序*：

![插件安装1](/img/post/2020031302.jpg)

##### 在打开的页面中将右上角的开发者模式切换为开启状态

![插件安装2](/img/post/2020031303.jpg)

##### 点击 加载已解压的扩展程序

![插件安装3](/img/post/2020031304.jpg)

##### 选择压缩包中的 SwithyOmega_Chromium 目录，然后点击选择文件夹

![插件安装4](/img/post/2020031305.jpg)

##### 这时会自动弹出 SwitchOmega 的设置窗口

![插件安装5](/img/post/2020031306.jpg)

##### 如果没有自动弹出，可以右键点击浏览器右上角的圆圈形状的图标，点击 管理扩展程序 进入

![插件安装6](/img/post/2020031307.jpg)

#### SwitchyOmega 配置

##### 点击左侧 proxy 按下图填写好配置之后再点击左侧 应用选项

![插件配置1](/img/post/2020031308.jpg)

##### 此时，在浏览器右上角点击圆圈形状的 SwitchyOmega 图标，选择 proxy 即可翻墙

![插件配置2](/img/post/2020031309.jpg)

##### 上述步骤中选择 系统代理 即可取消翻墙，但我们希望浏览器能根据页面网址来判断哪些需要翻墙哪些不需要，自动替我们决定是否翻墙，因此有了下一步操作

##### 点击左侧 auto switch 再点击 添加在线规则列表

![插件配置3](/img/post/2020031310.jpg)

##### 选择 auto swith 然后复制 <https://gitlab.com/gfwlist/gfwlist/raw/master/gfwlist.txt> 到规则列表网址，然后点击立即更新，点击应用选项

![插件配置4](/img/post/2020031311.jpg)

![插件配置5](/img/post/2020031312.jpg)

##### 由于规则列表网址的访问不太稳定，不一定能马上更新到，可以先用上面 proxy 的方法，以后再用 auto switch

##### 将多余的规则删除

![插件配置6](/img/post/2020031313.jpg)

##### 将规则列表规则的情景模式改为 proxy 后点击左侧应用选项

![插件配置7](/img/post/2020031314.jpg)

##### 至此配置完毕 在浏览器右上角点击圆圈形状的 SwitchyOmega 图标，选择 auto switch 即可自动智能翻墙
