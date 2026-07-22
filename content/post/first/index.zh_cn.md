---
title: "服务器网络问题解决方案"
description: 
date: 2023-11-10T15:34:23+08:00
image: linux_network.jpg
math: 
license: 
hidden: false
comments: true
slug: linux-network
categories: 
    - linux
    - network
tags:
    - server
    - proxy
    - git


---

# 对服务器网络问题的总结

最近使用实验室linux服务器跑代码，对其中频繁出现的网络问题烦不胜烦，诸如git和pip问题层出不穷，其中的问题不是简单换一个清华源可以，根源性需要让服务器走**中转代理**，实现完全性**翻墙**（该方法仅对内网有用）。

现在把其中常见的命令总结如下，方便以后快捷使用。

## git的代理设置
先对**clash**进行设置

![clash设置](clash.png)

```bash

git config --global https.proxy 127.0.0.1:7890 #端口号参考clash
git config --global http.proxy 127.0.0.1:7890  #端口前面的ip地址参考开clash的主机ip
```

如果报错提示不让写入修改，在最后加入`--replace-all`，代码如下

```bash
git config --global https.proxy 127.0.0.1:7890 --replace-all
git config --global http.proxy 127.0.0.1:7890 --replace-all
```

## 服务器全局设置

考虑到有些工具无法设置代理，所以使用全局代理，但这种方式在针对ICMP协议时会失效，因为使用的代理走的都是会话层，无法影响到ip报文解包的结果。

代码如下

```bash
export proxy="http://127.0.0.1:7890" ##ip和端口地址按情况替换
export https_proxy=$proxy
export http_proxy=$proxy
```

## 效果
![](torch.jpg)

可以让torch的安装速度从几百k涨到最高10mb/s。















*备注*

如果无法正常部署博客到github.io，请检查标题里是不是把草稿draft设置为了true，之后再运行如下命令

```bash
hugo 
hugo --gc --minify --cleanDestinationDir
```

