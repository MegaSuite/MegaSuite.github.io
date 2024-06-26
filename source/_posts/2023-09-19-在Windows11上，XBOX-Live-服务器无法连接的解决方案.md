---
title: 在Win11上，XBOX Live无法连接的解决方案
date: 2023-09-19 15:58:44
permalink: XBOX-Live-Error-Windows11/
tags:
- Game
- XBOX
categories:
- Daily
index_img: https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202309191601314.png
excerpt: 畅玩地平线4的解决方案（bushi
---

# 在Windows11上，XBOX Live 服务器无法连接的解决方案

> 在玩地平线4的时候，多人模式总是无法连接，网上的教程又都是基于`Windows10`的，折腾到凌晨两点也没有成功。
>
> 第二天早上突发奇想，跟随一篇教程检查了`ipv6`状态，从而解决了这个问题

## 首先

1、记住下面这行代码，后面我们将多次用到。

> 由于`Windows11`去除了`XBOX网络`选项，我们需要通过这行命令来查看链接状态。

```bash
netsh interface Teredo show state
```

2、学会使用管理员模式运行命令行工具，这里使用`Windows11`自带的终端。开始菜单中搜索得到终端之后，右键`用管理员身份运行`。

我们将在这里进行命令行输入。

![打开terminal](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202309191612414.png)

3、检查自己的`Windows`版本，如果是家庭版（笔记本基本都是)，需要通过运行脚本获取“组策略”选项。

- 新建一个文本文档，比如`a.txt`，将下面的代码粘贴进去。

```bash
  @echo off
  
  pushd "%~dp0"
  
  dir /b C:\Windows\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientExtensions-Package~3*.mum >List.txt
  
  dir /b C:\Windows\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientTools-Package~3*.mum >>List.txt
  
  for /f %%i in ('findstr /i . List.txt 2^>nul') do dism /online /norestart /add-package:"C:\Windows\servicing\Packages\%%i"
  
  pause
```

- 保存文档，修改文件名为`a.cmd`，右键`->`以管理员身份运行

  > 如果无法更改后缀，可用`VS Code`打开，或者[查看此文章](https://www.php.cn/faq/569056.html)

- 按`win`+`R`组合键，输入`gpedit.msc`，回车，若出现组策略编辑窗口则成功。

4、检测本地`ipv6`是否开启，有的加速器会把`ipv6`关掉，比如小黑盒加速器

> `ipv6`的配置方式可以问度娘，或者[查看此文章](https://www.sohu.com/a/322875365_594016)

## 开始

按照上文方式，打开“本地组策略编辑器“，找到计算机配置——管理模版——网络——`TCPIP`设置——`IPv6`转换技术。
将所有配置设置为“未配置”。

![组策略编辑器](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202309191622804.png)

管理员权限运行终端，在其中运行以下命令：

【建议一条一条来】

```bash
# 启用windows防火墙
netsh advfirewall set allprofiles state on
# 修复注册表：
reg add HKLMSystemCurrentControlSetServicesTcpip6Parameters /v DisabledComponents /t REG_DWORD /d 0x0
# 禁用Teredo功能：
netsh interface Teredo set state disable
# 还原teredo状态：
netsh interface Teredo set state servername=default
# 检测teredo状态：
netsh interface Teredo show state
# 设置客户端类型：
netsh interface Teredo set state type=enterpriseclient
# 查看teredo情况，状态异常为disable，正常为dormant / qualified
```

如果服务器无法连接，可以命令行运行下列任意一条命令更改`Teredo`服务器，之后运行`netsh interface Teredo show state`检测，多尝试几次

```bash
netsh interface teredo set state server=win1901.ipv6.microsoft.com
# or
netsh interface teredo set state server=win10.ipv6.microsoft.com

# 可尝试的服务器
teredo2.remlab.net
teredo.iks-jena.de
win10.ipv6.microsoft.com
win1901.ipv6.microsoft.com
teredo.ipv6.microsoft.com
teredo.trex.fi
```

> 以上命令行也可以通过直接在“组策略编辑器”中编辑实现
>
> - 启用防火墙可以通过`Windows Defender`完成
>
> - `设置Teredo默认限定`->已启用
> - `设置Teredo服务器名称`->已启用-填入下列任意服务器，如`win1901.ipv6.microsoft.com`
> - `设置Teredo状态`->已启用-状态选择`企业客户端`

按下`Win` + `R`组合键打开运行，输入`regedit`点击确定打开注册表; 
`HKEY_CURRENT_USER SOFTWARE Microsoft Windows CurrentVersion Internet Settings Connections `,将`Connections`项删除，重启即可正常使用 

> 可以通过`设置`->`网络`->`高级网络设置`->`网络重置`完成

![fail](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202309191629279.png)

![Success!](https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202309191630104.png)

> 上述成功图片所示并不能算是最完美的结果，但开个加速器就可以用了，具体最后的结果与所处网络环境有关，毕竟巨硬的服务器一向比较玄学。。。

如果上述仍然失效，可以尝试使用`奇游加速器`的修复功能。（非广告，笔者修复过程中确实使用过此功能，与上述步骤结合，从而无法确定其是否发挥作用，大家可以尝试一下）

> 环境：湖北某大学校园网，`Windows 11`，地平线4

**地平线，启动！**

## 参考

[^1]:[微软游戏无法正常加入在线游戏的解决教程](https://www.qiyou.cn/games/758.html)
[^2]:[极限竞速：地平线 Teredo不合格解决方案](https://51huanqi.cn/极限竞速：地平线-teredo不合格解决方案/)
