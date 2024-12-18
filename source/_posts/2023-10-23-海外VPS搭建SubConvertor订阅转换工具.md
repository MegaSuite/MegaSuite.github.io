---
title: 海外VPS搭建SubConvertor订阅转换工具
date: 2023-10-23 14:10:40
permalink: SubConvertor/
tags:
- Server
- Proxy
categories:
- Major
- Proxies
index_img: https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310261058310.png
excerpt: 基于Docker搭建订阅转换服务，clash也能实现分流规则自定义
---

# 海外VPS搭建SubConvertor订阅转换工具

> 示例链接：[SubConverter](https://convertor.wisedrifter.top)
>
> 博主自用规则文件：[Clash-Configs/Policy/ClashGeneral.ini](https://github.com/MegaSuite/Clash-Configs/blob/main/Policy/ClashGeneral.ini)

>
> 教程由以下两部分组成：
>
> - 后端 subconverter
> - 前端 sub-web

## 后端搭建

```dockerfile
docker run -d \
	--restart=always \
	--name subconverter \
	-p 34324:25500 \
	tindy2013/subconverter:latest
	
#-p 外部映射端口自选，与下方同步即可
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
> #--name 根据需要更改
>

使用`docker ps`查看运行中的容器。

![container](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271931496.png)

运行之后，使用以下命令查看状态

```bash
curl http://localhost:34324/version
# 如果出现`subconverter vx.x.x backend`字样，表明运行成功
```

![success](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271930936.png)

根据需要为后端绑定域名，配置反代（可选，可以直接用`ip:port`访问）

> 以下示意图引用自博主另一篇文章，端口不对应，注意更改

在域名提供商处设置`DNS`记录

当然，添加网站和`SSL`证书申请是必要的前期工作。

![添加网站](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271934061.png)

![申请证书](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271934224.png)

开启反向代理，代理到本地相应端口。

![settings](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271934112.png)

![reproxy](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271934964.png)

## 前端搭建

使用`docker`运行前端需要更改文件后自行构建镜像。

```bash
# Clone项目到本地 
git clone https://github.com/CareyWang/sub-web.git
cd sub-web 
```

```bash
# 编辑.env配置文件(也可搭配其他文件编辑器，如VScode)
vi .env 
```

进入`.env`

```bash
# 没有其他需求就只改`API后端`就行。

VUE_APP_PROJECT = "https://github.com/CareyWang/sub-web"

VUE_APP_BOT_LINK = "https://t.me/subconverter_discuss"

VUE_APP_BACKEND_RELEASE = "https://github.com/tindy2013/subconverter/actions"

VUE_APP_SUBCONVERTER_REMOTE_CONFIG = "https://raw.githubusercontent.com/tindy2013/subconverter/master/base/config/example_external_config.ini"

# API 后端，填写自己绑定的域名，或者`ip:port`
VUE_APP_SUBCONVERTER_DEFAULT_BACKEND = "https://sub.wisedrifter.top"

# 短链接后端（可选）
VUE_APP_MYURLS_API = "https://suo.yt/short"

# 文本托管后端
VUE_APP_CONFIG_UPLOAD_API = "https://oss.wcc.best/upload"

# 页面配置
VUE_APP_USE_STORAGE = true 
VUE_APP_CACHE_TTL = 86400
```

```bash
# 编辑subconverter.vue
cd src/views 
vi Subconverter.vue  
```

进入`subconverter.vue`

```bash
# 修改下列内容

# 修改后端，填你刚才搭建的后端，注意后面的`/sub?`
backendOptions: [{ value: "https://sub.wisedrifter.top/sub?" }],

# 修改远程配置，第一个为博主自己写的配置，包含`Disney+`,`Netflix`,`Prime Video`,`OneDrive`等常用分流
remoteConfig: [
          {
            label: "universal",
            options: [
              {
                label:"Konrad自用分流，（Github：MegaSuite）",
                value:
                "https://raw.githubusercontent.com/MegaSuite/Clash-Configs/main/Policy/ClashGeneral.ini"
              },
              {
                label: "ACL4SSR_Online 默认版 分组比较全 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online.ini"
              },
              {
                label: "ACL4SSR_Online_AdblockPlus 更多去广告 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_AdblockPlus.ini"
              },
              {
                label: "ACL4SSR_Online_NoAuto 无自动测速 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoAuto.ini"
              },
              {
                label: "ACL4SSR_Online_NoReject 无广告拦截规则 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoReject.ini"
              },
              {
                label: "ACL4SSR_Online_Mini 精简版 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini.ini"
              },
              {
                label: "ACL4SSR_Online_Mini_AdblockPlus.ini 精简版 更多去广告 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_AdblockPlus.ini"
              },
              {
                label: "ACL4SSR_Online_Mini_NoAuto.ini 精简版 不带自动测速 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_NoAuto.ini"
              },
              {
                label: "ACL4SSR_Online_Mini_Fallback.ini 精简版 带故障转移 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_Fallback.ini"
              },
              {
                label: "ACL4SSR_Online_Mini_MultiMode.ini 精简版 自动测速、故障转移、负载均衡 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_MultiMode.ini"
              },
              {
                label: "ACL4SSR_Online_Full 全分组 重度用户使用 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full.ini"
              },
              {
                label: "ACL4SSR_Online_Full_NoAuto.ini 全分组 无自动测速 重度用户使用 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_NoAuto.ini"
              },
              {
                label: "ACL4SSR_Online_Full_AdblockPlus 全分组 重度用户使用 更多去广告 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_AdblockPlus.ini"
              },
              {
                label: "ACL4SSR_Online_Full_Netflix 全分组 重度用户使用 奈飞全量 (与Github同步)",
                value:
                  "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_Netflix.ini"
              },
              {
                label: "ACL4SSR 本地 默认版 分组比较全",
                value: "config/ACL4SSR.ini"
              },
              {
                label: "ACL4SSR_Mini 本地 精简版",
                value: "config/ACL4SSR_Mini.ini"
              },
              {
                label: "ACL4SSR_Mini_NoAuto.ini 本地 精简版+无自动测速",
                value: "config/ACL4SSR_Mini_NoAuto.ini"
              },
              {
                label: "ACL4SSR_Mini_Fallback.ini 本地 精简版+fallback",
                value: "config/ACL4SSR_Mini_Fallback.ini"
              },
              {
                label: "ACL4SSR_BackCN 本地 回国",
                value: "config/ACL4SSR_BackCN.ini"
              },
              {
                label: "ACL4SSR_NoApple 本地 无苹果分流",
                value: "config/ACL4SSR_NoApple.ini"
              },
              {
                label: "ACL4SSR_NoAuto 本地 无自动测速 ",
                value: "config/ACL4SSR_NoAuto.ini"
              },
              {
                label: "ACL4SSR_NoAuto_NoApple 本地 无自动测速&无苹果分流",
                value: "config/ACL4SSR_NoAuto_NoApple.ini"
              },
              {
                label: "ACL4SSR_NoMicrosoft 本地 无微软分流",
                value: "config/ACL4SSR_NoMicrosoft.ini"
              },
              {
                label: "ACL4SSR_WithGFW 本地 GFW列表",
                value: "config/ACL4SSR_WithGFW.ini"
              }
            ]
          },
        ]
```

代码结构如下：

![subconverter.vue](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271947399.png)

保存之后，进行镜像构建。

> 注意最后的"`.`"，不要漏掉。
>
> 冒号前的`sub-web`是目前存放前端的文件夹名
>
> 构建命令需要在前端文件夹内执行，否则会提醒`找不到Dockerfile`

```dockerfile
#构建镜像
docker build -t sub-web:latest .  
#查看镜像是否构建成功,有同名镜像即为成功
docker images
#部署服务 
docker run -d \
	-p 34325:80 \
	--restart always \
	--name subweb \
	sub-web:latest
```

执行`docker ps`查看状态

![sub-web](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310271957970.png)

浏览器中输入`ip:port`可以看到网页，如`123.11.112.111:34325`

![page](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310272000366.png)

同样可以为前端设置域名和反代，跟前面的流程相同，不赘述。

到现在为止，一个完整的私有订阅转换就搭建完了。

## Reference

[^1]:[tindy2013/subconverter](https://github.com/tindy2013/subconverter)
[^2]:[subconverter/README-docker.md](https://github.com/tindy2013/subconverter/blob/master/README-docker.md)
[^3]:[CareyWang/sub-web](https://github.com/CareyWang/sub-web)
[^4]:[Subconverter+Subweb+MyUrls搭建教程 (Docker版本) - Steve's Blog](https://blog.steveee.me/posts/Subconverter/)
[^5]:[Clash-Configs/Policy/ClashGeneral.ini](https://github.com/MegaSuite/Clash-Configs/blob/main/Policy/ClashGeneral.ini)
