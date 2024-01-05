# Fiddler

> Fiddler是位于客户端和服务器端的HTTP代理（proxy）。Fiddler是目前最常用的http抓包工具之一 。Fiddler能够记录客户端和服务器之间的所有 HTTP请求，可以针对特定的HTTP请求，分析请求数据、设置断点（breakpoint）、调试web应用、修改请求（request）的数据，甚至可以修改服务器返回的数据，功能非常强大，是web调试的利器

# 1 下载安装

## 1.1 下载

> 1. 下载链接 ：https://www.telerik.com/download/fiddler
> 
> 2. 下载流程 ：填写用途、邮箱、国家后会跳转下载

## 1.2 安装

>  使用默认配置安装

## 1.3 可能的报错信息及解决方法

# 2 配置

配置教程 ：[https://zhuanlan.zhihu.com/p/410150022](https://zhuanlan.zhihu.com/p/410150022)

[ Fiddler 抓取火狐浏览器 https请求](https://blog.csdn.net/qq_34626094/article/details/113115309)

[https://zhuanlan.zhihu.com/p/37374178](https://zhuanlan.zhihu.com/p/37374178)

## 2.1 HTTPS配置

> 1. 打开【Fiddler】
> 
> 2. 点击【Tools-Options】
> 
> 3. 导航栏切换到【HTTPS】下
> 
> 4. 勾选【Capture HTTPS CONNECTs】、【Decrypt HTTPS traffic】、【Ignore server certificate errors(unsafe)】
> 
> 5. 弹出对话框【SCARY TEXT AHEAD：Read Carefully！】，点击【YES】
> 
> 6. 弹出对话框【Add certificate to the Machine Root List？】，点击【YES】
> 
> 7. 弹出对话框【TrustCert Success】，点击【确定】
> 
> 8. 点击【Options】中的【ok】，保存配置

## 2.2 Connections配置

> 1. 打开【Fiddler】
> 
> 2. 点击【Tools-Options】
> 
> 3. 导航栏切换到【Connections】下
> 
> 4. 在【Fiddler listens on port】内输入【8888】
> 
> 5. 勾选【Allow remote computers to connect】
> 
> 6. 弹出对话框【Enabling Remote Access】，点击【确定】
> 
> 7. 点击【Options】中的【ok】，保存配置

## 2.3 Scripting

> 1. 打开【Fiddler】
> 
> 2. 点击【Tools-Options】
> 
> 3. 导航栏切换到【Scripting】下
> 
> 4. 在【Fiddler listens on port】内输入【8888】
> 
> 5. 自定义脚本语言设置，可以选择【C#】或【Jscript.NET】
> 
> 6. 点击【Options】中的【ok】，保存配置

## 2.4 浏览器配置（FireFox）

### 2.4.1 导出证书

> 1. 打开【Fiddler】
> 
> 2. 点击【Tools-Options】
> 
> 3. 导航栏切换到【HTTPS】下
> 
> 4. 点击【Actions】
> 
> 5. 点击【Export Root Certificate to Desktop】
> 
> 6. 查看桌面是否有【FiddlerRoot.cer】文件
> 
> 7. 点击【Options】中的【ok】，保存配置

### 2.4.2 浏览器导入证书

> 1. 打开 【火狐浏览器】
> 
> 2. 点击 【设置-隐私与安全-证书-查看证书】
> 
> 3. 选择 【证书颁发机构】
> 
> 4. 点击 【导入】
> 
> 5. 找到保存的【FiddlerRoot.cer】证书文件，点击【打开】

### 2.4.3 浏览器配置

> 1. 打开 【火狐浏览器】
> 
> 2. 在地址栏输入 【about:config】
> 
> 3. 搜索 【security.tls.version.max】
> 
> 4. 双击修改为 3
> 
> 5. 重启浏览器

# 3 常见问题及其解决方法

## tunnel to

# 4 使用

Decrypt HTTPS traffic中的选项说明：

```text
from all processes : 
抓取所有的 https 程序, 包括电脑程序和手机APP。
from browsers only : 
只抓取浏览器中的https请求。
from non-browsers only : 
只抓取除了浏览器之外的所有https请求。
from remote clients only：
只抓取远程的客户端的https请求，就是只抓取手机APP上的https请求
```

## 原理

![](https://pic2.zhimg.com/80/v2-f0646108d9572dbbc52c940ca796cd8d_720w.jpg)
