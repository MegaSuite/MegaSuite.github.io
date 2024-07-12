---
title: 使用YOURLS Docker版，搭建短链接服务
math: false
permalink: yourls-docker/
date: 2024-07-12 14:57:06
tags:
- Docker
categories:
- Docker
index_img: https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407121459709.png
excerpt: 使用YOURLS Docker版，搭建短链接服务
---

# 使用YOURLS Docker版，搭建短链接服务

## 最终目录结构

<img src="https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407121518422.png" alt="image-20240712151858363" style="zoom:80%;" />

以上是本项目完成时`/root/yourls`目录的结构，由于本项目部分组件的教程文档过于简陋，所以很容易放错位置，请务必注意目录的结构。

## 基础环境安装

首先，创建工作目录，比如`/root/yourls`，在其中新建`docker-compose.yml`，加入以下内容：

```yaml
version: '3.1'

services:

  yourls:
    image: yourls
    restart: always
    ports:
      - 34324:80
    environment:
      YOURLS_DB_NAME: yourls
      YOURLS_DB_USER: yourls
      YOURLS_DB_PASS: yourls
      YOURLS_SITE: https://url.xxxx.xxx
      YOURLS_USER: admin
      YOURLS_PASS: admin
    volumes:
      - ./yourls_data/:/var/www/html

  mysql:
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: yourls
      MYSQL_USER: yourls
      MYSQL_PASSWORD: yourls
    volumes:
      - ./mysql/db/:/var/lib/mysql
      - ./mysql/conf/:/etc/mysql/conf.d
```

> - “34324”端口为外部暴露端口，根据需要修改
> - YOURLS_DB_NAME -> MYSQL_DATABASE，YOURLS_DB_USER -> MYSQL_USER，YOURLS_DB_PASS -> MYSQL_PASSWORD，三组变量一一对应，根据自己需要填写
> - YOURLS_SITE为网站地址，填写自己即将使用的域名
> - YOURLS_USER/PASS是管理员登录账号密码，根据需要填写
> - 为便于修改内部文件，我们使用挂载卷的形式，`yourls.volumes`挂载在本地的`/root/yourls/yourls_data`，`mysql.volumes`挂载在本地的`/root/yourls/mysql`中的两个文件夹

运行

```bash
docker compose up -d
```

### 反向代理

若要使用域名访问，请设置反向代理，具体方式可以查看之前文章，不再赘述。

完成反向代理配置之后，打开`url.xxxx.xxx/admin`点击`install`进行安装，输入`YOURLS_USER/PASS`中设置的管理员账号密码，现在你就可以看到一个简陋的后台页面了。

**但是请注意，默认的`yourls`是只有后台管理员界面的，当你尝试访问`url.xxxx.xxx`只能得到大大的`Forbidden`**

如果你没有更多需求，那么通过这个`url.xxxx.xxx/admin`页面的管理员后台也是勉强可以使用的，但是这并不是我们印象中的所谓“短链接"平台，所以我们要安装前台页面，顺便给管理员后台美化一下。

## Sleeky美化包的安装

访问下方仓库下载主题包并查看简单教程

{% note primary %}

[Flynntes/Sleeky: 🎨 A sleek and simple frontend & backend theme for YOURLS](https://github.com/Flynntes/Sleeky)

{% endnote %}

### backend

下载之后解压，将其中的`sleeky-backend`移动至`yourls/yourls_data/user/plugins`，刷新后在管理员后台`manage plugins`选项中激活`sleeky backend`插件（点击`action`列的`activate`），顺道提一句，自带的`random shorturls`插件也建议启用，否则后缀只是按照数字排序，没内味😉

> 插件启用后记得刷新，否则浏览器缓存会导致看不到生效后的效果

<img src="https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407121538705.png" alt="240712153704" style="zoom:80%;" />

### frontend

打开下载的主题包中的`sleeky-frontend`文件夹，将其中的`frontend`文件夹和`index.php`文件复制到`/root/yourls/yourls_data`目录中，**注意看文章开头给出的目录结构**，这一步开发者文档中的说明有些模糊，容易造成误解。

重启程序

```bash
docker compose restart
```

打开`url.xxxx.xxx`就可以查看前台页面

<img src="https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407121547596.png" alt="image-20240712154707361" style="zoom: 33%;" />

### 其他美化和插件

可以进入以下仓库查看更多皮肤与插件

{% note success %}

[YOURLS/awesome: 🎉 A curated list of awesome things related to YOURLS](https://github.com/YOURLS/awesome)

{% endnote %}

## FAQ

### 页面显示不正常[^1]

在上述方法安装前台页面之后，可能会出现显示页面显示不正常的情况，对`/root/yourls/yourls_data/frontend/header.php`进行修改，删除下图第15行中的`<?php echo $YOURLS_SITE ?>` 后重启程序即可。

![240711202807](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202407121548066.png)

## References

[^1]:[Frontend page isn't displayed correctly · Issue #130 · Flynntes/Sleeky](https://github.com/Flynntes/Sleeky/issues/130#issuecomment-1824605122)
[^2]:[YOURLS/awesome: 🎉 A curated list of awesome things related to YOURLS](https://github.com/YOURLS/awesome)
[^3]:[YOURLS/YOURLS: 🔗 The de facto standard self hosted URL shortener in PHP](https://github.com/YOURLS/YOURLS)
[^4]:[【好玩儿的Docker项目】10分钟搭建一个自己的短链接服务](https://blog.laoda.de/archives/docker-compose-install-yourls)
[^5]:[yourls - Official Image | Docker Hub](https://hub.docker.com/_/yourls?tab=description)
