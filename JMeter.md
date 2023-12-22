# JMeter

# 1 安装

1. 下载链接 ：[JMeter下载](https://jmeter.apache.org/download_jmeter.cgi)

2. 安装教程 ：解压即可，无需安装

# 2 配置

配置教程 ：[jmeter环境配置](https://blog.csdn.net/kk_lzvvkpj/article/details/132415348)

## 2.1 配置环境变量

- 打开 windows 【高级系统设置】

- 点击【环境变量】，在【系统变量】窗口下

- 新建变量，名为 【JMETER_HOME】，变量值为JMeter安装路径

- 新建变量，变量为【CLASSPATH】 ，变量值为
  
  %JMETER_HOME%\lib\ext\ApacheJMeter_core.jar;%JMETER_HOME%\lib\jorphan.jar;%JMETER_HOME%\lib\log4j-core-2.8.2.jar

- 双击 【path】变量，点击【新建】，添加变量。变量值为 【%JMETER_HOME%\bin】
