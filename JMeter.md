# JMeter

# 1 安装

> 1. 下载链接 ：[JMeter下载](https://jmeter.apache.org/download_jmeter.cgi)
> 
> 2. 安装教程 ：解压到相应路径即可，无需安装

# 2 配置

> 配置教程 ：[jmeter环境配置](https://blog.csdn.net/kk_lzvvkpj/article/details/132415348)

## 2.1 配置环境变量

> 1. 打开 windows 【高级系统设置】
> 
> 2. 点击【环境变量】，在【系统变量】窗口下
> 
> 3. 新建变量，变量名为 【JMETER_HOME】，变量值为JMeter解压路径
> 
> 4. 新建变量，变量名为【CLASSPATH】 ，变量值为
>    
>    %JMETER_HOME%\lib\ext\ApacheJMeter_core.jar;%JMETER_HOME%\lib\jorphan.jar;%JMETER_HOME%\lib\log4j-core-2.8.2.jar
> 
> 5. 双击 【path】变量，点击【新建】，变量值为 【%JMETER_HOME%\bin】

### 2.2 Jmeter-plugins-manager安装

### 2.2.1 下载安装

> 1. 下载地址 ：[Jmeter-plugins-manager](https://jmeter-plugins.org/install/Install/)
> 
> 2. 安装 ：将下载的jar文件放在jmeter安装目录下的【/lib/ext】路径中，重启Jmeter，在导航栏【选项-Plugins Manager】中打开Plugins Manager

### 2.2.2 常用插件

| 插件名称                    | 功能                   |
|:-----------------------:|:--------------------:|
| Custom JMeter Functions | 支持Base64加解密等多个函数的插件  |
| PerfMon                 | 监控服务器性能指标，CPU、内存、IO等 |

# 3 使用

> Jmeter解压目录下的bin中，使用jmeter.bat打开
