---
title: 利用Cloudflare自建Github加速代理
date: 2023-09-14 20:09:13
permalink: Cloudflare-Github-Proxy/
tags:
- Github
- Cloudflare
categories:
- Major
- Proxies
index_img: https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202309142011300.png
excerpt: 基于cloudflare域名托管和cloudflare worker
---

# 利用Cloudflare自建Github加速代理

{% note success %}

本教程基于`Cloudflare`域名托管和`Cloudflare Worker`搭建

{% endnote %}

## 基础

进入Cloudflare官网注册账号（这个应该不用教了吧），点击`dashboard`左下方的`Workers&Pages`->`Overview`->`Create application`->`Create Worker`创建`Worker`

![Create](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241859096.png)

![worker](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241900109.png)

填写`Name`项（内容随便，记得住就行），点击右下方`Deploy`，代码先不用管。

![Deploy](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241902964.png)

在接下来出现的页面中点击`Edit code`，填入示例代码。

![edit](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241903088.png)

```javascript
'use strict'

const ASSET_URL = 'https://megasuite.github.io/ghproxy-page/'
//此地址为直接访问加速地址所展示的静态页面，为笔者自己部署于`Github Pages`的页面，可根据需要自己改动。
const PREFIX = '/'
// 前缀，如果自定义路由为example.com/gh/*，将PREFIX改为 '/gh/'，注意，少一个杠都会错！
const Config = {
    jsdelivr: 0
}// 分支文件使用jsDelivr镜像的开关，0为关闭，默认关闭

const whiteList = [] // 白名单，路径里面有包含字符的才会通过，e.g. ['/username/']

/** @type {RequestInit} */
const PREFLIGHT_INIT = {
    status: 204,
    headers: new Headers({
        'access-control-allow-origin': '*',
        'access-control-allow-methods': 'GET,POST,PUT,PATCH,TRACE,DELETE,HEAD,OPTIONS',
        'access-control-max-age': '1728000',
    }),
}


const exp1 = /^(?:https?:\/\/)?github\.com\/.+?\/.+?\/(?:releases|archive)\/.*$/i
const exp2 = /^(?:https?:\/\/)?github\.com\/.+?\/.+?\/(?:blob|raw)\/.*$/i
const exp3 = /^(?:https?:\/\/)?github\.com\/.+?\/.+?\/(?:info|git-).*$/i
const exp4 = /^(?:https?:\/\/)?raw\.(?:githubusercontent|github)\.com\/.+?\/.+?\/.+?\/.+$/i
const exp5 = /^(?:https?:\/\/)?gist\.(?:githubusercontent|github)\.com\/.+?\/.+?\/.+$/i
const exp6 = /^(?:https?:\/\/)?github\.com\/.+?\/.+?\/tags.*$/i

/**
 * @param {any} body
 * @param {number} status
 * @param {Object<string, string>} headers
 */
function makeRes(body, status = 200, headers = {}) {
    headers['access-control-allow-origin'] = '*'
    return new Response(body, {status, headers})
}


/**
 * @param {string} urlStr
 */
function newUrl(urlStr) {
    try {
        return new URL(urlStr)
    } catch (err) {
        return null
    }
}


addEventListener('fetch', e => {
    const ret = fetchHandler(e)
        .catch(err => makeRes('cfworker error:\n' + err.stack, 502))
    e.respondWith(ret)
})


function checkUrl(u) {
    for (let i of [exp1, exp2, exp3, exp4, exp5, exp6]) {
        if (u.search(i) === 0) {
            return true
        }
    }
    return false
}

/**
 * @param {FetchEvent} e
 */
async function fetchHandler(e) {
    const req = e.request
    const urlStr = req.url
    const urlObj = new URL(urlStr)
    let path = urlObj.searchParams.get('q')
    if (path) {
        return Response.redirect('https://' + urlObj.host + PREFIX + path, 301)
    }
    // cfworker 会把路径中的 `//` 合并成 `/`
    path = urlObj.href.substr(urlObj.origin.length + PREFIX.length).replace(/^https?:\/+/, 'https://')
    if (path.search(exp1) === 0 || path.search(exp5) === 0 || path.search(exp6) === 0 || path.search(exp3) === 0 || path.search(exp4) === 0) {
        return httpHandler(req, path)
    } else if (path.search(exp2) === 0) {
        if (Config.jsdelivr) {
            const newUrl = path.replace('/blob/', '@').replace(/^(?:https?:\/\/)?github\.com/, 'https://cdn.jsdelivr.net/gh')
            return Response.redirect(newUrl, 302)
        } else {
            path = path.replace('/blob/', '/raw/')
            return httpHandler(req, path)
        }
    } else if (path.search(exp4) === 0) {
        const newUrl = path.replace(/(?<=com\/.+?\/.+?)\/(.+?\/)/, '@$1').replace(/^(?:https?:\/\/)?raw\.(?:githubusercontent|github)\.com/, 'https://cdn.jsdelivr.net/gh')
        return Response.redirect(newUrl, 302)
    } else {
        return fetch(ASSET_URL + path)
    }
}


/**
 * @param {Request} req
 * @param {string} pathname
 */
function httpHandler(req, pathname) {
    const reqHdrRaw = req.headers

    // preflight
    if (req.method === 'OPTIONS' &&
        reqHdrRaw.has('access-control-request-headers')
    ) {
        return new Response(null, PREFLIGHT_INIT)
    }

    const reqHdrNew = new Headers(reqHdrRaw)

    let urlStr = pathname
    let flag = !Boolean(whiteList.length)
    for (let i of whiteList) {
        if (urlStr.includes(i)) {
            flag = true
            break
        }
    }
    if (!flag) {
        return new Response("blocked", {status: 403})
    }
    if (urlStr.startsWith('github')) {
        urlStr = 'https://' + urlStr
    }
    const urlObj = newUrl(urlStr)

    /** @type {RequestInit} */
    const reqInit = {
        method: req.method,
        headers: reqHdrNew,
        redirect: 'manual',
        body: req.body
    }
    return proxy(urlObj, reqInit)
}


/**
 *
 * @param {URL} urlObj
 * @param {RequestInit} reqInit
 */
async function proxy(urlObj, reqInit) {
    const res = await fetch(urlObj.href, reqInit)
    const resHdrOld = res.headers
    const resHdrNew = new Headers(resHdrOld)

    const status = res.status

    if (resHdrNew.has('location')) {
        let _location = resHdrNew.get('location')
        if (checkUrl(_location))
            resHdrNew.set('location', PREFIX + _location)
        else {
            reqInit.redirect = 'follow'
            return proxy(newUrl(_location), reqInit)
        }
    }
    resHdrNew.set('access-control-expose-headers', '*')
    resHdrNew.set('access-control-allow-origin', '*')

    resHdrNew.delete('content-security-policy')
    resHdrNew.delete('content-security-policy-report-only')
    resHdrNew.delete('clear-site-data')

    return new Response(res.body, {
        status,
        headers: resHdrNew,
    })
}

```

点击`Send`出现`Status Code: 200`即为搭建成功。红线下的地址即为你的私有加速地址。

![Code](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241904387.png)

## 进阶

`Cloudflare`默认的域名显得些许冗长，而且这也是一个可以长期使用的项目，所以我们可以考虑使用自己的域名。

### 托管域名

首先，需要拥有一个托管于`Cloudflare`的域名，或者通过`Cloudflare`注册一个，下图展示了简单的托管教程。

![add site](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241922890.png)

填写要托管的域名，图上我是随便写的啊。

![填写要托管的域名](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241924563.png)

`Plan`选择`free`即可，一天一百万次，个人完全够了。

![plan](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241925872.png)

根据提示修改`DNS`服务器即可，之后会需要一段时间等待解析生效，成功之后会有邮件通知托管成功。

![dns server](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241926961.png)

### 添加路由

进入托管成功的域名管理页，在`DNS`->`Records`->`Add record`添加一条`A`类型解析记录，选择一个你喜欢的前缀（如`ghproxy`）填入`Name`项，`Content`项随便填一个合法的ip地址即可，然后打开橙色小云朵，点击保存。

![dns records](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241932536.png)

然后来到下方添加路由，`Workers Routes`->`Add route`

![Routes](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241937210.png)

`Route`项填入`前缀.域名/*`，注意`*`的位置，`Worker`项选择刚刚创建的`Worker`即可。

![edit route](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202310241938694.png)

保存之后便可以通过自定义域名访问加速站

点击下方链接查看笔者搭建的示例：

{% note primary %}

[GitHub Proxy by Konrad](https://ghproxy.wisedrifter.top/)

{% endnote %}

## Reference

[^1]:[参考项目：hunshcn/ghproxy](https://github.com/hunshcn/gh-proxy)
[^2]:[笔者的静态页面仓库：MegaSuite/ghproxy-page](https://github.com/MegaSuite/ghproxy-page)
