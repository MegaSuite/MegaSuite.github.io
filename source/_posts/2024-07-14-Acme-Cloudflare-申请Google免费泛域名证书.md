---
title: Acme + Cloudflare, 申请Google免费泛域名证书
math: false
permalink: google-free-cert/
date: 2024-07-14 15:24:48
tags:
- VPS
- Website
categories:
- Major
- Website
index_img: https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407141531852.png
excerpt: 使用acme.sh自动服务和Cloudflare dns验证完成Google泛域名证书的申请
---

# Acme + Cloudflare, 申请Google免费泛域名证书

## 获取Google验证信息

进入链接，点击左上角项目名称，获得`project id`

```
https://console.cloud.google.com/apis/dashboard
```

<img src="https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407141534696.png" alt="image-20240714153430529" style="zoom: 67%;" />

将得到的`project id`填入下方链接并访问，启用`Public Certificate Authority API`服务

```
https://console.cloud.google.com/apis/library/publicca.googleapis.com?project=<Project ID>
```

<img src="https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407141538598.png" alt="image-20240714153819514" style="zoom:80%;" />

点击下图图标，进入`Cloud Shell`

![shell](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407141540879.png)

在终端中输入以下代码

```bash
gcloud config set project <Project ID> # 注意替换project id
gcloud beta publicca external-account-keys create
```

![key](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407141545043.png)

获取生成的`b64MacKey`和`keyId`

## 配置acme

使用下方命令安装acme，**注意替换email**

```bash
curl https://get.acme.sh | sh -s email=<EMAIL> 
source ~/.bashrc
```

设置默认注册服务器为Google

```bash
acme.sh --set-default-ca --server google
```

添加账户，替换为申请到的key和id，若显示失效需要重新申请

```bash
acme.sh --register-account  --server google \
        --eab-kid <keyId>  \
        --eab-hmac-key <b64MacKey>
```

## 配置dns验证

打开下方网址

```
https://dash.cloudflare.com/profile/api-tokens
```

API Tokens -> Create Token -> Edit Zone DNS(点击 use template)

> 在此页面也可获取Global Token，更省事，但不安全，不建议

![token](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407141556594.png)

如果需要为多个域名申请证书，可以点击`Zone Resources`下的`Add more`，添加更多域名。

完成后点击`Continue to summary`

![zone dns](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407141558885.png)

获取`API token`，此项称为`CF_Token`

![API token](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407141600664.png)

进入需要申请证书的域名的管理页面，下滑，右下角获取`CF_Zone_ID`和`CF_Account_ID`

![zone and account](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407141603723.png)

进入终端，设置环境变量

```bash
# 如果获取单域token,使用此形式，CF_Zone_ID需要根据申请证书的域名所改变。
export CF_Token="xxxxxxxxxxxxxxxxxxxxxxxx"
export CF_Account_ID="xxxxxxxxxxxxxxxxxxxxxxxx"
export CF_Zone_ID="xxxxxxxxxxxxxxxxxxxxxxxx"

# 如果获取的是全局token，使用此形式
export CF_Key='xxxxxxxxxxxxxxxxxxxxxxxx'
export CF_Email="你的cloudflare邮箱"
```

## 申请证书

```bash
acme.sh --issue --dns dns_cf -d example.com #单域名
acme.sh --issue --dns dns_cf -d *.example.com #泛域名
acme.sh --issue --dns dns_cf -d sub1.example.com -d sub2.example.com #多域名
acme.sh --issue --dns dns_cf -d example.com --ecc #ECC 证书（目前默认就是ecc证书）
```

看到回显`Cert success`即为申请成功，可以在`~/.acme.sh/_.example.com/`下获取证书

证书有效期三个月，acme会自动检测并续期

## More

上文中阐述的环境变量设置方法只在当前终端中有效，如果需要永久环境变量，请使用以下方法：

```bash
# 以下命令均基于Debian系统，其他系统请自行更改bash文件的位置
# 修改编辑权限
chmod -v u+w ~/.bashrc

vim ~/.bashrc

# 将需要的环境变量添加在文件末尾
export CF_Token = "xxxxx"
...
...
...

# 刷新终端
source ~/.bashrc
```

永久有效，且对所有用户生效。

## Reference

[^1]:[acmesh-official/acme.sh Wik](https://github.com/acmesh-official/acme.sh/wiki)
[^2]:[Automate Public Certificate Lifecycle Management via ACME Client API](https://cloud.google.com/blog/products/identity-security/automate-public-certificate-lifecycle-management-via--acme-client-api)
[^3]:[使用 ACME.SH 申请 Google CA SSL 证书](https://www.cestlavie.moe/posts/acme-gts-ssl/#fnref:1)
[^4]:[使用 ACME 申请 Google CA SSL 证书](https://blog.iyume.top/other/155.html)
[^5]:[使用 acme.sh 配置自动续签 SSL 证书](https://u.sb/acme-sh-ssl/)
