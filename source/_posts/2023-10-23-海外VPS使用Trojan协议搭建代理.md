---
title: 海外VPS使用Trojan协议搭建代理
date: 2023-10-23 14:09:29
permalink: Trojan-Proxy/
tags:
- Trojan
- VPS
- Proxy
categories:
- Major
- Proxies
index_img: https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310231415554.jpg
excerpt: 通过Trojan协议搭建代理，实现快速安全访问
---

# 海外VPS使用Trojan协议搭建代理

## 使用自动脚本

安装前需要将任意域名解析到此VPS，如`proxy.sakiison.icu`，可以使用二级域名三级域名。

在root用户权限下运行下列脚本：

```bash
#安装/更新
source <(curl -sL https://git.io/trojan-install)

#卸载
source <(curl -sL https://git.io/trojan-install) --remove
```

![installing](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102051643.png)

使用`trojan`协议需要申请证书，个人常用的是第一种和第四种，新手建议使用第一种，有效期3个月可自动续签；第四种下文讲解。

![encrypt](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102051287.png)

安装数据库，建议用第一个就好。

![database](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102053695.png)

等他跑一会就装好了，之后命令行会出现管理面板。

![1](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102057007.png)

![2](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102058828.png)

![3](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102058973.png)

![5](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102058142.png)

可以通过浏览器访问`https://域名`进入管理面板进行多用户管理。

![panel](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102101095.png)

装完了，简单高效！~

## 证书有关问题

### 第一种,`Let's Encrypt`：

![let's encrypt](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102105991.png)

选择`1`，填入域名，会自动运行证书申请脚本。

### 第四种，自定义证书

> 此方案针对已经在域名注册商处申请证书的用户，导入证书使用。

部分域名注册商会提供免费证书额度，按照指示申请即可，以腾讯云为例。

![Tencent cloud](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102207858.png)

大部分网站用个免费的就够了。

![cert](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102208455.png)

根据提示设置就好，验证方式我一般用文件验证，用宝塔面板传个文件很容易，DNS有时候就很慢，很久不生效。自己选个适合的就行。

![验证](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102212187.png)

申请成功之后会有邮件短信提示，现在就可以下载证书了。

![download](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102213245.png)

下载格式随便选，注意要有`crt`和`key`文件。

![cert format](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102217418.png)

下载解压之后，上传。

![auth](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202311102157043.png)

将证书的`cert`和`key`上传至任意路径，粘贴路径于输入框即可。

---

**Enjoy！**

## Reference

[^1]:[VPS 初体验（三）在 VPS 上快速搭建 trojan 服务](https://kiku.vip/2021/10/16/在 VPS 快速搭建 trojan 服务/)
[^2]:[Jrohy/trojan: trojan多用户管理部署程序, 支持web页面管理](https://github.com/Jrohy/trojan)