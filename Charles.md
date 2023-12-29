# Charles

> 【Charles】是一款网络代理工具，用于监控计算机与互联网之间的数据通信，允许用户查看计算机与互联网之间通过HTTP和HTTPS发送的数据，提供了重发请求、修改请求和响应等功能

# 1 安装

## 1.1 下载安装

> 1. 下载链接 ：[Download a Free Trial of Charles](https://www.charlesproxy.com/latest-release/download.do)
> 
> 2. 下载流程  ：选择对应系统的版本即可下载，安装使用默认配置

## 1.2 破解

> 1. 在线破解链接 ：[Charles破解工具](https://www.zzzmode.com/mytools/charles/)
> 
> 2. 破解教程 ：输入自定义的【RegisterName】，点击生成注册码。打开Charles输入设置的【RegisterName】以及对应的注册码

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

> [Charles （火狐）抓包设置](https://www.cnblogs.com/f-ichigo/p/13402985.html)

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

> 1. Only for selected hosts ：勾选后，只对添加的hosts做网速限制，其他的hosts还是正常网速访问
> 
> 2. Throttle perset ：提供几个预设的网络模拟场景
> 
> 3. Bandwidth（带宽）【吞吐量】 ：带宽定义数据可以传送超过时间上限，这是千比特每秒指定。可以指定上载和下载链接的不同带宽限制。
> 
> 4. Utilisation（利用） ：利用率是总带宽的百分比，可以在任何一个时间使用。它只是作为可用带宽的缩放因子。对于大多数现代互联网连接利用率始终是100%。
> 
> 5. Round-trip Latency（请求往返延迟）【延时】 ：往返延迟测量客户端和远程服务器之间的第一次往返通信的毫秒延迟。它用于客户端向服务器和服务器向客户端的每次请求。
> 
> 6. MTU（最大传输单元） ：在任何传输的TCP数据包的最大尺寸。指定MTU不改变的可用带宽，但允许Charles在MTU分配带宽大小的块，导致在每个传输包分割的现实水平。
> 
> 7. Reliability（可靠性）【丢包】 ：可靠性是衡量连接完全失败的可能性。这是非常有用的模拟不可靠的网络条件。可靠性是指定为成功发射10kib消息的可能性，所以，值为50%意味着所有10kib传输一半会成功。较大的邮件或更小的消息或多或少都有可能失败，所以20kib传输将只有25%的成功率和5kib传输成功率约70%。
> 
> 8. Stability（稳定性）【抖动】 ：稳定性是衡量一个连接的可能性是不稳定的，因此降低了质量。这是非常有用的模拟网络，如移动网络，定期连接质量差。如果连接不稳定，则连接的质量会在不稳定的质量范围内随机下降。此质量值，然后应用作为另一个缩放因子的可用带宽。
> 
> 9. unstable quality range（不稳定质量范围）：此处设置主要针对于Stability中设置中的范围

## 4.2 断点(Brekingpoint)
