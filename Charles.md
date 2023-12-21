# Charles

> Charles是一款网络代理工具，主要用于监控计算机与互联网之间的数据通信。它允许用户查看计算机与互联网之间通过HTTP和HTTPS发送的数据，提供了重发请求、修改请求和响应等功能，这使得它成为开发人员、测试人员和网络管理员常用的工具之一

# 1 安装使用教程

## 1.1 下载安装

> 1. 下载链接 ：[Download a Free Trial of Charles](https://www.charlesproxy.com/latest-release/download.do)
> 
> 2. 下载流程  ：选择对应系统的版本即可下载，安装使用默认配置

## 1.2 破解

> 1. 在线破解链接 ：[Charles破解工具](https://www.zzzmode.com/mytools/charles/)
> 
> 2. 破解教程 ：输入RegisterName，点击生成注册码。打开Charles输入RegisterName以及注册码

## 1.3 可能的报错信息及解决方法

# 2 配置

## 2.1 Charles配置

### 2.1.1 安装Charles证书

> 1. 进入 【Charles】
> 
> 2. 导航栏点击 【Help – SSL Proxying – Install Charles Root Ceriticate】
> 
> 3. 点击 【安装证书】
> 
> 4. 选择 【本地计算机】
> 
> 5. 选择 【将所有的证书放入下列储存】，点击【浏览】 
> 
> 6. 选择 【受信任的根证书颁发机构】， 点击 【确定】
> 
> 7. 点击 【下一步】
> 
> 8. 点击 【完成】
> 
> 9. 若出现安全警告，点击 【是】

### 2.1.2 配置Charles代理

> 1. 进入 【Charles】
> 
> 2. 导航栏点击 【Proxy - SSL Proxying Settings】
> 
> 3. 点击 【Add】
> 
> 4. 【Host】和【Port】设置为 *，表示全部监听
> 
> 5. 点击【OK】

### 2.1.3 导出 Charles证书

> 1. 进入 【Charles】
> 
> 2. 导航栏点击 【Help – SSL Proxying – Save Charles Root Certificate....】
> 
> 3. 设置存放文件夹及文件名
> 
> 4. 选择保存类型为 【Binary certificate (.cer)】
> 
> 5. 点击保存

## 2.2 web配置（以firefox为例）

[Charles （火狐）抓包设置](https://www.cnblogs.com/f-ichigo/p/13402985.html)

### 2.2.1 设置使用系统代理

> 1. 打开 【火狐浏览器】
> 
> 2. 点击 【设置-常规-网络设置-设置-使用系统代理】

### 2.2.2 导入Charles证书

> 1. 打开 【火狐浏览器】
> 
> 2. 点击 【设置-隐私与安全-证书-查看证书】
> 
> 3. 选择 【证书颁发机构】
> 
> 4. 点击 【导入】
> 
> 5. 找到保存的Charles证书，点击【打开】

## 2.3 ios移动端配置

### 2.3.1 获取Charles代理地址

> 1. 打开【Charles】
> 
> 2. 导航栏点击 【Help-Local IP Address】，查看电脑IP地址以及Charles端口

### 2.3.2 iOS使用Charles代理

> 1. 确保【Charles】与【iOS】处于同一局域网下
> 
> 2. 打开【iOS移动端】
> 
> 3. 点击【设置-WiFi】
> 
> 4. 选择使用的局域网，进入局域网配置
> 
> 5. 点击【配置代理】
> 
> 6. 选择【手动】，输入Charles代理地址对应的服务器与端口
> 
> 7. 点击 【存储】

### 2.3.3 安装Charles证书

> 1. 打开iOS移动端
> 
> 2. 打开 【Safari】
> 
> 3. 访问 [Charles证书下载](chls.pro/ssl)，下载证书
> 
> 4. 打开 【设置】
> 
> 5. 点击 【通用-关于本机-证书信任设置】
> 
> 6. 勾选 【Charles】证书

## 2.4 android 配置（以小米为例）

### 2.4.1 获取Charles代理地址

> 1. 打开【Charles】
> 
> 2. 导航栏点击 【Help-Local IP Address】，查看电脑IP地址以及Charles端口

### 2.3.2 android使用Charles代理

> 1. 确保【Charles】与【android】处于同一局域网下
> 
> 2. 打开【android】移动端
> 
> 3. 点击【设置-WiFi】
> 
> 4. 选择使用的局域网，进入局域网配置
> 
> 5. 点击【配置代理】
> 
> 6. 选择【手动】，输入Charles代理地址对应的服务器与端口
> 
> 7. 点击 【存储】

### 2.3.3 安装Charles证书

> 1. 打开【android】移动端
> 
> 2. 打开 【浏览器】
> 
> 3. 访问 [Charles证书下载](chls.pro/ssl)，下载证书（格式需为【.crt】）
> 
> 4. 打开 【设置】，安装下载的【Charles】证书

# 3 常见问题

## 3.1 iOS端异常

### 3.1.1 iOS使用Charles代理时 ，App Store无法使用

> 1. 打开【Charles】
> 
> 2. 导航栏点击 【Proxy - SSL Proxying Settings】
> 
> 3. 在【Exclude】窗口中点击【Add】 
> 
> 4. 添加 : 【Host】和【Port】分别设置为 \*apple* 和 空
> 
> 5. 添加 : 【Host】和【Port】分别设置为 \*mzstatic* 和 空
> 
> 6. 保存重启【Charles】

# 4 使用

## 4.1 限流（Throttle）

## 4.2 断点(Brekingpoint)
