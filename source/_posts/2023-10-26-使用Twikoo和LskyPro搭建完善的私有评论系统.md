---
title: 使用Twikoo和LskyPro搭建完善的私有评论系统
date: 2023-10-26 16:49:57
permalink: Comment-Twikoo-LskyPro/
tags:
- Docker
categories:
- Docker
index_img: https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310261652395.png
excerpt: 使用Twikoo和LskyPro搭建自有评论，安全放心
---

# 使用Twikoo和LskyPro搭建完善的私有评论系统

> 教程包含以下部分：
>
> - Twikoo搭建
> - Twikoo嵌入Hexo-Fluid
> - LskyPro-Twikoo配置

## 基于Docker搭建Twikoo

### 启动容器

```dockerfile
docker run -d \
	--name twikoo \
	--restart="always" \
	-e TWIKOO_THROTTLE=1000 \
	-p 34322:8080 \
	-v /home/konrad/twikoo/data/:/app/data \
	-d imaegoo/twikoo
```

{% note success %}

注意在**服务商控制台**安全组里放行相应端口，**不要**用宝塔放行，大部分不生效。

{% endnote %}

> #-p 外部映射端口任意选择，**不冲突**即可，内部暴露端口不建议改，根据镜像作者的说明来，有的镜像更改会出问题。
>
> 由于Docker的内部端口是独立的，所以内部暴露端口不需要考虑冲突问题，也就是冒号后面的端口可以在不同容器间重复，比如有很多容器的暴露端口都是`8080`。
>
> 外部映射端口，也就是冒号前面的端口是不能重复的，因为这是宿主机的端口，每个端口都是唯一的，不能复用。
>
> #-v 配置data所在目录，根据自己服务器环境更改
>
> #--name 根据需要更改

> 访问相应`ip:port`网址访问页面，如`123.123.122.111:34322`

![success](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310311411665.png)

### 绑定域名

根据`Twikoo`官方文档的阐述，我们必须要通过绑定域名的方式来进行`https`加密，笔者实测，使用`ip:port`的常规访问方式是不生效的，所以，这次的域名配置将是必选。

去域名注册商或DNS提供商处添加`A`型解析，解析到当前服务器。

![dns](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310311415873.png)

然后是熟悉的宝塔添加网站，证书，并配置反代的过程。

> 以下示意图引用自博主另一篇文章，端口不对应，注意更改

当然，添加网站和`SSL`证书申请是必要的前期工作。

![添加网站](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271934061.png)

![申请证书](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271934224.png)

开启反向代理，代理到本地相应端口。

![settings](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271934112.png)

![reproxy](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271934964.png)

域名绑定成功之后，我们就可以通过访问域名看到刚才同样的成功信息。

![success](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310311420815.png)

## Hexo-Fluid启用Twikoo

在`_config.fluid.yml`进行配置

![set](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310311425398.png)

下滑，填入`envID`，即所绑定的域名

![bond](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310311427568.png)

保存生成之后，就可以看到页面底部的评论系统。

点击齿轮可以进入设置

![Comment System](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310311428334.png)

## 接入LskyPro

LskyPro图床的搭建，请查看

{% post_link 基于LskyPro和阿里云对象存储搭建私有图床 %}

### token获取

```shell
curl -X POST -F "email=your_email@address" -F "password=your_passwd" https://your.domain/api/v1/tokens
```

将上述信息替换成你自己的，运行代码即可获得以数字开头的`token`，如`1|fafdafdadfasfadf`

### 配置

进入`Twikoo管理面板`->`配置管理`->`插件`

填入私有图床网址和`token`，保存即可。

![pichost](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310311436236.png)

---

到此为止，你的博客已经拥有了一套完整的评论系统。



↓↓↓你也可以在下面尝试一下↓↓↓↓

## Reference

[^1]:[云函数部署 | Twikoo 文档](https://twikoo.js.org/backend.html#私有部署-docker)
[^2]:[lsky-org/lsky-pro · Discussion #357 (github.com)](https://github.com/lsky-org/lsky-pro/discussions/357)
