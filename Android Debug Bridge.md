# Android Debug Bridge

> ADB 全称为 Android Debug Bridge，即安卓调试桥，是一个**客户端-服务器端程序**。其中客户端是用来操作的电脑，服务端是 Android 设备。ADB是android sdk里的一个工具, 用这个工具可以直接操作管理android模拟器或者真实的android设备。

# 1 安装

## 1.1 下载

下载链接 ：https://adbdownload.com/，选择对应系统的版本

## 1.2 安装

将压缩包解压即可，无需安装

# 2 配置

## 2.1 配置环境变量

> 1. 打开windows 【高级系统设置】
> 
> 2. 点击【环境变量】
> 
> 3. 双击 【path】变量，点击【新建】，变量值为ADB安装包的解压路径

# 3 使用

5037：adb默认端口

## 3.1 常用命令

> 1. adb version ：显示 adb 版本
> 
> 2. adb help：帮助信息，查看adb所支持的所有命令
> 
> 3. adb devices：查看当前连接的设备，已连接的设备会显示出来
> 
> 4. adb get-serialno：也可以查看设备号
> 
> 5. adb root：获取Android管理员（root用户）的权限
> 
> 6. adb shell：登录设备 shell，该命令将登录设备的shell（内核）
> 
> 7. adb -d：如果同时连了usb，又开了模拟器，连接当前唯一通过usb连接的安卓设备
> 
> 8. adb -e shell：指定当前连接此电脑的唯一的一个模拟器
> 
> 9. adb -s <设备号> shell：当电脑插多台手机或模拟器时，指定一个设备号进行连接
> 
> 10. exit：退出
> 
> 11. adb kill-server：杀死当前adb服务，如果连不上设备时，杀掉重启
> 
> 12. adb start-server：杀掉后重启
> 
> 13. adb -p <端口> start-server：指定一个 adb shell 启动端口
> 
> 14. adb shell pm list packages：列出当前设备/手机，所有的包名
> 
> 15. adb shell pm list packages -f：显示包和包相关联的文件(安装路径)
> 
> 16. adb shell pm list packages -d：显示禁用的包名
> 
> 17. adb shell pm list packages -e：显示当前启用的包名
> 
> 18. adb shell pm list packages -s：显示系统应用包名
> 
> 19. adb shell pm list packages -3：显示已安装第三方的包名
> 
> 20. adb shell pm list packages xxxx：加需要过滤的包名，如：xxx = taobao
> 
> 21. adb install <文件路径\apk>：将本地的apk软件安装到设备(手机)上。如手机外部安装需要密码，记得手机输入密码
> 
> 22. adb install -r <文件路径\apk>：覆盖安装



# 4 工作原理

当启动某个 `adb` 客户端时，该客户端会先检查是否有 `adb` 服务器进程已在运行。如果没有，它会启动服务器进程。服务器在启动后会与本地 TCP 端口 5037 绑定，并监听 `adb` 客户端发出的命令。所有 `adb` 客户端均使用端口 5037 与 `adb` 服务器通信。

然后，服务器会与所有正在运行的设备建立连接。它通过扫描 5555 到 5585 之间（该范围供前 16 个模拟器使用）的奇数号端口查找模拟器。服务器一旦发现 `adb` 守护程序 (adbd)，便会与相应的端口建立连接。

每个模拟器都使用一对按顺序排列的端口：一个用于控制台连接的偶数号端口，另一个用于 `adb` 连接的奇数号端口。例如：

模拟器 1，控制台：5554  
模拟器 1，`adb`：5555  
模拟器 2，控制台：5556  
模拟器 2，`adb`：5557  
依此类推。

如上所示，在端口 5555 处与 `adb` 连接的模拟器与控制台监听端口为 5554 的模拟器是同一个。服务器与所有设备均建立连接后，您便可以使用 `adb` 命令访问这些设备。由于服务器管理与设备的连接，并处理来自多个 `adb` 客户端的命令，因此您可以从任意客户端或从某个脚本控制任意设备。


