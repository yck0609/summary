# 1 Linux

> Linux是基于POSIX和Unix的多用户、多任务的类Unix操作系统，继承Unix以网络为核心的设计思想，支持多线程和多CPU，能运行主要的Unix工具软件、应用程序和网络协议，支持32位和64位硬件，是一个性能稳定的多用户网络操作系统

## 1.1 Linux的特点

> 1. **开源性**：Linux作为开源软件，其源代码可以被任何人查看、修改和分发。这使得用户可以自由地定制和优化操作系统，同时也促进了全球范围内的合作和创新
> 
> 2. **稳定性和可靠性**：Linux内核能够长时间运行而不需要重新启动，处理大量的并发任务和负载
> 
> 3. **安全性**：Linux具有较高的安全性。开源性使得安全漏洞可以被及时发现和修复，同时Linux社区也提供了强大的安全工具和机制来保护系统免受恶意攻击
> 
> 4. **多用户和多任务支持**：Linux支持多用户同时使用一个系统，并且能够同时运行多个任务。这使得Linux操作系统非常适合服务器环境和高性能计算
> 
> 5. **可移植性**：Linux可以运行在各种硬件平台上，从个人计算机到服务器、嵌入式设备和移动设备等。这种可移植性使得开发者可以在不同的平台上开发和部署应用程序
> 
> 6. **丰富的软件生态系统**：Linux操作系统拥有广泛的开源软件和应用程序，涵盖了各种领域和用途。用户可以从众多的软件包中选择适合自己需求的应用程序，并且可以通过自由软件许可证进行自由使用和分发。
> 
> 7. **强大的命令行界面**：Linux提供了强大的命令行界面，使用户可以通过命令和脚本来管理和操作系统。这种灵活性和强大的命令行工具使得Linux成为许多系统管理员和开发者的首选

## 1.2 Linux 哲学思想

> 1. **小工具**：将系统功能分解为多个小而独立的工具，每个工具专注于做好一件事情。这种模块化设计使得工具更易于理解、维护和扩展
> 2. **文本为界面**：倾向于使用文本界面进行系统管理和操作，如命令行界面。这种设计可以提供更大的灵活性和自动化能力，同时也减少了对图形界面的依赖
> 3. **自由开放**：强调自由软件的使用和开发，用户具有选择、修改和分发软件的自由。这种开放性促进了创新和合作，使得Linux成为一个开放的操作系统
> 4. **统一抽象**：将各种资源抽象为文件形式，使得对它们的访问和管理具有统一的方式。这种设计使得不同的资源可以通过相同的接口进行访问，提高了系统的一致性和可扩展性
> 5. **自动化和编程**：通过脚本和自动化工具，尽量减少用户交互，实现任务的自动化和以编程方式操作系统。这种自动化能力提高了效率和可靠性
> 6. **提供机制而非策略**：提供一系列机制和工具，让用户根据需求选择适合的策略，增强系统的灵活性。这种设计使得用户可以根据自己的需求进行定制和配置
> 7. **假定用户聪明**：假定用户具有一定的技术能力和知识，提供丰富的工具和选项，鼓励用户学习和探索。这种设计使得用户可以更好地理解和掌握系统
> 8. **共享合作**：强调共享和合作的精神，通过社区共享经验和知识，相互帮助解决问题和改进软件。这种合作性使得Linux社区成为一个充满活力和创造力的环境。
>    这些原则共同构成了Linux哲学思想，为Linux操作系统的设计和发展奠定了基础，使其成为一个强大、灵活和可定制的操作系统

## 1.3 Linux 发行版

> 1. **Ubuntu**：Ubuntu是最受欢迎的Linux发行版之一，注重易用性和用户友好性。它有一个活跃的社区和广泛的软件支持，适用于个人用户和企业用户
> 
> 2. **Debian**：Debian是一个稳定且可靠的发行版，以其强调自由软件和开放源代码而闻名。它采用包管理器APT来管理软件包，适用于服务器和桌面环境
> 
> 3. **Fedora**：Fedora是一个面向开发者和技术爱好者的发行版，它采用最新的开源技术，并以其先进的功能和安全性而闻名
> 
> 4. **CentOS**：CentOS是基于Red Hat Enterprise Linux（RHEL）源代码的免费发行版，注重稳定性和安全性，适用于服务器环境
> 
> 5. **Arch Linux**：Arch Linux是一个轻量级和灵活的发行版，注重简单性和自定义性。它采用滚动更新模型，允许用户始终使用最新的软件版本
> 
> 6. **openSUSE**：openSUSE是一个注重用户友好性和稳定性的发行版，它提供了易于使用的图形界面和强大的软件管理工具
> 
> 7. **Linux Mint**：Linux Mint是一个基于Ubuntu的发行版，注重易用性和用户体验。它提供了一个简洁而直观的桌面环境，适合新手用户
> 
> 8. **Gentoo**：Gentoo是一个源代码驱动的发行版，用户可以根据自己的需求自定义安装和配置。它注重性能和灵活性

## 1.4 Linux 安装与配置

> 安装WSL2（Windows Subsystem for Linux 2）和Ubuntu是在Windows系统上运行Linux发行版的一种方法

### 1.4.1 启用WSL功能

1. 打开Windows PowerShell（管理员权限）

2. 运行以下命令启用WSL功能：
   
   ```
   dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
   ```

3. 运行以下命令启用虚拟机平台功能：
   
   ```
   dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
   ```

4. 重启计算机

### 1.4.2 安装WSL2 Linux内核更新包

访问[WSL2 Linux内核更新包](https://aka.ms/wsl2kernel)页面，下载并安装适用于你的Windows版本的WSL2 Linux内核更新包

### 1.4.3 设置WSL2为默认版本

1. 打开Windows PowerShell（管理员权限）
2. 运行以下命令以将WSL2设置为默认版本：

```
wsl --set-default-version 2
```

### 1.4.4 安装Ubuntu

1. 打开Microsoft Store，搜索"Ubuntu"并选择适合你的版本（例如Ubuntu 20.04 LTS）
2. 点击"获取"按钮以开始安装
3. 安装完成后，点击"启动"按钮以启动Ubuntu

## 1.5 Linux 运行级别（Runlevel）

> 运行级别是指Linux系统在不同模式下的运行状态。每个运行级别定义了一组特定的服务和功能，决定了系统启动时运行哪些进程以及如何配置系统。有七个标准运行级别（0到6）以及一个特殊的运行级别"S"，可以使用`runlevel`命令查看当前运行级别，使用`init`或`telinit`命令切换运行级别。运行级别的配置文件位于`/etc/inittab`或`/etc/init`目录中，可以根据需要进行自定义配置

1. **运行级别0**：关机模式，系统停止运行，并关闭电源
2. **运行级别1**：单用户模式，只有一个控制台可用，用于系统维护和故障排除
3. **运行级别2**：多用户模式（无网络连接），系统启动时只加载最基本的服务，没有网络连接
4. **运行级别3**：多用户模式（完全），系统启动时加载所有必要的服务，包括网络连接
5. **运行级别4**：保留未使用，可以自定义
6. **运行级别5**：图形用户界面（GUI）模式，系统启动时加载图形界面和相关服务
7. **运行级别6**：重新启动模式，系统停止运行并重新启动
8. **运行级别"S"**：也称为"单用户模式"或"系统初始化模式"，需要 root 用户权限才能进入该模式，系统启动到一个最小的环境，用于系统维护和故障修复

## 1.6 Linux 架构

### 1.6.1 Linux内核

> Linux内核是Linux操作系统的核心部分，它是操作系统与硬件之间的桥梁。内核负责管理系统资源、提供基本的功能和服务，并与硬件设备进行交互。它处理进程管理、内存管理、文件系统、设备驱动程序、网络协议等方面的功能

### 1.6.2 Shell

> Shell是用户与操作系统进行交互的命令行解释器。常见的Linux Shell包括Bash、Zsh、Fish等，它们提供了命令行环境和脚本编写功能

### 1.6.3 GNU工具

> GNU工具集是一组开源的命令行工具和实用程序，用于完成各种系统管理和开发任务。其中包括文本编辑器（如Vim、Emacs）、编译器（如GCC）、调试器（如GDB）等

### 1.6.4 图形服务器（X Server）

图形服务器负责显示图形界面，并处理用户的输入。X Server通常与窗口管理器（Window Manager）和桌面环境（Desktop Environment）一起使用，提供了直观的图形界面。

### 1.6.5 窗口管理器（Window Manager）

> 窗口管理器控制窗口的布局、外观和行为。它负责窗口的打开、关闭、移动和调整大小等操作

### 1.6.6 桌面环境（Desktop Environment）

> 桌面环境是一个完整的用户界面，包括窗口管理器、面板、文件管理器、应用程序启动器等。常见的桌面环境有GNOME、KDE、XFCE等

### 1.6.7 文件系统（File System）

> 文件系统用于组织和管理文件和目录。常见的Linux文件系统包括Ext4、XFS、Btrfs等，它们提供了文件的存储、访问和权限控制功能

### 1.6.8 软件包管理器（Package Manager）

> 软件包管理器用于安装、升级和删除软件包。常见的Linux软件包管理器有APT（Debian/Ubuntu）、DNF（Fedora）和Pacman（Arch Linux）等

### 1.6.9 网络协议栈（Network Stack）

> 网络协议栈实现了网络通信的协议和功能，包括TCP/IP协议、网络设备驱动程序和网络配置工具等

### 1.6.10 系统服务（System Services）

> 系统服务是在后台运行的程序，提供各种功能和服务，如网络服务（如SSH、HTTP）、打印服务、时间同步服务等

# 2 快捷键

1. **Ctrl + C**：中断当前运行的程序或命令
2. **Ctrl + D**：在终端中输入EOF（End of File）字符，通常用于退出终端会话或结束输入
3. **Ctrl + Z**：将当前运行的程序暂停，并将其放入后台
4. **Ctrl + A**：将光标移动到命令行的开头
5. **Ctrl + E**：将光标移动到命令行的末尾
6. **Ctrl + L**：清除终端屏幕，并将光标移动到顶部
7. **Ctrl + R**：在历史命令中进行反向搜索
8. **Ctrl + S**：暂停终端屏幕输出，按下Ctrl + Q继续输出
9. **Ctrl + U**：删除光标之前的所有字符
10. **Ctrl + W**：删除光标之前的一个单词
11. **Ctrl + K**：删除光标之后的所有字符
12. **Ctrl + Y**：粘贴之前使用Ctrl + U、Ctrl + K或Ctrl + W删除的文本
13. **Ctrl + P**：显示上一个命令（历史命令）
14. **Ctrl + N**：显示下一个命令（历史命令）
15. **Ctrl + T**：交换光标位置前后的两个字符
16. **Ctrl + H**：删除光标之前的一个字符，相当于Backspace键
17. **Ctrl + ]**：用于telnet或ssh连接时，发送特殊命令
18. tab 补齐
19. ctrl + shift + t 打开很多终端

# 3 Linux 命令

> 命令本身是一个可执行的二进制格式程序文件，发起命令即请求内核将某个二进制程序运行为一个进程，静态转为动态

## 3.1 命令语法

```shell
COMMAND [OPTIONS...] [ARGUMENTS...]
```

- `command`是命令名称，是执行特定操作的关键字

- `options`是命令选项，用于修改命令行为

- `arguments`是命令的参数，提供命令执行所需的输入信息

- `[]`表示可选内容 

- `<>` 表示必须提供的内容

- `a|b` 表示多选一 

- `...` 表示同类内容可出现多个 

### 3.1.1 选项（OPTIONS）

> 选项用于调整命令的运行特性，修改命令行为
> 
> Linux命令选项是用于修改命令行行为的标记或参数。它们通常以短横线（-）或双短横线（--）开头，后跟一个字母、单词或字母组合。命令选项可以提供额外的功能、修改默认行为、控制输出格式等

1. 短选项（Short Options）：短选项通常由单个字母组成，前缀为单个短横线（-）
2. 长选项（Long Options）：长选项通常由一个或多个单词组成，前缀为双短横线（--）
3. 组合选项（Combined Options）：组合选项是多个短选项的组合，用于在一条命令中同时指定多个选项，减少命令的长度，提高命令的效率
4. 布尔选项（Boolean Options）：布尔选项是一种特殊类型的选项，通常用于开启或关闭某些功能，没有参数值，提供选项表示开启某功能，省略选项表示关闭该功能
5. 参数选项（Argument Options）：参数选项后面需要紧跟参数值，用于提供进一步的信息

### 3.1.2 参数（ARGUMENTS）

> 命令参数是在执行命令时提供的值，跟在命令选项后，并以空格分隔，用于指定命令的操作对象、配置选项或其他必要的信息

1. 位置参数：位置参数是指按照特定顺序提供的参数值，命令根据位置参数的顺序来解析和使用这些参数。`cp file1 file2`中的`file1`和`file2`就是两个位置参数，用于指定源文件和目标文件
2. 选项参数：选项参数是指用于配置命令行为的参数值，通常与选项一起使用。`grep -i "pattern" file.txt`中的`-i`选项需要一个选项参数`"pattern"`
3. 环境变量参数：环境变量参数是指通过环境变量传递给命令的参数值。环境变量是在操作系统中定义的一些全局变量，可以在命令行中使用或通过配置文件设置。例如，`echo $PATH`中的`$PATH`是一个环境变量参数，用于显示系统的路径配置
4. 标准输入参数：标准输入参数是指通过标准输入流（stdin）传递给命令的参数值。标准输入通常用于从文件、管道或重定向输入数据。例如，`cat > file.txt`中的`>`符号用于将标准输入保存到`file.txt`文件中
5. 通配符参数：通配符参数用于匹配文件名或路径模式。通配符可以用于批量操作文件或选择特定的文件。例如，`rm *.txt`中的`*.txt`是一个通配符参数，用于删除所有以`.txt`为扩展名的文件
6. 命令替换参数：命令替换参数允许将一个命令的输出作为参数值传递给另一个命令。命令替换可以使用反引号或美元符号加括号来实现

## 3.2 命令分类

> 按照命令的类型进行分类，内置命令、外部命令和别名命令

### 3.2.1 内置命令（Built-in Commands）

> 内置命令内置在Shell解释器中，不需要额外的可执行文件

### 3.2.2 外部命令（External Commands）

> 外部命令由系统管理员或第三方开发者编写，用于提供特定的功能和操作，位于系统的某个可执行文件路径下，可以通过Shell来调用和执行，是独立的可执行文件

### 3.2.3 别名命令（Alias Commands）

> 别名命令是一种用户自定义的命令，用于简化常用命令的输入。通过为一个或多个命令设置别名，可以将复杂或冗长的命令映射为一个简短的别名

- 别名命令不会替换命令的选项和参数。如果要为别名命令添加选项和参数，可以使用`$@`表示传递给别名的所有参数
- 别名命令的定义不支持换行。如果命令序列很长，可以使用反斜杠(`\`)来继续命令到下一行
- 别名命令的定义会覆盖同名的外部命令。如果定义了与系统已有命令同名的别名命令，那么在使用该命令时将执行别名命令而不是系统命令。如果需要执行系统命令，可以使用绝对路径或者使用命令的完整路径

#### 3.2.3.1 创建别名命令

> 使用`alias`命令创建别名命令，后跟别名和对应的命令序列，别名和命令序列之间使用等号(`=`)分隔

```
alias ll='ls -l'
```

#### 3.2.3.2 查看别名命令

> 使用`alias`命令，不带任何参数，查看当前定义的别名命令，将显示所有已定义的别名及其对应的命令序列

```
alias
```

#### 3.2.3.3 删除别名命令

使用`unalias`命令，后跟要删除的别名，删除指定别名命令

```
unalias ll
```

#### 3.2.3.4 永久保存别名命令

默认情况下，通过`alias`命令创建的别名只在当前会话中有效，一旦退出终端，别名就会失效。如果希望永久保存别名命令，可以将其添加到Shell配置文件中。打开Shell配置文件，添加别名命令的行，保存并关闭文件。例如，在`~/.bashrc`中添加别名`ll`，重新加载Shell配置文件，或者重新启动终端，别名命令就会在每次登录时自动生效

```
alias ll='ls -l'
```

## 3.3 命令基本操作

### 3.3.1 查看命令类型

> `type -a`命令后跟要查询的命令名称，将显示指定命令的类型，包括内置命令、外部命令和别名命令

```
type -a COMMAND
```

### 3.3.2 执行多个命令

1. 使用分号(`;`)来执行多个命令，多条顺序执行，之间没有逻辑关系
   
   ```shell
   command1; command2; command3
   ```

2. 使用`&&`运算符：命令2只有在命令1正确执行（返回状态码为0）时才会执行
   
   ```shell
   command1 && command2
   ```

3. 使用`||`运算符：命令2只有在命令1执行失败（返回状态码非0）时才会执行
   
   ```shell
   command1 || command2
   ```

### 3.3.3 取消命令

> 使用组合键`Ctrl+c`，取消正在执行的命令。按下`Ctrl+c`将发送一个中断信号给当前正在执行的命令，从而终止其执行

### 3.3.4 获取命令帮助

#### 3.3.4.1 man

> `man`是"manual"的缩写，用于查看命令的手册页，使用箭头键进行上下滚动，按`q` 退出手册页

```shell
man ls # 显示`ls`命令的手册页，包含了命令的详细说明、用法、选项和示例等信息
```

#### 3.3.4.2 --help

> `--help`命令用于显示命令的简要帮助信息

```shell
ls --help # 显示`ls`命令的简要帮助信息，包括命令的用法和可用选项
```

#### 3.3.4.3 info

> `info`命令用于查看命令的帮助信息，使用箭头键进行导航，按`q`键退出`info`页面

```shell
info ls # 显示`ls`命令的详细帮助信息，包括命令的说明、用法、选项和示例等
```

# 4 常用命令

sudo命令,它在非root用户下,去调用一些root用户的命令,或者修改一些文件
sudo命令是需要配置的,sudo的配置文件是/etc/sudoers

```
#给bow用户配置sudo权限
[root@bow ~]# vim /etc/sudoers
##
## Allow root to run any commands anywhere 
root ALL=(ALL) ALL
#给bow用户设置sudo命令权限
bow ALL=(ALL) ALL
```

## 4.1 系统命令

### 4.1.1 关机

#### 4.1.1.1 shutdown

> `shutdown`命令用于安全地关闭系统，允许指定关机的时间和方式，以及向用户发送关机通知

```bash
shutdown [OPTIONS...] [TIME] [WALL...]
```

- **OPTIONS**
  
  1. `-h`：关机后停止系统
  
  2. `-r`：关机后重新启动系统
  
  3. `-c`：取消预定的关机

- **TIME**：指定关机的时间
  
  1. `now`：立即关机
  
  2. `+m`：在m分钟后关机
  
  3. `hh:mm`：在指定的小时和分钟关机
  
  4. `+hh:mm`：在指定的小时和分钟之后关机

- **WALL**：可选参数，用于向用户发送关机通知

```shell
1. shutdown -h now
2. shutdown -r 2:30
3. shutdown -h +10 "系统将在10分钟后关机，请保存您的工作"
```

#### 4.1.1.2 其他关机命令

```shell
1. halt 
2. poweroff
3. init 0
4. telinit 0
```

### 4.1.2 重启

```shell
1. reboot
2. sudo shutdown -r now
```

### 4.1.3 系统信息

| 命令                                        | 描述                      |
|:-----------------------------------------:|:-----------------------:|
| <mark>df -h</mark>                        | 显示磁盘使用情况和可用空间           |
| `dmesg`                                   | 显示系统消息                  |
| `fdisk -l`                                | 列出磁盘分区                  |
| `firewall-cmd --zone=public --list-ports` | 列出防火墙中打开的端口             |
| `free -h`                                 | 显示内存使用情况                |
| <mark>lsblk</mark>                        | 列出块设备（如硬盘、USB驱动器和光驱）的信息 |
| `lshw`                                    | 列出硬件信息                  |
| `lsmod`                                   | 列出加载的内核模块               |
| `lsusb`                                   | 列出 USB 设备               |
| `lspci`                                   | 列出 PCI 设备               |
| `mkfs filesystem`                         | 在设备上创建文件系统              |
| `mount <device> <mountpoint>`             | 将设备挂载到指定的挂载点            |
| `quota <username>`                        | 显示用户的磁盘使用情况和限制          |
| `restart firewalld`                       | 重新启动防火墙服务               |
| `roscd package_name`                      | 切换到指定的 ROS 包目录          |
| `roscore`                                 | 启动 ROS 核心               |
| `ros_graph`                               | 显示 ROS 计算图              |
| `roslaunch package_name launch_file`      | 启动 ROS 包中的 launch 文件    |
| `rosparam set parameter value`            | 设置 ROS 参数的值             |
| `rosrun package_name node_name`           | 从指定的 ROS 包中运行一个节点       |
| `rosservice list`                         | 列出可用的 ROS 服务            |
| `rostopic list`                           | 列出可用的 ROS 主题            |
| <mark>runlevel</mark>                     | 显示当前运行级别                |
| `status firewalld`                        | 显示防火墙服务的状态              |
| `status ssh`                              | 显示 SSH 服务的状态            |
| `stop firewalld`                          | 停止防火墙服务                 |
| `ethtool eth0`                            | 显示以太网设备的信息              |
| `firewall-cmd --reload`                   | 重新加载防火墙配置               |
| <mark>uname -a</mark>                     | 显示系统信息                  |

### 4.1.4 网络命令

| 命令                                      | 描述                                                           |
|:---------------------------------------:|:------------------------------------------------------------:|
| curl [options...] <url>                 | 通过指定的URL获取数据。可以使用不同的选项来自定义请求，例如添加请求头、设置请求方法等                 |
| dig <domain>                            | DNS查询工具，获取域名的DNS记录                                           |
| ifconfig / ip addr list                 | 用于查看活动接口的IP地址。它们会显示计算机上所有网络接口的配置信息，包括IP地址、子网掩码、广播地址等         |
| ifdown <interface`                      | 禁用指定的网络接口                                                    |
| ifup <interface>                        | 启用指定的网络接口                                                    |
| ip addr                                 | 用于查看网卡信息，显示与计算机相关的网络接口的详细信息，包括IP地址、MAC地址、状态等                 |
| iptables [options...]                   | 配置Linux防火墙规则                                                 |
| netcat [options...] <host> <port>       | 网络工具，用于创建TCP/UDP连接并进行数据交换                                    |
| netstat [options...]                    | 用于显示网络状态信息。可以使用不同的选项来获取不同类型的网络统计信息，如TCP连接、UDP连接、路由表等         |
| nslookup <domain>                       | 查询域名对应的IP地址                                                  |
| <mark>ping</mark>                       | 用于探测网络上目标主机与当前主机之间的连通性。通过向目标主机发送ICMP回显请求，可以检查是否能够成功与目标主机进行通信 |
| route [options...]                      | 显示和操作IP路由表                                                   |
| scp [options...] <source> <destination> | 通过SSH协议在计算机之间进行文件传输                                          |
| service network restart                 | 用于重启网络服务，重新加载网络配置并重启相关的网络服务，以使新的配置生效                         |
| service network start                   | 用于启动网卡，启动网络服务，并使计算机上的网络接口处于活动状态                              |
| service network stop                    | 用于关闭网卡，停止网络服务，并使计算机上的网络接口处于非活动状态                             |
| ssh <user>@<host>                       | 通过SSH协议远程登录到另一台计算机                                           |
| traceroute [options...] <host>          | 跟踪数据包在网络上的路径                                                 |
| <mark>tty</mark>                        | 显示当前正在使用的终端设备名称                                              |
| <mark>wget [options...] <url></mark>    | 下载文件或网页到本地计算机                                                |

### 4.1.5 服务/进程管理

| 命令                    | 描述                                        |
|:---------------------:|:-----------------------------------------:|
| chkconfig             |                                           |
| jps                   | 显示当前系统中所有的Java进程及其对应的进程ID（PID）            |
| jps -l                | 显示当前系统中所有的Java进程及其对应的进程ID和完整的主类名          |
| jps -m                | 显示当前系统中所有的Java进程及其对应的进程ID和传递给主类的参数        |
| jps -q                | 仅显示当前系统中所有的Java进程的进程ID，不显示进程名。            |
| jps -v                | 显示当前系统中所有的Java进程及其对应的进程ID和传递给主类的参数以及JVM参数 |
| kill <进程号>            | 结束指定的进程                                   |
| kill -9 <进程号>         | 以强制方式结束指定的进程                              |
| killall -9 <进程号>      | 强制终止所有与指定进程名相关的进程                         |
| lsof -i:端口号           | 查看指定端口是否被占用                               |
| ps                    | 显示当前终端下的进程信息                              |
| ps -a                 | 显示所有终端下的进程信息，包括其他用户的进程                    |
| ps -A                 | 显示所有进程的信息，包括系统进程和其他用户的进程                  |
| ps -e                 | 显示所有进程的信息                                 |
| ps -f                 | 显示包含完整格式的进程信息                             |
| ps -p <PID>           | 显示指定PID的进程信息                              |
| pstree                | 显示当前进程及其子进程的树状结构                          |
| pstree -p             | 显示当前进程及其子进程的树状结构，并显示每个进程的PID              |
| pstree -u             | 显示当前进程及其子进程的树状结构，并显示每个进程的用户               |
| service <服务名> start   | 启动指定的服务                                   |
| service <> status     | 查看指定服务的运行状态                               |
| service <> stop       | 停止指定的服务                                   |
| service <> restart    | 重启指定的服务                                   |
| ss -tuln              | 查看系统中所有被占用的端口                             |
| systemctl disable xxx | 取消指定的服务在系统启动时的自动启动                        |
| systemctl enable xxx  | 设置指定的服务在系统启动时自动启动                         |
| systemctl start xxx   |                                           |
| systemctl stop xxx    |                                           |
| sysv-rc-conf          |                                           |
| top                   |                                           |

## 4.2 用户命令

### 4.2.1 创建用户

> `adduser` 命令创建用户，并自动创建用户的家目录，并设置用户组、用户权限等一些默认配置，会提示你设置用户的密码和其他信息

```shell
sudo adduser <username>
```

> `useradd` 命令只会创建用户，不会自动创建用户的家目录和设置默认配置，需要手动设置用户的家目录、用户组、用户权限等信息

```shell
sudo useradd <username>
```

### 4.2.2 删除用户

```shell
1. sudo deluser <username>
2. sudo userdel <username>
```

### 4.2.3 修改用户信息

```shell
1. sudo usermod -l <newusername> <oldusername> # 修改用户名
2. sudo usermod -d 新家目录 <username> # 修改用户的家目录
3. sudo usermod -g
3. sudo usermod -s 新Shell 用户名 # 修改用户的登录Shell
4. sudo passwd 用户名 # 用于修改用户的密码
5. sudo chfn 用户名 # 用于修改用户的详细信息，如姓名、办公室电话
6. sudo chsh -s 新Shell 用户名 # 用于修改用户的登录Shel
```

### 4.2.4 修改用户权限

### 4.2.5 查询用户信息

```shell
1. id <username> # 显示用户的UID和所属的用户组ID
2. w             # 显示当前登录系统的用户信息，包括登录名、终端、登录时间、运行命令等
4. whoami        # 显示当前登录的用户名
5. last          # 显示最近登录系统的用户信息
6. finger <username> # 显示用户的详细信息，包括登录名、真实姓名、终端、登录时间
7. groups <username> # 显示用户所属的用户组
8. cat /etc/passwd   # 查询所有用户信息
9. cut -d: -f1 /etc/passwd # 显示`/etc/passwd`文件中所有用户的用户名
```

### 4.2.6 切换用户

```shell
1. su - <username> #切换用户,但不加载用户的环境变量
2. logout # 注销当前用户
3. exit #退出当前用户的登陆
```

## 4.3 用户组命令

### 4.3.1 创建用户组

```shell
1. sudo addgroup <groupname> # <groupname> 是要创建的用户组名
2. sudo groupadd <groupname> 
```

### 4.3.2 删除用户组

```shell
1. sudo delgroup <groupname> # <groupname> 是要删除的用户组名
2. sudo groupdel <groupname>
```

## 4.4 目录命令

### 4.4.1 查看目录

```shell
1. pwd        # 命令，查看当前目录的路径
2. ls         # 列出当前目录的内容
4. ls -l      # 列出当前目录的详细信息（包括权限、所有者、大小等）
3. ls <path>  # 列出指定目录的内容
```

### 4.3.2 切换目录

```shell
1. cd 目录路径 # 进入指定目录
2. cd ~ # 进入当前用户的主目录（家目录
3. cd .. # 进入上级目录
```

### 4.3.3 创建目录

```shell
1. mkdir <DIRECTORY>               # 创建单个目录
2. mkdir <DIRECTORY1> <DIRECTORY2> # 创建多个目录（同时创建多个目录，可以用空格分隔）
3. mkdir -m 权限 <DIRECTORY>        # 创建目录并设置权限
4. mkdir -p # 自动按需创建父目录
5. mkdir -v verbose # 显示详细过程
```

```shell
mktemp [OPTION]... [TEMPLATE]
-d, --directory` 创建临时目录
-u, --dry-run` do not create anything
~]# mktemp -u file.XXX
注意：mktemp 会将创建的临时文件名直接返回，因此，可直接通过命令引用保存起来
```

### 4.3.4 删除目录

```shell
p 删除某目录后，如果其父目录为空，则一并删除之；
v verbose
rmdir [参数] 目录路径 #删除目录命令，rmdir默认只能删除空目录
rmdir ./dir #删除当前目录下的dir目录
rmdir -p 目录路径 #表示删除目录和它的父目录（目录要是一个空目录）
rmdir -p a/b/c #删除当前目录下的a/b/c目录
```

```shell
rm [参数] 路径 #删除命令
rm 1.txt #删除当前目录下的1.txt文件，删除时会提示，是否删除如果输入y表示删除，输入n表示不删除
rm -f /root/2.txt #-f表示强制删除，不会提示,强制删除/root目录下的2.txt
rm -r a/ #递归的删除当前目录下a目录下的所有内容
[root@bow ~]# rm -r a/
rm：是否进入目录"a/"? y
rm：是否进入目录"a/b"? y
rm：是否进入目录"a/b/c"? y
rm：是否删除普通空文件 "a/b/c/3.txt"？y
rm：是否删除目录 "a/b/c"？y
rm：是否删除普通空文件 "a/b/2.txt"？y
rm：是否删除目录 "a/b"？y
rm：是否删除普通空文件 "a/1.txt"？y
rm：是否删除目录 "a/"？y
rm -rf a/ #强制删除当前目录下a目录及a目录下的所有内容

rm -rf * #删除当前目录下的所有内容
rm -rf a/* #删除当前目录下a目录下的所有内容
rm -rf *.txt #删除当前目录下的所有txt文件
rm -rf *s* #删除当前目录下所有名字中包含s的文件或文件夹
```

### 4.3.5 移动目录

cp`命令：用于复制目录及其内容。以下是一些常见的用法：

- 复制目录及其内容到指定目录：
  
  ```shell
  cp -r 源目录 目标目录
  ```

- 复制目录及其内容并保持权限：
  
  ```shell
  cp -rp 源目录 目标目录
  ```

`mv`命令：用于移动或重命名目录。以下是一些常见的用法：

- 移动目录到指定位置：
  
  ```shell
  mv 源目录 目标目录
  ```

- 重命名目录：
  
  ```shell
  mv 旧目录名 新目录名
  ```

- 查看当目录下各个文件/文件夹空间占用：`du -sh *`

### 压缩目录

- 压缩：`zip xxx.tar xxx` 或 `tar -cf xxx.tar xxx` 或 `tar -czf xxx.tar.gz xxx`

- 解压缩：`unzip xxx.tar` 或 `tar -xf xxx.tar` 或 `tar -xzf xxx.tar.gz`

压缩命令

安装zip和unzip命令:

```
yum -y install zip unzip
```

zip压缩命令

zip 压缩文件名 要压缩的文件路径

```
zip 2.zip 2.txt #将2.txt压缩到2.zip中

zip data.zip data #只会压缩文件夹,不会压缩文件夹下的内容

zip da.zip da/* #压缩文件夹和文件夹内的文件(压缩文件夹和它的下一级文件) 

zip -r data.zip date #-r表示递归地将文件夹及它的子目录文件全部压缩
```

unzip解压命令

unzip 压缩文件路径

```
unzip 2.zip #将2.zip压缩包解压到当前目录下
unzip -l 压缩文件名 #不解压文件,查看压缩包内的文件
unzip -l da.zip #查看da.zip压缩文件中包含的文件
unzip da.zip -d 目标目录 #将压缩文件解压到指定目录 
unzip da.zip -d tm/ #将压缩文件da.zip解压到tm目录下
```

tar命令,用来压缩和解压缩.tar和.tar.gz包

压缩.tar包:

```
tar cvf 压缩文件名 要压缩的文件或目录
tar cvf 2.tar 2.txt #将2.txt压缩为2.tar包
tar cvf data.tar data #将data目录夸张到data.tar包中
```

解压.tar包:

tar xvf 压缩文件名 [-C 指定解压目录]

```
tar xvf 2.tar #将2.tar解压到当前目录
tar xvf 2.tar -C a/ #将2.tar解压到a目录
tar xvf data.tar #解压data.tar到当前目录
```

压缩.tar.gz包:

```
tar zcvf 压缩文件名 要压缩的文件
tar zcvf tm.tar.gz tm #将当前目录下的tm目录压缩为tm.tar.gz
```

解压.tar.gz包:

```
tar zxvf 压缩文件名
tar zxvf tm.tar.gz #将tm.tar.gz解压到当前目录
gzip命令,将文件压缩为.gz包(可以用来压缩.tar文件)
gzip 要压缩的文件 
gzip 2.txt #将2.txt压缩为2.txt.gz
gzip data.tar #将data.tar压缩为data.tar.gz
```

### 4.3.5 创建软连接

## 4.4 文件命令

### 查看文件内容

```bash
cat [options...] <filename>
-a 显示所有内容
-n 同时显示行号
-n, --number : 显示行号
-E, --show-ends : 显示行结束符
-v， --show-nonpriting : 显示非打印字符
-e = -vE : 显示行结束符及非打印字符
-s, --squeeze-blank suppress repeated empty output lines

tail [options...] <filename> # 查看文件最末尾的内容
-n 显示最后几行的内容
-f 显示最后10行内容，并且有新内容增加的时候，自动显示新增的内容
ls #表示查看当前目录下的文件
ls -l #表示查看当前目录下的详细信息
ls -a #表示查看当前目录下的所有文件(包含隐藏文件)
ls -la #表示查看当前目录下的所有文件（包含隐藏文件）的详细信息
ls -lh #h是以适当的单位来显示文件的大小 ls -lh表示查看当前目录下的文件的详细信息，并以合适单位显示文件大小 
ls -l / #表示查看根目录"/"下文件的详细信息
ls /etc #表示查看目录/etc下的文件
ls --help #查看命令的帮助文档
--help参数：所有linux上的命令都有，但写法上有如下几种：
 (1)--help
  (2)--h
  (3)-help
  (4)-h
ll命令:它和ls -l命令功能相同，但是不是所有的linux上都默认安装
ls [OPTIONS]... [FILE]...
ls options
  -a : 显示所有文件，包括隐藏文件
  -A :  显示所有文件，除了.和..的所有隐藏文件
  -r,--reverse : 逆序显示
  -R,--recursive : 递归显示
  -i, --inode
  --file-type：/
  -lc ctime
  -lu atime
  -t : 按修改时间先后显示
  -m : 填满宽度的逗号分隔列表条目
  -S : 以文件大小排序显示
 -d, --directory : 查看目录自身属性信息，结合使用l选项，-ld
  -h, --human-readable : 人为可读的格式显示，换算后的结果会丢失精度
  -l，--long : 长格式列表，即显示文件的详细属性信息
ls -l
`-l` 参数可以查看访问权限，后面可以添加路径，默认当前路径
第 1 列表示访问权限，其中：
第 1 位表示文件类型，`d` 表示目录，`-` 表示文件，`l` 表示文件链接；
后面 9 位分为 3 组，每 3 位为 1 组，第 1 组表示文件所有者权限（`u`），第 2 组表示所属组权限（`g`），第 3 组表示其他用户权限（`o`）；
`r` （read）表示读权限，`w`（write）表示写权限，`x`（execute）表示执行权限
查看当前目录下文件：`ls`（追加 `-a` 参数可以查看隐藏文件）
查看当前目录下文件详细信息：`ll` 或 `ls -l`
全局查找某个文件：`sudo find / -name xxx`
查看文件内容类型 `file [FILE]...` ASCII, ELF
tac [-n] [-E]
more/less
head 查看文件的前n行
n # 或者 -#
tail 查看文件的后n行
n #  或者 -#
-f 查看文件尾部内容结束后不退出，跟随显示新增的行
more 文件路径 #分页查看文件内容
more linux常用命令.txt #分页查看当前目录下linux常用命令.txt文件的内容
#按空格或回车，会继续加载文件内容，按q退出查看
#当加载到文件末尾时，会自动退出查看
less 文件路径 #分页查看文件内容
less linux常用命令.txt #分页查看文件内容，按空格继续加载文件，按q退出查看，不会自动退出查看
head [参数] 文件路径 #从文件开始查看文件
head linux常用命令.txt #查看文件的前10行内容
head -n 文件路径 # n是一个正整数，表示查看文件的前n行数据
head -20 linux常用命令.txt #查看文件的前20行内容
tail [参数] 文件路径 #从文件的末尾查看文件内容
tail linux常用命令.txt #查看文件的后10行内容
tail -n 文件路径 # n是一个正整数，表示查看文件的后n行数据
tail -15 linux常用命令.txt #查看文件后15行内容
tail -f 文件路径 #动态的查看文件的最后几行内容(查看文件时，等待文件更新，如果文件更新了，会显示出新的内容)
tail -f 1.txt #查看文件1.txt的最新内容，tail -f 一般用来查看日志文件 按CTRL+C或才CTRL+Z退出查看
CTRL+C：表示暂停进程
CTRL+Z: 表示停止进程
```

### 移动文件

移动文件：`sudo mv <原文件路径> <目标文件路径>`

### 修改文件名称

mv

```sh
mv [OPTION]... SOURCE... DIRECTORY
mv [OPTION]... -t DIRECTORY SOURCE...
文件重命名：`sudo mv <原文件名> <新文件名>`
options
  -i : interactive
  -f : force

```sh
rm

options
 -i interactive
 -f force
 -r recursive

注意：所有不用的文件建议不要直接删除，而是移动至某个专用目录；（模拟回收站
```

mv 移动命令,它可以移动文件,也可以给文件改名

mv 原文件路径 目标文件路径 #将文件从一个地方拷贝到另一个地方

```
mv 1.txt 12.txt #将文件1.txt改名为12.txt
mv tmp tmp #将tmp目录改名为tm
mv 12.txt tm #将文件12.txt移动到tm目录下
```

### 复制文件

- 拷贝文件：`sudo cp <file-path> <target-path>`
- 拷贝文件夹：`sudo cp -r <file-path> <target-path>`

cp

- 单源复制：`cp [OPTION]... [-T] SOURCE DEST`
- 多源复制：`cp [OPTION]... SOURCE... DIRECTORY`

`# cp [OPTION]... -t DIRECTORY SOURCE...`

- 源复制：`cp [OPTION]... [-T] SOURCE DEST`
  
  - 如果 **DEST** 不存在：则事先创建此文件，并复制源文件的数据流至 **DEST** 中；
  - 如果 **DEST** 存在：
    - 如果 **DEST** 是非目录文件：则覆盖目标文件；
    - 如果 **DEST** 是目录文件：则先在 **DEST** 目录下创建一个与源文件同名的文件，并复制其数据流；

- 多源复制：`cp [OPTION]... SOURCE... DIRECTORY
  
  - cp [OPTION]... -t DIRECTORY SOURCE...`
  - 如果 **DEST** 不存在：错误；
  - 如果 **DEST** 存在：
    - 如果 **DEST** 是非目录文件：错误；
    - 如果 **DEST** 是目录文件：分别复制每个文件至目标目录中，并保持原名；

```sh
-i interactive : 覆盖文件时提醒信息
-f force : 强制覆盖目标文件
-r, -R recursive : 递归复制目录
-d : 复制符号链接文件本身，而非其指向的源文件；--preserver=links
-p： 连同档案的属性一起复制过去，而非使用默认属性
-a -pdR --preserve=all, archive 用于实现归档
  --preserve=
    mode：权限
    ownership：属主和属组
    timestamps: 时间戳
    context：安全标签
    xattr：扩展属性
    links：符号链接
    all：上述所有属性
```

### 创建文件

touch
-c : 指定的文件路径不存在时不予创建
-a : 修改access time
-m : 修改modify time
-t [[CC]YY]MMDDhhmm[.ss]
touch 1.txt #在当前目录下创建一个1.txt文件
touch /root/2.txt #在/root目录下创建一个2.txt文件

### 删除文件

### 修改文件所有者

chown

`chown`：更改文件或目录的所有者

chown 命令,它是更改文件所属用户

```
chown -R 用户[:用户组] 目录或文件
-rwxrw-rw-. 1 root root 31 3月 24 07:46 2.txt
chown bow 2.txt #将2.txt的所属用户改为bow
-rwxrw-rw-. 1 bow root 31 3月 24 07:46 2.txt
chown bow:bows 2.txt #将2.txt所属的用户改为bow,用户组改为bows
-rwxrw-rw-. 1 bow bows 31 3月 24 07:46 2.txt
drwxr--r--. 4 root root 81 3月 24 08:06 data
chown -R bow:bows data #将data目录及它子目录文件的所属用户改为bow,用户组改为bows
drwxr--r--. 4 bow bows 81 3月 24 08:06 data
```

### 修改文件所在组

`chgrp`：更改文件或目录的所属组

### stat

> 显示文件系统状态的文件

- 文件
  
  - 元数据：metadata，inode
  - 数据: data, block

- 时间戳：
  
  - access time
  - modify time(数据)
  - change time(元数据)

wc

wc 命令,word count的缩写,它是查看文件的单词个数

wc [参数] 文件

```
wc -l linux常用命令.txt #-l表示line行数 计算文件的行数
wc -w linux常用命令.txt #-w表示word单词个数 计算文件的单词个数
```

### 查找文件

```
find 名【通配符完全匹配】
grep 名【正则表达式的包含匹配】
which命令名【命令位置】
```

> 在文件系统上查找符合条件的文件

#### find 命令

> 实时查找工具，通过遍历指定起始路径下的文件系统层级结构完成文件查找

```
find [OPTIONS] [查找起始路径] [查找条件] [处理动作]
```

- 查找起始路径：
  
  - 指定具体搜索目标起始路径；
  - 默认为当前目录；

- 查找条件：
  
  - 可以根据文件名、大小、类型、从属关系、权限等等标准进行；
  - 默认为找出指定路径下的所有文件；

- 处理动作：
  
  - 对符合查找条件的文件做出的操作，例如删除等操作；
  - 默认为输出至标准输出；

- 查找条件：
  
  - 表达式：选项和测试
  
  - 选项：
    
    - -maxdepth levels
    - -mindepth levels
  
  - 测试：结果通常为布尔型（"true"， "false"）
  
  - 根据文件名查找
    
    - `-name "pattern"`
    - `-iname "pattern"` 支持glob风格通配符
    - `*,?[],[^]`
    - `-regex "pattern"` 基于正则表达式查找文件，匹配是整个路径
  
  - 根据文件从属关系查找
    
    - `-user` USERNAME 查找属主指定用户的所有文件
    - `-group` GROUPNAME 查找属组指定用户的所有文件
    - `-uid UID` 查找属主指定的UID的所有文件
    - `-gid GID` 查找属组指定的UID的所有文件
    - `-nouser` 查找没有属主的所有文件
    - `-nogroup` 查找没有属组的所有文件
  
  - 根据文件的类型查找
    
    - `-type {f|d|l|b|c|s|p}`
  
  - 根据文件的节点查找
    
    - `-inum NUMBER`
  
  - 组合测试：
    
    - `与：-a`, 默认组合逻辑
    
    - `或：-o`
    
    - `非：-not, ！`
    
    - 或条件表达式时，必须加上括号`\(....\)`
    
    - `find /tmp \( -nouser -o -uid 1003 \) -ls`
    
    - `find /tmp -not \( -user root -o -iname "*fstab*" \) -ls`
    
    - `!A -a !B = !(A -o B) -o并(或)`
    
    - `!A -o !B = !(A -a B) -a交(且)`
    
    - `# find /tmp ! -user root -a ! -name "*fsta*" -a -type f -ls`
    
    - `# find /tmp ! \( -user root -o -name "*fsta*" \) -a -type f -ls`
  
  - 根据文件的大小查找： `-size[+|-]#UNIT`, 常用单位：c,k,M,G
    
    - `#UNIT：-1 < ? <= #`
    - `-#UNIT: 0 < ? <= #-1`
    - `+#UNIT: # < ?`
    - `find /tmp -size 115k`
  
  - 根据时间戳查找：
    
    - 以”天“为单位：`-atime [+|-]#`
    
    - #：[#, #-1], 大于#-1天至#天
    
    - -#：[#, 0] , #天内
    
    - +#：[oo, #-1], #-1天之前所有
    
    - -mtime
    
    - -ctime
    
    - 以“分钟”为单位：-amin, -mmin, -cmin
  
  - 根据权限查找：
    
    - `-perm mode`：精确查找
    
    - `-perm /mode`：任何一类用户(u, g, o)的权限中的任意一位(r,w,x)符合条件即满足；9位权限之间存在"或"关系(包含权限)
      
      - /002 其他用于有写权限的文件
      - /222 ugo权限当中至少有写权限满足（至少有一个有=任何一位有）
    
    - `-perm -mode`：每一类用户(u,g,o)的权限中的每一位(r,w,x)同时符合条件即满足；9位权限之间存在"与"关系(包含权限)
    
    - -222 ugo权限当中所有类必须都有写权限满足
    
    - -not -222 ugo权限当中至少有一类没有
    
    - `-perm +mode`
  
  - 精确查找：`find ./ -perm 644 -ls`

- 处理动作：
  
  - `-print`：输出至标准输出，默认的动作；
  - `-ls`：输出文件的详细信息，类似于"ls -l"命令
  - `-delete`：删除查找到的文件
  - `-fls /PATH/TO/SOMEFILE`：长格式保存在指定文件
  - `-ok COMMAND {} \;`
    - 对查找到的每个文件执行由COMMAND表示的命令
    - 每次操作显示提示确认执行命令
  - `-exec COMMAND {} \；`对查找到的每个文件执行由COMMAND表示的命令
  - `{}`：查找到的文件的占位符
  - `# find ./ -perm /002 -exec mv {} {}.danger \;`

- 注意：find传递查找到的文件路径至后面的命令时，实现查找出所有符合条件的文件格式，并一次性传递给后面的命令；

- 但是有些命令不能接受过长的参数，此时命令执行会失败，另一种方式可规避此问题；

- `find | xargs COMMAND`

```sh
查找/var目录下属主为root，且属组为mail的所有文件或目录；
# find /var -user root -a -group mail -ls

查找/usr目录下不属于root, bin或hadoop的所有文件或目录；用两种方法；
# find /usr -not -user root -a -not -user bin -a -not -user hadoop
# find /usr -not \( -user root -o -user bin -o -user hadoop \) -ls

查找/etc目录下最近一周内其内容修改过，且属主不是root用户也不是hadoop用户的文件或目录；
# find /etc -mtime -7 -a -not \( -user root -o -user hadoop \) -ls
# find /etc -mtime -7 -a -not -user root -a -not -user hadoop -ls

查找当前系统上没有属或属组，且最近一周内曾被访问过的文件或目录；
# find  /  \( -nouser -o -nogroup \)  -atime  -7  -ls

 查找/etc目录下大于1M且类型为普通文件的所有文件；
# find /etc -size +1M -type f -exec ls -lh {} \;

查找/etc目录下所有用户都没有写权限的文件；
# find /etc -not -perm /222 -type f -ls

查找/etc目录至少有一类用户没有执行权限的文件；
# find /etc -not -perm -111 -type f -ls

查找/etc/init.d/目录下，所有用户都有执行权限，且其它用户有写权限的所有文件；
# find /etc -perm -113 -type f -ls
```

### 根据文件名查找

- `-name "pattern"`
- `-iname "pattern"`
- 支持glob风格通配符 `*,?[],[^]`
- `-regex "pattern"` 基于正则表达式查找文件，匹配是整个路径

### 根据文件从属关系查找

- `-user` USERNAME 查找属主指定用户的所有文件
- `-group` GROUPNAME 查找属组指定用户的所有文件
- `-uid UID` 查找属主指定的UID的所有文件
- `-gid GID` 查找属组指定的UID的所有文件
- `-nouser` 查找没有属主的所有文件
- `-nogroup` 查找没有属组的所有文件

### 根据文件的类型查找：

`-type {f|d|l|b|c|s|p}`

### 根据文件的节点查找：

`-inum NUMBER`

### 组合测试

- 与：`-a`, 默认组合逻辑

- 或：`-o`

- 非：`-not, ！`

- 或条件表达式时，必须加上括号`\(....\)`

```sh
~]# find /tmp \( -nouser -o -uid 1003 \) -ls
~]# find /tmp -not \( -user root -o -iname "*fstab*" \) -ls
```

```sh
!A -a !B = !(A -o B) -o并(或)
!A -o !B = !(A -a B) -a交(且)
```

```sh
~]# find /tmp ! -user root -a ! -name "*fsta*" -a -type f -ls
~]# find /tmp ! \( -user root -o -name "*fsta*" \) -a -type f -ls
```

### 根据文件的大小查找

`-size[+|-]#UNIT`

- 常用单位：c,k,M,G
  - `#UNIT：-1 < ? <= #`
  - `-#UNIT: 0 < ? <= #-1`
  - `+#UNIT: # < ?`

`~]#find /tmp -size 115k`

### 根据时间戳查找

- 以”天“为单位：
  
  - `-atime [+|-]#`
  
  - `#：[#, #-1]`, 大于#-1天至#天
  
  - `-#：[#, 0]`, #天内
  
  - `+#：[oo, #-1]`, #-1天之前所有
  
  - `-mtime`
  
  - `-ctime`

- 以“分钟”为单位：
  
  - `-amin`
  - `-mmin`
  - `-cmin`

### 根据权限查找

`-perm mode` 精确查找

`-perm /mode` 任何一类用户(u, g, o)的权限中的任意一位(r,w,x)符合条件即满足；9位权限之间存在"或"关系(包含权限)

/002 其他用于有写权限的文件

/222 ugo权限当中至少有写权限满足（至少有一个有=任何一位有）

`-perm -mode` 每一类用户(u,g,o)的权限中的每一位(r,w,x)同时符合条件即满足；9位权限之间存在"与"关系(包含权限)

-222 ugo权限当中所有类必须都有写权限满足

-not -222 ugo权限当中至少有一类没有

`-perm +mode`

精确查找：`find ./ -perm 644 -ls`

Linux文件操作：

```
创建空文件：touch 文件名
删除文件：rm -rf 文件名
查看文件：cat 文件名【显示全部内容，文件太大无法全部显示】
    cat -n 文件名 【查看内容，并添加行号】
    more 文件名【分屏显示文件内容（百分比），空格下翻页；b上翻页；q退出】
    head 文件名【显示头部，默认10行】
    head -n 行数 文件名 
    tail 文件名
    tail -n 行数 文件名
```

## 4.5 输出 / 输入命令

从标准输入获取内容创建和执行命令

```sh
xargs -n 数字

一行显示列数(空白字符分割)
```

```sh
# echo 1 2 3 4 5 > file.txt
# xargs -n 4 < file.txt
```

### 回显

```sh
echo [SHORT-OPTION]... [STRING]...

echo OPTIONS

-n : do not output the trailing newline
-e : 转义符生效
  \r: 回车符
  \n： 换行符
  \t：制表符
  \v：纵向制表符
  \b：退格符
  \\：反斜线`
  `\033[31m  \033[0m


第一个 `#`
1 加粗
3 前景
4 背景色
5 闪烁
7 前景背景互换


第二个 `#`：颜色，1-7

`\033[0m` 控制结束符

# echo -e "\033[31;1;42mHello\033[0m"
```

- STRING 可以使用引号，单引号和双引号
  
  - 单引号：**强引用**，变量不会被替换
  - 双引号：**弱引用**，变量引用会替换
  - 反引号：**命令解析**，$(COMMAND)=`COMMAND`

- 注意：变量引用的正规符号 `${变量名}`

### install

- 单源复制 : `install [OPTION]... [-t] SOURCE DEST`

- 多源复制
  
  - `install [OPTION]... SOURCE... DIRECTORY`
  - `install [OPTION]... -t DIRECTORY SOURCE...`

- 创建空目录：`install [OPTION]... -d DIRECTORY...`

```sh
-m, --mode=MODE : 设定目标文件权限，默认为 755
-o, --owner=OWNER : 设定文件的属主
-g, --group=GROUP : 设定文件的属组
-d : 创建空目录
-t : 指定目标目录
```

**注意**：`install` 命令不能复制目录

```sh
# install -d demo demo3
# install -m 770 -o redhat -g linux -t /tmp/etc/ /etc/*
```

---

```sh
dd 命令 - convert and copy a file

# dd if=/PATH/FROM/SRC of=/PATH/TO/DEST bs=block size count=数量`

bs=#，block size，复制单元大小，单位：byte

磁盘对考：`~]# dd if=/dev/sda of=/dev/sdb`
备份 MBR: `~]# dd if=/dev/sda of=/tmp/mbr.bak bs=512 count=1`
清除分区：`~]# dd if=/dev/zero of=/dev/sda bs=512 count=1`
破坏MBR中的Bootloader：`~]# dd if=/dev/zero of=/dev/sda bs=256 count=1`
```

- 两个特殊设备
  - `/dev/null` 数据黑洞
  - `/dev/zero` 吐零机

sudo super user do

su - username

wsl --shutdown

wsl -l -v

wsl --list

Shell操作

- find /data -type f -name "*.txt" | xargs sed -i 's/oldgirl/oldboy/g'
- mkdir -p /data/oldboy && echo
- 算术运算
  - $[]
  - $(())
  - $(expr a + b) 或者 `expr a \* b` :注意运算符两边要空格，且乘法符号要转义
- 查看http的并发请求数与其TCP连接状态
  - netstat -tan | awk '/^tcp>/{split($5,ip,":");count[ip[1]]++}END{for(i in count) print i,count[i]}'
- awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr -k1 | head -n 10
- cat /dev/urandom | head -1 | md5sum | haed -c 5
- watch -n 1 "/sbin/ifconfig eth0 | grep bytes"
- find /opt -size +15k -exec mv {} /tmp/ ;
- sed和awk
  - 如果文件是格式化的，即由分隔符分为多个域的，优先使用awk
  - awk适合按列（域）操作，sed适合按行操作
  - awk适合对文件的抽取整理，sed适合对文件的编辑。
- 写一个脚本，该脚本能对标准的apache日志进行分析并统计出总的访问次数和每个访问ip的访问次数，按访问次数列出前5名？
- 显示/test下所有目录
  - ls -d */
  - find . -type d -maxdepth 1
  - ls -F | grep '/$'
  - ls -l | grep '^d' | awk '{print $9}'
- 将文件/etc/a 下中除了 b文件外的所有文件压缩打包放到/home/a下，名字为a.tar.gz
  - tar -exclude /etc/a/b -Pcvfz /home/a/a.tar.gz /etc/a
- 如何查看某进程打开的所有文件
  - lsof -p `ps -ef | grep crond | awk '{print $2}'`
- 获取网卡eth0的80端口的数据包信息，找出访问最高的
  - tcpdump -i eth0 -tnn dst port 80 -c 1000 | awk -F "." '{print 1"."2"."3"."4"."}'|sort|uniq -c|sort -nr|head -5
- 查看/var/log目录下的文件数
  - ls /var/log -1R | grep "-" | wc -l
- 查看Linux系统每个IP的连接数
  - netstat -n | awk '/^tcp/{print 5}' | awk -F":" ’{print 1}' | sort | uniq -c | sort -rn
- 用iptables控制来自192.168.1.2主机的80端口请求
  - iptables -A INPUT -p tcp -s 192.168.1.2 -dport 80 -j ACCEPT
- Linux如何挂载Windows下的共享目录
  - mount .cifs //IP地址/server /mnt/server -o user=username,password=123
- 生成32位随机密码
  - cat /dev/urandom | head -1 | md5sum | head -c 32
- 密码加密
  - echo abc | openssl md5
  - echo abc | openssl base64
  - echo abc | openssl sha
- ps aux 中的VSZ代表什么意思，RSS代表什么意思？
  - VSZ：虚拟内存集，进程占用的虚拟内存空间
  - RSS：物理内存集，进程占用的实际物理内存空间
- 修改内核参数
  - vi /etc/sysctl.conf
  - sysctl -p
- 取0-39随机数
  - expr $[RANDOM%39] + 1 # 注意操作符两边的空格
- 限制apache每秒新建连接数为1，峰值为3
  - iptables -A INPUT -d 172.16.100.1 -p tcp -dport 80 -m limit -limit 1/second -j ACCEPT
- 怎么把脚本添加到系统服务里，即用service来调用？
  - 脚本里添加
    - !/bin/bsh
    - chkconfig: 345 85 15
    - description: httpd
  - chkconfig httpd -add
  - service start httpd
- 按修改时间排序显示目录中的文件
  - ls -lrt /etc
- 打印文件的权限值
  - stat -c %a /etc/inittab
- 查看 ARP 缓存记录的命令是?
  - “arp –a”
- 软件工具的原则
  - 一次做好一件事
  - 处理文本行，不要处理二进制数据
  - 使用正则表达式
  - 默认使用标准输入、输出
  - 避免喋喋不休
  - 输出格式必须与可接受的输出格式一致
  - 让工具去做困难的部分
  - 构建特定工具前，先想想
- 获取密码
  - printf "Enter new password:"
  - stty -echo
  - read pass < /dev/tty
  - printf "Enter again:"
  - read pass2 < /dev/tty
  - stty echo
- 在程序中执行跟踪：
  - set -x：打开跟踪功能
  - set +x：关闭跟踪功能
- 为/home/qiuye目录结构建立一份副本在/home/qy下
  - find /home/qiuye -type -d -print | sed 's;/home/qiuye/;/home/qy/;' | sed 's/^/mkdir /' | sh -x
- sed 's/Tony/Camus/2'：只替换第二次匹配到的
- 单词频率过滤器
  - tr -cs A-Za-z' '\n' | tr A-Z a-z | sort | uniq -c | sort -k1,1nr -k2 | head 25
- tcpdump tcp port 80 -s 0 -w net_stat.pcap

### echo

```
echo Hello World #打印Hello World
echo $PATH #打印环境变量PATH的值,其中$是取变量值的符号，用法：$变量名 或者 ${变量名}

echo -n #打印内容但不换行
echo -n Hello World
```

### >和>>命令

和>>:输出符号，将内容输出到文件中，>表示覆盖(会删除原文件内容) >>表示追加

```
echo Hello World > 1.txt #将Hello World输出到当前目录下的1.txt文件
 #如果当前目录下没有1.txt文件会创建一个新文件，
  #如果当前目录下有1.txt，则会删除原文件内容，写入Hello World
echo 1234 >> 1.txt #将1234追加到当前目录下的1.txt中，如果文件不存在会创建新文件
```

通过>和>>都可以创建文件

### source命令

source 文件路径 #让配置文件修改结果立即生效,(还可以在shell脚本中引用其他的shell脚本)

```
/etc/profile #linux上的系统环境变量配置文件
source /etc/profile #将系统环境变量生效
```

### export命令

```
export 导入全局变量(环境变量)

export 变量名=变量值
export 变量名

变量的赋值:
变量名=变量值
```

<<EOF

<<EOF … EOF:将<<EOF和EOF之间的多行内容传给前面的命令,
其中EOF可以是任意字符串,但约定都使用EOF

```
[root@bow ~]# cat <<EOF
> HELLO
> WORD
> JOB
> SMITH
> EOF
HELLO
WORD
JOB
SMITH
```

<<EOF是shell脚本中使用sqlplus的基础

```
[root@bow ~]# cat <<A
> 11234
> 1234
> 1234
> 1253
> 1253
> A
11234
1234
1234
1253
1253
```

注意:EOF必须顶行写

```
[root@bow ~]# cat <<EOF
> ASDF
> EOF
> ASDFASDF
> EOF
ASDF
 EOF
ASDFASDF
```

### cut命令

cut 截取命令

```
-f 参数,指定列
-d 参数指定列和列之间的分隔符,默认的分隔符是\t(行向制表符)
cut -f 1 1.txt #取1.txt文件中的第1列内容(列分隔符默认为\t)
cut -f 2 1.txt #取1.txt文件中的第2列内容
cut -f 1 -d ',' 3.txt #取3.txt文件中的第1列(列分隔符为,)
cut -f 2 -d ',' 3.txt #取3.txt第2列
```

wc -l linux常用命令.txt | cut -f 1 -d ’ ’ #取文件linux常用命令.txt的行数(分隔符是空格)

```
[root@bow ~]# cut -f 1 -d ',' <<EOF
> A,B,C
> D,E,F
> EOF
A
D
```

### printf命令

```
%ns　　输出字符串，n是数字，指代输出几个字符
%ni　　输出整数。n是数字，指代输出几个数字
%m.nf　　位数和小数位数。例如：%8.2f 代表输出8位数，其中2位是小数，6位是整数
```

printf 格式字符串 内容

```
[root@bow ~]# printf '%s,%s,%s\n' abc def ghj klj klo qer #一行单词第三个打印成一行,单词和单词之间用逗号隔开
abc,def,ghj
klj,klo,qer
[root@bow ~]# printf '%s %s\n' $(cat 4.txt) #将文件4.txt中的一行内容中的单词划分为两个一组打印 cat 合作查看文件内容 $(cat 4.txt)表示取cat命令的执行结果
empno ename
job sal
comm depno
5.txt内容
A B C D E
F G H
[root@bow ~]# printf '%s,%s\n' $(cat 5.txt)
A,B
C,D
E,F
G,H
[root@bow ~]# printf '%5.2f\n' 12.1 #%5.2f表示输出一个小数,数的长度是5,其中有两个小数
12.10
[root@bow ~]# printf '%5.2f\n' 121234.116134 #如果输出的值最大长度超出5,那么整数部分不变量,小数部分会按照四舍五入的方法保存两位
121234.12
[root@bow ~]# printf '%i\n' 1234.5678  #%i只取数字的整数部分
-bash: printf: 1234.5678: 无效数字
1234
```

### awk命令

awk 命令字符串 要处理的内容

```
[root@bow ~]# awk '{printf $1 "\n"}' 1.txt #printf 打印 $n 表示取第几列 $1表示取第1列 
Hello
smith
tomcat
```

awk ‘{print $2}’ 1.txt #取1.txt的第2列,print和printf功能相同是打印,比printf多一个换行功能

```
[root@bow ~]# awk '{printf $1 ","}' 1.txt
Hello,smith,tomcat,[root@bow ~]#
[root@bow ~]# awk '{printf $1}' 1.txt
Hellosmithtomcat
[root@bow ~]# awk '{printf $1 "\v"}' 1.txt
Hello
     smith
          tomcat
[root@bow ~]# awk '{printf $1 ","}' 1.txt
Hello,smith,tomcat,
```

### sed命令

sed 参数 命令 要处理的内容

```
-n　　一般sed命令会把所有数据都输出到屏幕。如果加入此选择，则只会把经过sed命令处理的行输出到屏幕。
-e　　允许对输入数据应用多条sed命令编辑
-i　　用sed的修改结果直接修改读取的数据的文件，而不是修改屏幕输出
[root@bow ~]# sed '2p' 1.txt #查询第2行
Hello world
smith 18
smith 18
tomcat etl
[root@bow ~]# sed -n '2p' 1.txt
smith 18
[root@bow ~]# sed -i 's/18/20/g' 1.txt 使用sed命令修改1.txt内容,将1.txt中18替换为20
[root@bow ~]# cat 1.txt
Hello world
smith 20
tomcat etl
a\　　追加，在当前行后添加一行或多行。添加多行时除最后一行外，每行末尾需要用"\"代表数据未完结。
d　　删除，删除指定的
p　　打印，输出指定的行
[root@bow ~]# sed -i '2a !' 1.txt #在第2行后面追加一行 !
[root@bow ~]# cat 1.txt
Hello world
smith 20
!
tomcat etl
[root@bow ~]# sed -i '3d' 1.txt #删除文件的第3行内容
[root@bow ~]# cat 1.txt
Hello world
smith 20
tomcat etl
[root@bow ~]# vim 6.txt
[root@bow ~]# cat 6.txt
abcd/home/bow
if ad
 -e /home/bow
abcd/home/bow
if ad
 -e /home/bow
abcd/home/bow
if ad
 -e /home/bow
#将6.txt文件中的/home/bow修改为/user/bw
#注意:替换时,的符号是根据/来判断 s/原字符串/目标字符串/g 如果原字符串或新的字符串中有/时,需要使用\来转义
# 错误写法:s//home/bow//user/bw/g 正确写法 s/\/home\/bow/\/user\/bw/g
[root@bow ~]# sed -i 's/\/home\/bow/\/user\/bw/g' 6.txt
[root@bow ~]# cat 6.txt
abcd/user/bw
if ad
 -e /user/bw
abcd/user/bw
if ad
 -e /user/bw
abcd/user/bw
if ad
 -e /user/bw
```

注意:linux中字符串的下标是从0开始的

### chkconfig命令

chkconfig命令检查，设置系统的各种服务

```
chkconfig 服务名 on|off #on表示打开服务 off表示关闭服务 通过chkconfig设置的服务是永久生效
centos6及以下版本永久关闭或打开防火墙
chkconfig iptables on #打开防火墙
chkconfig iptables off #永久地关闭防火墙
```

防火墙:

centos7以上:

```
systemctl start firewalld #启动防火墙
systemctl stop firewalld #关闭防火墙(临时关闭)
systemctl status firewalld #查看防火墙状态
systemctl disable firewalld #永久关闭防火墙
systemctl enable firewalld #打开防火墙(不是启动防火墙)
通过firewall-cmd来配置防火墙
```

centos6及以下:

防火墙配置文件:/etc/iptables,这个文件可以详细的配置防火墙,如果没有/etc/iptables文件可以使用iptables save可以生成该文件

iptables 命令配置防火墙

```
service iptables start #centos6及6以下版本,启动防火墙的命令
     service iptables stop #centos6及6以下版本,关闭防火墙(注意,关闭防火墙,只是临时关闭,下次重启之后防火墙依然会启动)
     service iptables restart #重启防火墙
```

环境变量配置文件

/etc/profile是linux系统上配置系统环境变量的一个文件(针对所有用户的配置)
用户根目录下的.bash_profile:是用户环境变量的配置(针对当前用户有效)

```
su - 用户名 #切换用户时,会加载用户根目录下的.bash_profile环境变量配置文件
su 用户名 #不会加载.bash_profile
```

网络配置文件

网卡配置文件目录:/etc/sysconfig/network-scripts

网卡配置文件名都是以ifcfg-开头,其中ifcfg-lo是本地网卡,是不需要配置的

```
vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
#网卡类型
TYPE="Ethernet"
#协议 dhcp表示:ip地址是自动分配的,static表示静态ip(手动配置ip地址),none表示没有协议(也是需要手动配置ip地址)
BOOTPROTO="dhcp"
DEFROUTE="yes"
#网卡名称
NAME="enp0s3"
UUID="deed3fd2-bd67-459b-8a49-ef0dd6e575a2"
DEVICE="enp0s3"
#配置网卡是否随机启动,yes:表示随机启动,no:表示需要手动启动
ONBOOT="yes"
#配置静态ip,BOOTPROTO必须是static或none
#ip地址配置
IPADDR=192.168.1.106
#配置子网掩码
NETMASTER=255.255.255.0
#配置网关
GATEWAY=192.168.1.1
#配置dns:域名解析服务器可以配置多个
DNS1=192.168.1.1
DNS2=192.168.5.1
```

修改完网卡文件之后,*重启网络*即可

### netstat

netstat命令也属于net-tools软件包

```
netstat -tulp | grep 1521 #查看oracle监听器程序是否正常启动
```

文件查找

> 在文件系统上查找符合条件的文件

日期相关命令

> 系统启动时从硬件读取日期和时间信息；读取完成以后，就不在与硬件相关联

- 系统时钟
- 硬件时钟

```sh
date [OPTIONS]... [+FORMAT]

显示系统时钟

+FORMAT:
  %F fulldate
  %Y year
  %m month
  %d day
  %T time
  %H hour
  %M minute
  %S second
  %s unixtimestamp, unix元年 1970-1-1 0:0:0之后经过的秒数
```

`~]# date +"%F %T"`

设定日期时间：`date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]`

```sh
clock 显示硬件时钟

是 hwclcok 的软链接
```

```sh
hwclock - query or set the hardware clock(RTC)

-s, --hctosys : Set the System Time from the Hardware Clock.（硬件时钟是对的）
-w, --systohc : Set the Hardware Clock to the current System Time.(系统时钟是对的)
```

```sh
cal [options] [[[day] month] year]
```

```sh
which OPTIONS

OPTIONS:
  --skip-alias
```

```sh
whereis OPTIONS

OPTIONS:
-b binary path
-m man path
```

```sh
who options

options:
  -b latest system start time
  -r runlevel
  -u show process number
```

```sh
w 

增强版的 who 命令
```

- IDEL

- JCPU 与该tty终端连接的所有进程占用的时间，不包括过去的后台作业时间

- PCPU 当前进程(即w项中显示的)所占用的时间

- `-print` 输出至标准输出，默认的动作；

- `-ls` 输出文件的详细信息，类似于"ls -l"命令

- `-delete：删除查找到的文件

- `-fls /PATH/TO/SOMEFILE` 长格式保存在指定文件

- `-ok COMMAND {} \;`
  
  - 对查找到的每个文件执行由COMMAND表示的命令
  - 每次操作显示提示确认执行命令

- `-exec COMMAND {}` \；对查找到的每个文件执行由COMMAND表示的命令

- `{}` 查找到的文件的占位符

`~]# find ./ -perm /002 -exec mv {} {}.danger \;`

注意：find传递查找到的文件路径至后面的命令时，实现查找出所有符合条件的文件格式，并一次性传递给后面的命令；

但是有些命令不能接受过长的参数，此时命令执行会失败，另一种方式可规避此问题；

`~]# find | xargs COMMAND`

1. 查找 `/var` 目录下属主为 `root`，且属组为 `mail` 的所有文件或目录； `~]# find /var -user root -a -group mail -ls`

2. 查找 `/usr` 目录下不属于 `root`, `bin` 或 `hadoop` 的所有文件或目录；用两种方法； `~]# find /usr -not -user root -a -not -user bin -a -not -user hadoop`

`~]# find /usr -not \( -user root -o -user bin -o -user hadoop \) -ls`

3)查找 `/etc` 目录下最近一周内其内容修改过，且属主不是 `root` 用户也不是 `hadoop` 用户的文件或目录；

```sh
~]# find /etc -mtime -7 -a -not \( -user root -o -user hadoop \) -ls
~]# find /etc -mtime -7 -a -not -user root -a -not -user hadoop -ls
```

4)查找当前系统上没有属或属组，且最近一周内曾被访问过的文件或目录；

```sh
~]# find  /  \( -nouser -o -nogroup \)  -atime  -7  -ls
```

5)查找 `/etc` 目录下大于 `1M` 且类型为普通文件的所有文件；

```sh
~]# find /etc -size +1M -type f -exec ls -lh {} \;
```

6)查找 `/etc` 目录下所有用户都没有写权限的文件；

```sh
~]# find /etc -not -perm /222 -type f -ls
```

7)查找 `/etc` 目录至少有一类用户没有执行权限的文件

```sh
~]# find /etc -not -perm -111 -type f -ls
```

8)查找 `/etc/init.d/`目录下，所有用户都有执行权限，且其它用户有写权限的所有文件； `~]# find /etc -perm -113 -type f -ls`

## grep

```bash
 <command> | grep [options...] <RE>
```

常见的option有

- `-i 忽略大小写` 

locate

- 依赖于事先构建好的索引库

- 系统自动实现（周期性任务）

- 工作特性：
  
  - 查找速度快；
  - 模糊查找；
  - 非实时查找；

- `locate [OPTIONS] ... PATTERN...`
  
  - -b：只匹配路径中的基名
  - -c：仅显示匹配的数量
  - -r, --regexp：BRE

- 手动更新数据库 `~]# updatedb`

- 注意：索引构建过程中需要遍历整个跟文件系统，极消耗资源；

- 依赖于事先构建好的索引库

- 系统自动实现（周期性任务）

#### locate 工作特性

- 查找速度快；

- 模糊查找；

- 非实时查找；

- `# locate [OPTIONS] ... PATTERN...`
  
  - `-b`：只匹配路径中的基名
  - `-c`：仅显示匹配的数量
  - `-r`, `--regexp`：BRE

#### locate 手动更新数据库

`~]# updatedb`

- 注意：索引构建过程中需要遍历整个跟文件系统，极消耗资源；

## 其他

### 时间命令

history

！1

# 5 vim

vim命令总体分为两类

vim 文件路径 --进入非编辑模式

非编辑模式命令：

```
yy：复制光标当前行
p：粘贴
dd:删除光标当前行
$:光标跳到当前行的行尾
^:光标跳到当前行的行首

:s/原字符串/新字符串/:替换光标当前行内容
:%s/原字符串/新字符串/g:全文替换 #g表示global i表示ignore忽略大小写

/要查找的内容:从光标当前行向后查找内容
/d #在文件中查找d字母
?要查找的内容：从光标当前位置向前查找内容
?d #查找文件中的d字母
CTRL+F:向下翻1页
CTRL+B:向上翻1页

:set nu：显示文件的行号
:set nonu: 去掉行号显示
u:撤消

**:set ff :显示文件的格式 #unix表示在unix上的文件 dos表示文件是windows上的文件**
:w ：表示保存文件
:q :表示退出vim命令
:wq:保存并退出
:w!:强制保存
:q!:强制退出但不保存
:wq!:强制保存并退出
i:表示进入编辑模式，并且光标在当前行
o：表示进入编辑模式，并且光标出现的当前行的下一行(新行)
```

编辑模式命令：

编辑模式下可以能过方向键控制光标的位置，并且可以输入文件到光标当前位置

```
 ESC:退出编辑模式
```

## 4.1 使用模式

正常模式 （按Esc或Ctrl+[进入） 左下角显示文件名或为空

插入模式 （按i键进入） 左下角显示–INSERT–

命令行模式

## 4.2 快捷键

esc

插入命令
i 在当前位置生前插入
I 在当前行首插入
a 在当前位置后插入
A 在当前行尾插入
o 在当前行之后插入一行
O 在当前行之前插入一行

移动命令
h 左移一个字符
l 右移一个字符，这个命令很少用，一般用w代替。
k 上移一个字符
j 下移一个字符

以上四个命令可以配合数字使用，比如20j就是向下移动20行，5h就是向左移动5个字符，在Vim中，很多命令都可以配合数字使用，比如删除10个字符10x，在当前位置后插入3个！，3a！，这里的Esc是必须的，否则命令不生效。

w 向前移动一个单词（光标停在单词首部），如果已到行尾，则转至下一行行首。此命令快，可以代替l命令。
b 向后移动一个单词 2b 向后移动2个单词
e，同w，只不过是光标停在单词尾部
ge，同b，光标停在单词尾部。
^ 移动到本行第一个非空白字符上。
0（数字0）移动到本行第一个字符上，
移动到本行第一个字符。同0健。
移动到下面3行的行尾
gg 移动到文件头。 = [[
G（shift + g） 移动到文件尾。 = ]]
f（find）命令也可以用于移动，fx将找到光标后第一个为x的字符，3fd将找到第三个为d的字符。
F 同f，反向查找。

跳到指定行，冒号+行号，回车，比如跳到240行就是 :240回车。另一个方法是行号+G，比如230G跳到230行。

Ctrl + e 向下滚动一行
Ctrl + y 向上滚动一行
Ctrl + d 向下滚动半屏
Ctrl + u 向上滚动半屏
Ctrl + f 向下滚动一屏
Ctrl + b 向上滚动一屏

撤销和重做
u 撤销（Undo）
U 撤销对整行的操作
Ctrl + r 重做（Redo），即撤销的撤销。

删除命令
x 删除当前字符
3x 删除当前光标开始向后三个字符
X 删除当前字符的前一个字符。X=dh
dl 删除当前字符， dl=x
dh 删除前一个字符
dd 删除当前行
dj 删除上一行
dk 删除下一行
10d 删除当前行开始的10行。
D 删除当前字符至行尾。D=d 删除当前字符之后的所有字符（本行）
kdgg 删除当前行之前所有行（不包括当前行）
jdG（jd shift + g） 删除当前行之后所有行（不包括当前行）
:1,10d 删除1-10行
:11,d 删除所有行
J(shift + j)　　删除两行之间的空行，实际上是合并两行。

拷贝和粘贴
yy 拷贝当前行
nyy 拷贝当前后开始的n行，比如2yy拷贝当前行及其下一行。
p 在当前光标后粘贴,如果之前使用了yy命令来复制一行，那么就在当前行的下一行粘贴。
shift+p 在当前行前粘贴
:1,10 co 20 将1-10行插入到第20行之后。
:1, 将整个文件复制一份并添加到文件尾部。

正常模式下按v（逐字）或V（逐行）进入可视模式，然后用jklh命令移动即可选择某些行或字符，再按y即可复制

ddp交换当前行和其下一行
xp交换当前字符和其后一个字符

剪切命令
正常模式下按v（逐字）或V（逐行）进入可视模式，然后用jklh命令移动即可选择某些行或字符，再按d即可剪切

ndd 剪切当前行之后的n行。利用p命令可以对剪切的内容进行粘贴
:1,10d 将1-10行剪切。利用p命令可将剪切后的内容进行粘贴。
:1, 10 m 20 将第1-10行移动到第20行之后。

## 4.3 常用命令

set nu

set nonu

#### 退出命令

:w 保存文件但不退出

:w file 将修改另外保存到file中，不退出

:w! 强制保存，不推出

:wq 保存文件并退出

:wq! 强制保存文件，并退出

:q 不保存文件，退出

:q! 不保存文件，强制退出

:e! 放弃所有修改，从上次保存文件开始再编辑命令历史

#### 文件切换命令

打开单个文件 vim file

同时打开多个文件 vim file1 file2 file3 …

在vim窗口中打开一个新文件 :open file

在新窗口中打开文件 :split file

切换到下一个文件 :bn

切换到上一个文件 :bp

查看当前打开的文件列表，当前正在编辑的文件会用[]括起来。:args

打开远程文件，比如ftp或者share folder

:e ftp://192.168.10.76/abc.txt

:e \qadrive\test\1.txt

#### 查找命令

/text　　查找text 按n健查找下一个 按N健查找前一个

?text　　查找text，反向查找，按n健查找下一个，按N健查找前一个。

#### 转义命令

vim中有一些特殊字符在查找时需要转义　　.*[]^%/?~$

:set ignorecase　　忽略大小写的查找
:set noignorecase　　不忽略大小写的查找

查找很长的词，如果一个词很长，键入麻烦，可以将光标移动到该词上，按*或#键即可以该单词进行搜索，相当于/搜索。而#命令相当于?搜索。

:set hlsearch　　高亮搜索结果，所有结果都高亮显示，而不是只显示一个匹配。
:set nohlsearch　　关闭高亮搜索显示
:nohlsearch　　关闭当前的高亮显示，如果再次搜索或者按下n或N键，则会再次高亮。
:set incsearch　　逐步搜索模式，对当前键入的字符进行搜索而不必等待键入完成。
:set wrapscan　　重新搜索，在搜索到文件头或尾时，返回继续搜索，默认开启。

#### 替换命令

ra 将当前字符替换为a，当期字符即光标所在字符。
s/old/new/ 用old替换new，替换当前行的第一个匹配
s/old/new/g 用old替换new，替换当前行的所有匹配
%s/old/new/ 用old替换new，替换所有行的第一个匹配
%s/old/new/g 用old替换new，替换整个文件的所有匹配
:10,20 s/^/ /g 在第10行知第20行每行前面加四个空格，用于缩进。
ddp 交换光标所在行和其下紧邻的一行。

#### 窗口命令

:split或new 打开一个新窗口，光标停在顶层的窗口上
:split file或:new file 用新窗口打开文件
split打开的窗口都是横向的，使用vsplit可以纵向打开窗口。
Ctrl+ww 移动到下一个窗口
Ctrl+wj 移动到下方的窗口
Ctrl+wk 移动到上方的窗口

关闭窗口 :close 最后一个窗口不能使用此命令，可以防止意外退出vim。
:q 如果是最后一个被关闭的窗口，那么将退出vim。
ZZ 保存并退出。

关闭所有窗口，只保留当前窗口

:only

录制宏
按q键加任意字母开始录制，再按q键结束录制（这意味着vim中的宏不可嵌套），使用的时候@加宏名，比如qa。。。q录制名为a的宏，@a使用这个宏。

执行shell命令
:!command
:!ls 列出当前目录下文件
:!perl -c script.pl 检查perl脚本语法，可以不用退出vim，非常方便。
:!perl script.pl 执行perl脚本，可以不用退出vim，非常方便。
:suspend或Ctrl - Z 挂起vim，回到shell，按fg可以返回vim。

#### 注释命令

perl程序中#开始的行为注释，所以要注释某些行，只需在行首加入#

3,5 s/^/#/g 注释第3-5行
3,5 s/^#//g 解除3-5行的注释
1,$ s/^/#/g 注释整个文档。
:%s/^/#/g 注释整个文档，此法更快。

#### 帮助命令

:help or F1 显示整个帮助
:help xxx 显示xxx的帮助，比如 :help i, :help CTRL-[（即Ctrl+[的帮助）。
:help ‘number’ Vim选项的帮助用单引号括起
:help 特殊键的帮助用<>扩起
:help -t Vim启动参数的帮助用-
：help i_ 插入模式下Esc的帮助，某个模式下的帮助用模式_主题的模式

帮助文件中位于||之间的内容是超链接，可以用Ctrl+]进入链接，Ctrl+o（Ctrl + t）返回

其他非编辑命令
. 重复前一次命令

:set ruler?　　查看是否设置了ruler，在.vimrc中，使用set命令设制的选项都可以通过这个命令查看
:scriptnames　　查看vim脚本文件的位置，比如.vimrc文件，语法文件及plugin等。
:set list 显示非打印字符，如tab，空格，行尾等。如果tab无法显示，请确定用set lcs=tab:>-命令设置了.vimrc文件，并确保你的文件中的确有tab，如果开启了expendtab，那么tab将被扩展为空格。

Vim教程
在Unix系统上

$ vimtutor

在Windows系统上

:help tutor

syntax

:syntax 列出已经定义的语法项
:syntax clear 清除已定义的语法规则
:syntax case match 大小写敏感，int和Int将视为不同的语法元素
:syntax case ignore 大小写无关，int和Int将视为相同的语法元素，并使用同样的配色方案

- 编辑相关
  - awk
    - NF：字段总数
    - NR：第几行数据
    - FS：分隔字符
  - sed
    - -n
    - -i 直接修改
    - 4a：在第四行后添加
    - 4i：在第四行前插入
    - 1,5c sting：用sting替换1到5行的内容
    - s/要被替换的字符串/新的字符串/g
  - sort
    - -t
    - -nr sort \|uniq -c \|sort -nr
  - tr
    - -d：删除
    - \[a-z\] \[A Z\]：替换

# 6 rwx权限

修改权限

`chmod`：更改文件或目录的权限

chmod 赋权限命令

```
chmod 权限 文件路径
-rw-r--r--. 1 root root   31 3月  24 07:46 2.txt
chmod u+x 2.txt #给用户加上执行权限
-rwxr--r--. 1 root root   31 3月  24 07:46 2.txt
chmod g+w 2.txt #给用户组加写权限
-rwxrw-r--. 1 root root   31 3月  24 07:46 2.txt
chmod o+x 2.txt #给其他用户加执行权限
-rwxrw-r-x. 1 root root   31 3月  24 07:46 2.txt
chmod g-w 2.txt #去掉用户的写权限 
-rwxr--r-x. 1 root root   31 3月  24 07:46 2.txt
```

用3个数字来设置文件或目录的权限,第1个数字表示用户权限,第2数字表示用户组权限,第3个数字表示其他用户权限

```
chmod 755 2.txt #设置用户的权限为rwx,用户组的权限r-x,其他用户的权限r-x
-rwxr-xr-x. 1 root root 31 3月 24 07:46 2.txt
chmod 766 2.txt #设置用户权限为rwx,用户组权限rw-,其他用户的权限rw-
-rwxrw-rw-. 1 root root 31 3月 24 07:46 2.txt
```

设置目录权限时,要使用-R参数,保证目录下的所有文件和目录的权限相同

```
drwxr-xr-x. 4 root root 81 3月 24 08:06 data
chmod -R 777 data #将data目录以及它下面的所有文件的权限设置为rwxrwxrwx
drwxrwxrwx. 4 root root 81 3月 24 08:06 data
```

# 7 Linux文件系统

文件是什么？众多文件如何有效组织起来？

linux下所有的绝对路径都是从根目录"/"开始

root:是linux下root用户的根目录

home:是linux下其他用户的默认根目录 （例如：在linux上创建了一个bow用户，那么就会在/home下面生成一个bow目录作为bow用户的根目录）

etc:是linux下系统配置文件目录

tmp:临时文件目录，所有用户都可以用

### 文件

> 存储空间存储的一段流式数据，对数据可以做到按名存取

文件必须有**名字**，**文件名与文件内容没有关系，文件名是文件的外围属性**。
所以**文件是文件名、大小、属性这些外围属性所组成的**

- 文件有两类数据
  - **元数据**：metadata, 文件名，大小属性等
  - **数据**：data

文件索引信息就是元数据，而元数据所指向的就相当于数据。目录也是一种文件，是一种特殊的文件。

### Linux文件类型

- -或f：常规文件；regular file；
- d: directory，目录文件；
- b: block device，块设备文件，支持以“block”为单位进行随机访问
- c：character device，字符设备文件，支持以“character”为单位进行线性访问
  - major number：主设备号，用于标识设备类型，进而确定要加载的驱动程序
  - minor number：次设备号，用于标识同一类型中的不同的设备
  - 8位二进制：0-255
- l：symbolic link，符号链接文件；
- p: pipe，命名管道；
- s: socket，套接字文件；

### 目录

> 是路径映射

### 文件名使用法则

- 文件名严格**区分字符大小写**
- **目录也是文件**，在同一路径下，两个文件不能同名；
- 文件名使用出了 `/` 以外的任意字符，但不建议使用特殊字符
- 文件名长度最长不能超过 **255 个字符**
- 所有 `.` 开头的文件都为隐藏文件
- `.` 当前目录
- `..` 当前目录的上一级目录

### 文件系统

> 层级结构；有索引

- /: 原初起点，根目录

- 第二层结构，子目录

- 第三层结构，子目录

- 倒置树状结构

- `/dev/pts/2`
  
  - 最左侧的`/`：表示**根目录**
  - 其他的`/`：表示**路径分隔符**
    - Linux 的路径分隔符是 `/`
    - Windows 的是 `\`

### 文件的路径表示

- 绝对路径：从根开始表示出的路径
- 相对路径：从当前位置开始表示出的路径
- 绝对路径：如 `/etc/init.d` 
- 当前目录和上层目录：`./ ../` 
- 主目录：`~/` 
- 切换目录：`cd`

### 用户家目录 home

> 用户的起始目录：普通用户管理文件的位置

### 工作目录

```sh
/etc/sysconfig/network-scripts/ifcfg-eno16777736
`basename` 最右侧的文件或目录名
`dirname` basename 左侧的路径
```

`~]# basename /PATH/TO/SOMEFILE` **SOMEFILE**

`~]# dirname /PATH/TO/SOMEFILE` **/PATH/TO**

- 文件是什么？存储空间存储的一段流式数据，对数据可以做到按名存取

## 程序编译方式：Linux: glibc(GNU libc)标准库

- 静态链接编译：运行程序复制一份系统库文件到程序当中并运行
- 动态链接编译：运行程序需要使用的系统库文件单独载入到内存中供多个程序同时使用

## 进程的类型

- 终端：硬件设备，关联一个用户接口
- 与终端相关：通过终端启动，交互式启动
- 与终端无关：操作引导启动过程当中自动启动

## 操作系统的组成

- 文件系统：层次结构（倒置树状结构）；有索引

- 在启动的时候需要使用文件，需要载入内存，有一个分区作为起始分区，这个分区被称为根分区。根是由内核直接访问的。

- 文件系统通过系统调用接口实现用户空间中用户操作

## 操作系统自身运行使用的

```sh
/bin
/sbin
```

## 运行正常功能的程序存放位置

```sh
/usr/bin
/usr/sbin
```

## 用来存放第三方软件的程序

```sh
/usr/local/bin
/usr/local/sbin
```

## Linux目结结构按类别组织的

- 应用程序：`/usr/bin`
- 数据文件：`/usr/share`
- 配置文件：`/etc`
- 启动服务：`/etc/init.d`

## [FHS, Filesytem Hierarchy Standard](http://www.pathname.com/fhs/)

- `/bin` 所有用户可用的命令基本命令程序文件

- `/sbin` 仅系统管理员使用的工具程序

- `/boot` 引导加载器必须用到的各静态文件 kernel,initramfs(initrd), grub等

- `/dev` 存储特殊文件或设备文件

- 设备类型：
  
  - 字符设备(线性设备), character device
    - 键盘，显示器
  - 块设备(随机设备), block device
    - 硬盘

- `/dev/tty[0-63]` 虚拟终端

- `/dev/ttyS[0-3]` 串口，串行终端

- `/dev/console` 物理终端，控制台

- `/dev/md[0-3]` 软raid设备

- `/dev/loop[0-7]` 本地回环设备

- `/dev/ram[0-15]` 内存

- `/dev/lp[0-3]` 并口

- `/dev/null` 无限数据接收设备，所有数据写入这个设备被丢弃
  
  - `~]# cat /dev/null > file` (清空文件内容)

- `/dev/zero` 无限0资源

- `/dev/hd[a-t]` IDE设备

- `/dev/sd[a-z]` SCSI设备

- `/dev/fd[0-7]` 标准软驱

- `/dev/cdrom => /dev/hdc`

- `/dev/fd[0-31]` framebuffer

- `/dev/modem` => /dev/ttyS[0-9]

- `/dev/pilot` => /dev/ttyS[0-9]，引导

- `/dev/random` 随机数设备

- `/dev/uradom` 随机数设备

- `/etc` 系统程序的配置文件，只能为静态文件

- `/etc/redhat-release`Linux发行版版本号

- `/etc/issue` 记录用户登陆前显示的信息

- `/etc/ld.so.conf` 额外的目录列表搜索共享库

- `/etc/fstab` 开机自动挂载磁盘文件

- `/etc/hosts` 主机名配置文件，设定IP与域名的对应解析表，相当于本地LAN的DNS

- `/etc/resolv.conf` 本地客户端DNS文件，实现域名和IP的互相解析

- `/etc/rc.local` 开机自动运行程序命令的文件

- `/etc/inittab` 初始化系统配置文件

- `/etc/networks` 静态信息网络名称文件

- `/etc/passwd` 用户信息文件

- `/etc/profile` 系统全局环境变量配置目录

- `/etc/profiel.d/` 系统全局环境变量目录

- `/etc/group` 用户组名相关信息

- `/etc/shadow` 密码信息文件

- `/etc/modprobe.conf`内核模块额外参数设定

- `/etc/protocols` 系统支持的协议文件

- `/etc/exports` NFS文件系统访问控制列表

- `/etc/motd` 系统登录显示消息文件

- `/etc/services` 网络服务端口名称

- `/etc/syslog.conf` syslogd配置文件

- `/etc/init.d/` 服务启动命令目录

- `/etc/xinit.d` 服务器通过xinetd模式运行目录

- `/etc/sysconfig` 系统级别的应用

- `/etc/sysconfig/network-scripts/ifcfg-eth0` 网卡配置文件
  
  - DEVICE eth0 设备名(0:第一个设备)
  
  - HWADDR=00:0C:29:BA:8C:8F 网卡 MAC地址
  
  - TYPE Ethernet 以太网
  
  - UUID=xxxxx
  
  - ONBOOT=yes 开机自动启动
  
  - BOOTPROTO={none|dhcp|static}
  
  - IPADDR=10.0.0.7
  
  - NETMAST=255.255.255.0 子网掩码（区分网络位和主机位）
  
  - DNS1=8.8.8.8
  
  - DNS2=202.106.0.20 域名解析服务器
  
  - GATEWAY=10.0.0.254 网关地址，路由器的地址
  
  - USERCTL=no
  
  - 修改IP为静态
    
    - `# cp ifcfg-etho{,.ori}`
    - `# vim ifcfg-eth0`
    - `# /etc/init.d/network restart` 影响所有网卡
    - `# netstat -an | grep 192`
    - `ifdown eth0 && ifup eht0`
  
  - `# route -n` 查看路由

- `# ifdown eth0 && ifup eth0`

- `/etc/sysct.conf` 内核参数里配置永久生效

- `/etc/sudoers` 执行使用sudo命令的配置文件

- `/etc/securetty` 设定哪些终端可以让root登录

- `/etc/rsyslog.conf` 日志设置文件

- `/etc/login.defs` 所有用户登录时的缺省配制

- `/etc/DIR_COLORS` 设定颜色

- `/etc/host.conf` 用户的系统如何查询节点名

- `/etc/hosts.allow` 设置允许使用inetd的机器使用

- `/etc/hosts.deny` 设置不允许使用inetd的机器使用

- `/etc/X11` X Window的配置文件

- `/etc/opt` /opt配置文件目录

- `/etc/x11` x Windows system(opitonal)

- `/etc/sgml` 配置SGML(opitonal)

- `/etc/xml` 配置XML(opitonal)

- `/home` 普通用户的家目录的集中位置；每个普通用户的家目录默认为此目录下与用户名同名的子目录，`/home/UERNAME`

- `/root`系统管理员家目录

- `/lib` 32bits库文件目录，为系统启动或根文件系统上的应用程序(`/bin,/sbin`等) 提供共享库，以及为内核提供内核模块

- `/lib/libc.so.*` 动态链接的C库

- `/lib/ld*` 运行时链接器/加载器

- `/lib/Modules/` 可装载内核模块的目录

- `/lib64` 64bit系统特有的存放64位共享库的路径

- `/lost-found` 丢失数据目录

- `/mnt` 其他文件系统的临时挂载点；

- `/media` 移动媒体挂载点

- `/opt` 附加应用程序的安装位置；用于有些软件包数据安装目录

- `/srv` 当前主机为服务提供的数据， swift服务

- `/tmp` 为那些会产生临时文件的程序提供的用于存储临时文件的目录；
  
  - 可供所用户执行写入操作；
  - 有特殊权限；

- `/usr` Universal Shareable, read-only data 全局共享的只读数据目录；

- `/usr/bin` 用户可执行文件目录

- `/usr/sbin` 管理员可执行文件目录

- `/usr/include` 程序的头文件存放目录

- `/usr/lib` 32bits库文件存放目录

- `/usr/lib64` 64bits库文件存放目录

- `/usr/share` 与体系结构无关的数据，架构特有的文件的存储位置

- `/usr/share/fonts` 字体目录

- `/usr/share/man` 命令手册帮助目录

- `/usr/share/doc` 程序自带文档

- `/usr/src` 源码目录

- `/usr/X11R6` X-Window程序的安装位置

- `/usr/local` 系统管理员安装本地应用程序，通常是第三方应用程序安装目录

- `/usr/local/bin` 应用程序目录

- `/usr/local/sbin` 管理员程序目录

- `/usr/local/lib` 32bits库目录

- `/usr/local/lib64` 64bits库目录

- `/usr/local/include` 头文件

- `/var` 经常变化的目录

- `/var/cache/` 应用缓存数据目录

- `/var/tmp` 临时文件保留与系统重启目录

- `/var/lib/` 变量状态信息

- `/var/local/` /usr/local变量数据目录

- `/var/lock/` 锁文件目录

- `/var/log/` 日志文件目录

- `/var/log/messages` 系统日志文件，按周自动轮询

- `/var/log/lastlog` 记录每个用户登录系统信息
  
  - 只能使用 `lastlog` 命令，不能打开文件

- `lastb` 记录所有用户登录时间

- `/var/log/secure` 系统安全信息日记(端口,pop3,ssh,telnet,ftp

- `/var/log/wtmp` 记录所有登录和注销信息

- `# last -f /var/log/wtmp`

- `/var/opt` 变量数据/opt

- `/var/run` 运行进程相关数据，运行时变量数据

- `/var/spool` 应用轴数据目录

- `/var/spool/mail` 邮件目录

- `/var/spool/cron` 定时任务的配置文件目录

- `/var/spool/postfix` postfix邮件目录

- `/var/spoo/clientmqueue` sendmail 临时邮件文件目录，很多原因导致目录碎片文件很多，比如crontab定时任务命令不加`>/dev/null`

- `/proc` **伪文件系统，内核映射文件**内核及**进程**信息虚拟文件系统，基于内存的虚拟文件系统，用于为内核及进程存储其相关信息；多为内核参数；例如：`net.ipv4.ip_forward`, 虚拟为 `net/ipv4/ip_forward`，存储于`/proc/sys/`，因此其完整路径为`/proc/sys/net/ipv4/ip_forward` 参考：https://www.ibm.com/developerworks/cn/linux/l-cn-sysfs/

- `/proc/filesystems` 目前系统已经加载的文件系统

- `/proc/uptime`

- `/proc/mounts` 系统已经挂载的数据

- `/proc/modules` Linux已经加载的模块列表

- `/proc/loadavg` 负载信息

- `/proc/meminfo` 内存信息

- `/proc/cpuinfo` CPU信息

- `/proc/version` 内核版本

- `/proc/sys/kernel` 系统内核功能

- `/proc/sys/net/ipv4/` 修改ipv4配置

- `/proc/sys/net/ipv4/tcp_max_tw_buckets` 36000

- `/proc/sys/net/ipv4/tcp_tw_reuse 1`

- `/proc/cmdline` 加载kernel时所下达的相关参数

- `/proc/kcore` 内存的大小

- `/proc/ioports` 目前系统上各个装置所配置的IO位址

- `/proc/interrupts`IRQ分配状态，正在使用的中断，和曾经有多少个中断

- `/sys` **伪文件系统，硬件设备相关的属性映射文件**sysfs 虚拟文件系统提供了比 proc 更为理想的访问**内核数据**的途径；其主要作用在于为管理Linux设备提供统一模型的接口；

- 《奇点临近》

## Linux文件类型

- -或f：常规文件；regular file；
- d: directory，目录文件；
- b: block device，块设备文件，支持以“block”为单位进行随机访问
- c：character device，字符设备文件，支持以“character”为单位进行线性访问
  - major number：主设备号，用于标识设备类型，进而确定要加载的驱动程序
  - minor number：次设备号，用于标识同一类型中的不同的设备
  - 8位二进制：0-255
- l：symbolic link，符号链接文件；
- p: pipe，命名管道；
- s: socket，套接字文件；

# 8 问题

### 怎么查看当前进程？怎么执行退出？怎么查看当前路径？

查看当前进程：`ps`
执行退出：`exit`
查看当前路径：`pwd`

### 怎么清屏？怎么退出当前命令？怎么执行睡眠？怎么查看当前用户 id？查看指定帮助用什么命令？

清屏：`clear`
退出当前命令：`ctrl+c` 彻底退出
执行睡眠 ：`ctrl+z` 挂起当前进程fg 恢复后台
查看当前用户 id：”id“：查看显示目前登陆账户的 uid 和 gid 及所属分组及用户名
查看指定帮助：如 man adduser 这个很全 而且有例子；adduser --help 这个告诉你一些常用参数；info adduesr；

### Ls 命令执行什么功能？可以带哪些参数，有什么区别？

ls 执行的功能：列出指定目录中的目录，以及文件
哪些参数以及区别：a 所有文件l 详细信息，包括大小字节数，可读可写可执行的权限等

### 建立软链接(快捷方式)，以及硬链接的命令。

软链接：`ln -s slink source`
硬链接：`ln link source`

### 目录创建用什么命令？创建文件用什么命令？复制文件用什么命令？

创建目录：`mkdir`
创建文件：典型的如 `touch`，`vi` 也可以创建文件，其实只要向一个不存在的文件输出，都会创建文件
复制文件：cp 7. 文件权限修改用什么命令？格式是怎么样的？
文件权限修改：`chmod`
格式如下：

> $ chmod u+x file 给 file 的属主增加执行权限 $ chmod 751 file 给 file 的属主分配读、写、执行(7)的权限，给 file 的所在组分配读、执行(5)的权限，给其他用户分配执行(1)的权限
> $ chmod u=rwx,g=rx,o=x file 上例的另一种形式 $ chmod =r file 为所有用户分配读权限
> $ chmod 444 file 同上例 $ chmod a-wx,a+r file同上例
> $ chmod -R u+r directory 递归地给 directory 目录下所有文件和子目录的属主分配读的权限

### 查看文件内容有哪些命令可以使用？

`vi 文件名` # 编辑方式查看，可修改
`cat 文件名` # 显示全部文件内容
`more 文件名` # 分页显示文件内容
`less 文件名` # 与 more 相似，更好的是可以往前翻页
`tail 文件名` # 仅查看尾部，还可以指定行数
`head 文件名` # 仅查看头部,还可以指定行数

### 随意写文件命令？怎么向屏幕输出带空格的字符串，比如”hello world”?

写文件命令：`vi`

向屏幕输出带空格的字符串:`echo hello world`

### 终端是哪个文件夹下的哪个文件？黑洞文件是哪个文件夹下的哪个命令？

终端 `/dev/tty`

黑洞文件 `/dev/null`

### 移动文件用哪个命令？改名用哪个命令？

mv mv

### Linux 下命令有哪几种可使用的通配符？分别代表什么含义?

“？”可替代单个字符。

“*”可替代任意多个字符。

方括号“[charset]”可替代 charset 集中的任何单个字符，如[a-z]，[abABC]

### 用什么命令对一个文件的内容进行统计？(行号、单词数、字节数)

wc 命令 - c 统计字节数 - l 统计行数 - w 统计字数。

### Grep 命令有什么用？如何忽略大小写？如何查找不含该串的行?

是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹 配的行打印出来。
grep [stringSTRING] filename grep \[^string\] filename

### Linux 中进程有哪几种状态？在 ps 显示出来的信息中，分别用什么符号表示的？

**1、** 不可中断状态：进程处于睡眠状态，但是此刻进程是不可中断的。不可中断， 指进程不响应异步信号。
**2、** 暂停状态/跟踪状态：向进程发送一个 SIGSTOP 信号，它就会因响应该信号 而进入 TASK_STOPPED 状态;当进程正在被跟踪时，它处于 TASK_TRACED 这个特殊的状态。
“正在被跟踪”指的是进程暂停下来，等待跟踪它的进程对它进行操作。

**3、** 就绪状态：在 run_queue 队列里的状态

**4、** 运行状态：在 run_queue 队列里的状态
**5、** 可中断睡眠状态：处于这个状态的进程因为等待某某事件的发生（比如等待 socket 连接、等待信号量），而被挂起
**6、** zombie 状态（僵尸）：父亲没有通过 wait 系列的系统调用会顺便将子进程的尸体（task_struct）也释放掉
**7、** 退出状态

> D 不可中断 Uninterruptible（usually IO）
> R 正在运行，或在队列中的进程
> S 处于休眠状态
> T 停止或被追踪
> Z 僵尸进程
> W 进入内存交换（从内核 2.6 开始无效）
> X 死掉的进程

### 利用 ps 怎么显示所有的进程? 怎么利用 ps 查看指定进程的信息？

ps -ef (system v 输出)

ps -aux bsd 格式输出

ps -ef | grep pid

### 哪个命令专门用来查看后台任务?

job -l

### 终止进程用什么命令? 带什么参数?

kill \[-s <信息名称或编号>][程序] 或 kill [-l <信息编号>\]

kill-9 pid

### 搜索文件用什么命令? 格式是怎么样的?

find <指定目录> <指定条件> <指定动作>

whereis 加参数与文件名

locate 只加文件名

find 直接搜索磁盘，较慢。

find / -name "string*"

### 查看当前谁在使用该主机用什么命令? 查找自己所在的终端信息用什么命令?

查找自己所在的终端信息：who am i

查看当前谁在使用该主机：who

### 使用什么命令查看用过的命令列表?

history

### 使用什么命令查看磁盘使用空间？空闲空间呢?

df -hl
文件系统 容量 已用 可用 已用% 挂载点
Filesystem Size Used Avail Use% Mounted on /dev/hda2 45G 19G 24G 44% /
/dev/hda1 494M 19M 450M 4% /boot

### 使用什么命令查看 ip 地址及接口信息？

ifconfig

### 通过什么命令查找执行命令?

which 只能查可执行文件

whereis 只能查二进制文件、说明文档，源文件等

### du 和 df 的定义，以及区别？

du 显示目录或文件的大小

df 显示每个<文件>所在的文件系统的信息，默认是显示所有文件系统。
（文件系统分配其中的一些磁盘块用来记录它自身的一些数据，如 i 节点，磁盘分布图，间接块，超级块等。这些数据对大多数用户级的程序来说是不可见的，通常称为 Meta Data。） du 命令是用户级的程序，它不考虑 Meta Data，而 df 命令则查看文件系统的磁盘分配图并考虑 Meta Data。

df 命令获得真正的文件系统数据，而 du 命令只查看文件系统的部分情况。

### **Unix和Linux有什么区别？**

Linux和Unix都是功能强大的操作系统，都是应用广泛的服务器操作系统，有很多相似之处，甚至有一部分人错误地认为Unix和Linux操作系统是一样的，然而，事实并非如此，以下是两者的区别。

**1、开源性**
Linux是一款开源操作系统，不需要付费，即可使用；Unix是一款对源码实行知识产权保护的传统商业软件，使用需要付费授权使用。

**2、跨平台性**
Linux操作系统具有良好的跨平台性能，可运行在多种硬件平台上；Unix操作系统跨平台性能较弱，大多需与硬件配套使用。

**3、可视化界面**
Linux除了进行命令行操作，还有窗体管理系统；Unix只是命令行下的系统。

**4、硬件环境**
Linux操作系统对硬件的要求较低，安装方法更易掌握；Unix对硬件要求比较苛刻，安装难度较大。

**5、用户群体**
Linux的用户群体很广泛，个人和企业均可使用；Unix的用户群体比较窄，多是安全性要求高的大型企业使用，如银行、电信部门等，或者Unix硬件厂商使用，如Sun等。

相比于Unix操作系统，Linux操作系统更受广大计算机爱好者的喜爱，主要原因是Linux操作系统具有Unix操作系统的全部功能，并且能够在普通PC计算机上实现全部的Unix特性，开源免费的特性，更容易普及使用！

### 查看文件内容有哪些命令可以使用？

```
vi 文件名 #编辑方式查看，可修改

cat 文件名 #显示全部文件内容

more 文件名 #分页显示文件内容

less 文件名 #与 more 相似，更好的是可以往前翻页

tail 文件名 #仅查看尾部，还可以指定行数

head 文件名 #仅查看头部,还可以指定行数
```

### 随意写文件命令？怎么向屏幕输出带空格的字符串，比如”hello world”?

写文件命令：vi

向屏幕输出带空格的字符串:echo hello world

### 终端是哪个文件夹下的哪个文件？黑洞文件是哪个文件夹下的哪个命令？

终端 /dev/tty

黑洞文件 /dev/null

### 系统调优参数

- /etc/sysctl.conf
  这个文件有没有改过？列举一些常见的kernel参数和作用。
  - time_wait相关
    - net.ipv4.tcp_tw_reuse =
      1：是否允许新的TCP连接重新应用处于time_wait状态的socket
    - net.ipv4.tcp_tw_recycle = 1：加速time_wait socket回收
    - net.ipv4.tcp_max_tw_buckets：time_wait套接字的最大数量，把time_wait所占用内存控制在一定范围
  - syn攻击相关
    - net.inet.tcp.syncookies =
      1：开启syncookies功能，防止dos攻击，syn攻击
    - net.ipv4.tcp_synack_retries =
      2：内核放弃连接之前发送SYN+ACK包的数量
    - net.ipv4.tcp_syn_retries =
      2：新连接，内核放弃连接之前发送SYN包的数量
    - net.ipv4.tcp_max_syn_backlog = 65536：表示SYN队列的长度
  - 缓冲区
    - net.core.rmem_default：接收套接字缓冲区大小缺省值
    - net.core.wmem_default：发送套接字缓冲区大小缺省值
    - net.core.rmem_max：最大TCP接收缓冲区大小
    - net.core.wmem_max：最大TCP发送缓冲区大小
  - kern.ipc.somaxconn ：并发连接数
  - net.core.netdev_max_backlog = 32768：进入包的最大设备队列

### 常见服务占用端口

- 80 8080 443
- 20 21 22 23 25 53
- 135（RPC）137（NetBIOS/UDP） 138（UDP） 139 （samba）
- 161 SNMP
- 1080 Socket代理
- 3306 11211 8080 jboss tomcat 50170

### 文件系统

- （ext4）性能 安全性

- 启动扇区 块组 超级块 inode表格 block 块对照表(Bitmap) inode对照表
  
  - 超级块
    - 记录整个文件系统的整体信息，包括inode（记录文件的权限与属性）与block（记录数据）总量、使用量、剩余量
  - inode表格 = inode + 存储block号码的block （ls -l命令）
  - inode本身不记录文件名，文件名的记录在目录的block中
  - 创建新的目录时，新目录的链接数是2（产生了/.），上层目录的链接数会增加1（产生了/..）

- 读写文件会遇到的问题
  
  - 文件数据离散：文件很大、经常变动、无法写在连续的块中、机械臂移动大、
    - 复制出来、格式化、复制回去
  - 创建文件流程
    - 查询目录权限
    - 在日志记录块中记录准备写入的信息
    - 查询inode bitmap，向inode中写入权限和属性
    - 查询block bitmap，向block写入数据
    - 更新inode指向block
    - 更新inode bitmap和block bitmap 的状态，更新superblock内容
    - 在日志记录块中完成文件记录
  - 读文件失败
    - 块数据损坏
    - inode损坏：记录数据块号码的块损坏
  - 写文件失败
    - 文件描述符不够
    - 存储空间不够了（块不够、inode不够）

- hdfs的一个block多大，为什么128M？
  
  - 不能远小于128M：减少硬盘寻道时间、减少Namenode内存消耗
  - 不能远大于128M：
    - Map崩溃问题 （数据块大，重新加载时间长）
    - 预设时间间隔问题（从数据块的角度大概估算，数据块越大，时间越长）
    - 问题分解问题：数据量大小和问题解决的复杂度成线性关系
    - 约束map输出：map之后的数据需要排序后再执行reduce，大文件不利于归并排序的思想

- ext4文件系统的block多大?
  
  - 4k
  - HDFS的块比磁盘块大，其目的是为了最小化寻址开销

- 索引式文件系统：ext

- 非索引式文件系统：FAT 碎片整理

- cp/mv/rm的区别（实现）
  
  - cp
    - -a（pdr：连同文件属性一起、链接文件属性、递归）
    - -u（新才复制）
    - -l -s （复制为链接）
    - -d
      复制链接文件时，默认复制的是源文件，除非加-d参数，才会复制链接文件
  - 当目标文件存在时，cp
    命令并不是先删除已经存在的目标文件，而是将原目标文件内容清空后再写入。
  - mv
    的主要功能就是检查初始文件和目标文件是否存在及是否有访问权限，之后执行
    rename 系统调用，因而，当目标文件存在时，mv 的行为由 rename()
    系统调用决定，即类似于删除文件后再重建一个同名文件。
  - 删除文件名是指在原目录下不再含有此文件名，并不一定删除磁盘上文件的内容。只有在文件的链接数为1，并且没有进程打开此文件的时候，unlink()
    才会真正删除文件内容。

- 软硬连接（inode这块，ln / ln -s）
  
  - 硬链接：一个inode节点对用不同的文件名，
    - 不创建新的inode，每增加一个硬链接，inode节点链接数加一
    - rm
      硬链接：删除的只是文件名，对应的数据块只有在inode节点链接数减少为0的时候才会被系统回收。
    - 不能对目录创建硬链接，因为文件系统不能存在链接环，否则会导致文件便利操作的混乱（du，pwd等命令的运作原理就是基于文件硬链接）
    - 不能跨文件系统
    - 不能对不存在的文件创建硬链接
  - 软链接：如果目标路径名较短则直接保存在inode中，如果较长则分配一个block存储
    - 创建新的inode，指向的数据块存放着源文件的路径
    - 删除源文件，软链接失效
    - 可以对目录 创建软连接，遍历操作会忽略目录的软链接
    - 可以跨文件系统
    - 可以对不存在的文件创建软链接

### 开机启动过程

- Mbr 与gpt的区别
- BIOS、CMOS、MBR、Boot
  Loader、Grub2、Kernel、/sbin/init、/etc/init/*.conf、/ect/inittab、/etc/rc.d/rc.sysinit、/etc/rc.d/rc.$runleave
- 双系统
  - 多重引导：MBR、各分区的启动扇区boot sector

###Shell常用脚本

- 从日志文件里面筛选出符合要求的ip或者其他信息
  - cat logname | sort | uniq -c | sort -nr | head -n 10
- 正则表达式匹配IP地址
  - [0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1-3}\.{1,3}
  - ^((25[0-5]|2[0-4]\d|[1]{1}\d{1}\d{1}|[1-9]{1}\d{1}|\d{1})(|(?!\\.)\.)){4}$
  - ((25[0-5]|2[0-4]\d|((1\d{2})|([1-9]?\d))).){3}(25[0-5]|2[0-4]\d|((1\d{2})|([1-9]?\d)))
- 20G大小的文件，内容都是IP，有重复的，如何找出这里面的top N ？
  - 分表、哈希
- 统计nginx日志出现次数最多的ip
  - awk '{print $1}' urllogfile | sort | uniq -c | sort -nr -k1
    | head -n 10
- 查看Web服务器（Nginx Apache）的并发请求数及其TCP连接状态
  - netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print
    a, S[a]}'

### Swap

- swap分区是怎么设置的？
  - 使用物理分区构建swap：fdisk分区（改分区类型ID）、mkswap格式化、swapon启动、free查看、
  - 使用文件构建swap：dd、mkswap、swapon、free
- 为什么要有swap分区，工作原理是什么？为什么云服务器上的swap没有开启？
  - 内存不足时，将内存中暂时不使用的程序与数据放置到swap中
  - 服务器休眠时，运行中的程序状态会被记录到swap
  - 某些程序运行时会利用swap的特性

### Iptables

- filter
  - INPUT
  - OUTPUT
  - FORWARD
- nat
  - PREROUTING
  - OUTPUT
  - POSTROUTING
- mangle
  - PREROUTING
  - INPUT
  - FORWERD
  - POSTROUTING
  - OUTPUT
- 语法
  - iptables [-t 表名] <-A|I|D|R>链名 -i|o网卡名称 -p
    协议类型 -s源IP --sport 源端口号 -d 目标IP地址 --dport
    目标端口号 <-j 动作>
  - iptables -P INPUT DROP
  - iptables -A INPUT -m state --state NEW -j DROP
  - iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
  - iptables -A INPUT -p tcp -dport 445 -j ACCEPT
  - iptables -A INPUT -p tcp -m multiport --dports 22,80 -j ACCEPT
  - 只允许某个IP上网

Linux与windows

- Linux：
  - 以进程为主，强调任务的独立性
  - 线程方面的处理：NPTL原生POSIX线程库
    - 一个线程与一个内核的调度实体一一对应
    - 新的线程同步机制：futex（快速用户空间互斥体）
  - Linux处理进程和线程的机制就是是否开启COW
    - 子进程先跟父进程共享内存，采用COW及术后，子进程还需要拷贝父进程的页面表。
- Windows
  - 以线程为主，强调任务的协同性
- windows的调度实体就是线程，进程只是一堆数据结构。而Linux不是。Linux将进程和线程做了同等对待，进程和线程在内核一级没有差别，只是通过特殊的内存映射方法使得它们从用户的角度上看来有了进程和线程的差别。
- Windows至今也没有真正的多进程概念，创建进程的开销远大于创建线程的开销。Linux则不然。Linux在内核一级并不区分进程和线程，这使得创建进程的开销和创建线程的开销差不多。
- Windows和Linux的任务调度策略也不尽相同。Windows会随着线程越来越多而变得越来越慢，这也是为什么Windows服务器在运行一段时间后必须重启的原因。Linux可以持续运行很长时间，系统的效率也不会有什么变化。

### 内核态和用户态

- 内核态和用户态的区别
  - 当进程执行系统调用而陷入内核代码中执行时，我们就称进程处于内核状态。此时处理器处于特权级最高的(0级)内核代码。当进程处于内核态时，执行的内核代码会使用当前的内核栈。每个进程都有自己的内核栈。
  - 当进程在执行用户自己的代码时，则称其处于用户态。即此时处理器在特权级最低的用户代码中运行。
  - 当正在执行用户程序而突然中断时，此时用户程序也可以象征性地处于进程的内核态。因为中断处理程序将使用当前进程的内核态。
  - 内核态与用户态是操作系统的两种运行级别，跟intel
    cpu没有必然联系，intel
    cpu提供Ring0-Ring3三种级别运行模式，Ring0级别最高，Ring3级别最低。Linux使用了Ring3级别运行用户态。Ring0作为内核态，没有使用Ring1和Ring2。Ring3不能访问Ring0的地址空间，包括代码和数量。Linux进程的4GB空间，3G-4G部分大家是共享的，是内核态的地址空间，这里存放在整个内核代码和所有的内核模块，以及内核所维护的数据。用户运行一程序，该程序所创建的进程开始是运行在用户态的，如果要执行文件操作，网络数据发送等操作，必须通过write，send等系统调用，这些系统会调用内核中的代码来完成操作，这时必须切换到Ring0，然后进入3GB-4GB中的内核地址空间去执行这些代码完成操作，完成后，切换Ring3，回到用户态。这样，用户态的程序就不能随意操作内核地址空间，具有一定的安全保护作用。
- 用户态和内核态的转换
  - 用户态切换到内核态的3种方式
    - 系统调用
      - 这是用户进程主动要求切换到内核态的一种方式，用户进程通过系统调用申请操作系统提供的服务程序完成工作。而系统调用的机制其核心还是使用了操作系统为用户特别开放的一个中断来实现，例如Linux的ine
        80h中断。
    - 异常
      - 当CPU在执行运行在用户态的程序时，发现了某些事件不可知的异常，这是会触发由当前运行进程切换到处理此异常的内核相关程序中，也就到了内核态，比如缺页异常。
    - 外围设备的中断
      - 当外围设备完成用户请求的操作之后，会向CPU发出相应的中断信号，这时CPU会暂停执行下一条将要执行的指令转而去执行中断信号的处理程序，如果先执行的指令是用户态下的程序，那么这个转换的过程自然也就发生了有用户态到内核态的切换。比如硬盘读写操作完成，系统会切换到硬盘读写的中断处理程序中执行后续操作等。
  - 具体的切换操作
    - 从出发方式看，可以在认为存在前述3种不同的类型，但是从最终实际完成由用户态到内核态的切换操作上来说，涉及的关键步骤是完全一样的，没有任何区别，都相当于执行了一个中断响应的过程，因为系统调用实际上最终是中断机制实现的，而异常和中断处理机制基本上是一样的，用户态切换到内核态的步骤主要包括：
    - （1）从当前进程的描述符中提取其内核栈的ss0及esp0信息。
    - （2）使用ss0和esp0指向的内核栈将当前进程的cs,eip，eflags，ss,esp信息保存起来，这个过程也完成了由用户栈找到内核栈的切换过程，同时保存了被暂停执行的程序的下一条指令。
    - （3）将先前由中断向量检索得到的中断处理程序的cs，eip信息装入相应的寄存器，开始执行中断处理程序，这时就转到了内核态的程序执行了。