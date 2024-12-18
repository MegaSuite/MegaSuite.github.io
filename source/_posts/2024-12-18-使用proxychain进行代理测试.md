---
title: 使用proxychain进行代理测试
math: false
permalink: proxychain/
date: 2024-12-18 15:19:35
tags:
- Proxy
- Server
categories:
- Major
- Proxies
index_img: https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202412181534845.png
excerpt: 使用proxychain进行简单的代理测试
---

# 使用ProxyChain进行代理测试

## 安装

> 系统：Debian GNU/Linux 12 (bookworm)
>
> 以下操作均基于`root`用户权限，如有需要请自行添加`sudo`

直接从软件包管理器安装：

```bash
apt install -y proxychains4
```

进行配置修改：

```bash
vim /etc/proxychains.conf
```

在配置文件的说明中，可以看到proxychains支持以下几种代理，但只支持填写数字ip地址

![proxies](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202412181543351.png)

在其中的`[ProxyList]`中根据格式填写代理信息：

![proxylist](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202412181546631.png)

保存并退出。

## 使用

在需要使用代理的命令前添加`proxychains4`即可。

下图是没有使用代理的测试结果：

![without](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202412181549345.png)

下图是使用代理后的检测结果（笔者对美国服务器使用了荷兰地区的Socks5代理）：

![with](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202412181551929.png)

## [可选]设置别名

`proxychains4`这个名字还是太长，可以设置`alias`来方便使用

```bash
alias pc=proxychains4
```

重载终端

```bash
source ~/.bashrc #请根据自己的系统做出相应更改
```

现在就可以用别名来使用了

![alias](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202412181556803.png)

## Reference

[^1]:[rofl0r/proxychains-ng](https://github.com/rofl0r/proxychains-ng)
[^2]:[linux下的全局代理工具proxychain](https://monkeywie.cn/2020/07/06/linux-global-proxy-tool-proxychain/)
