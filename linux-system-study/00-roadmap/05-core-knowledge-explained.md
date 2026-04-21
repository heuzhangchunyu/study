# Linux 核心知识详解

这份文档的目标不是列清单，而是把 Linux 入门最重要的概念讲清楚，并配上可以直接动手的例子。

## 1. Linux 到底是什么

很多人刚开始学 Linux，会把下面几个词混在一起：

- `Linux`: 严格说通常指 Linux 内核
- `发行版`: 在 Linux 内核外面，再打包命令工具、包管理器、桌面环境、系统服务后的完整系统，比如 Ubuntu、Debian、CentOS、Rocky Linux
- `Shell`: 你和系统交互的命令解释器，比如 `bash`、`zsh`
- `Terminal`: 打开命令行窗口的程序

可以把它理解成：

- 内核负责管理硬件和系统资源
- Shell 负责接收你输入的命令
- Terminal 负责把这个交互界面显示出来

例如你在终端里输入：

```bash
pwd
```

背后大致发生的是：

1. Terminal 把你输入的字符显示出来
2. Shell 识别你输入的是 `pwd`
3. Shell 调用对应程序
4. 程序向系统请求当前工作目录
5. 结果再显示回终端

## 2. 为什么 Linux 学习离不开命令行

图形界面适合日常使用，但 Linux 学习必须重视命令行，原因很简单：

- 服务器环境经常只有命令行
- 命令行可重复、可脚本化、可自动化
- 排查问题时，日志、进程、网络信息往往都通过命令查看

Linux 命令一般长这样：

```bash
command [options] [arguments]
```

例如：

```bash
ls -l /etc
```

拆开看：

- `ls`: 列出文件
- `-l`: 使用详细格式
- `/etc`: 要查看的目录

一个很重要的学习习惯是：不要只背命令名，要理解“命令 + 选项 + 目标对象”的结构。

## 3. 路径、目录和文件系统

### 3.1 当前目录是什么

你在终端里总是“站在某个位置”上，这个位置叫当前工作目录。

查看当前位置：

```bash
pwd
```

示例输出：

```text
/home/zcy/projects
```

这表示你当前正站在 `/home/zcy/projects` 目录里。

### 3.2 绝对路径和相对路径

绝对路径：从根目录 `/` 开始写的完整路径。

```bash
/home/zcy/projects/linux-study
```

相对路径：相对于你当前所在目录来写的路径。

```bash
./linux-study
../documents
```

常见符号：

- `/`: 根目录
- `.`: 当前目录
- `..`: 上一级目录
- `~`: 当前用户家目录

例如：

```bash
cd ~
cd ..
cd /var/log
```

### 3.3 Linux 常见目录的作用

初学阶段，下面这些目录必须认识：

- `/`: 整个文件系统的起点
- `/home`: 普通用户的家目录
- `/root`: root 用户的家目录
- `/etc`: 配置文件目录
- `/var`: 经常变化的数据，比如日志
- `/tmp`: 临时文件
- `/usr`: 大量系统程序和共享资源
- `/bin`: 基础命令
- `/sbin`: 系统管理命令

可以先运行：

```bash
ls /
```

再逐个观察这些目录名，先建立“这台 Linux 长什么样”的直觉。

## 4. 文件和目录操作的本质

Linux 里大多数工作，归根到底都在做下面几件事：

- 找到文件
- 查看文件
- 创建文件
- 修改文件
- 删除文件
- 调整文件权限

例如下面一组命令：

```bash
mkdir notes
touch notes/todo.txt
cp notes/todo.txt notes/todo.bak
mv notes/todo.bak notes/archive.txt
rm notes/archive.txt
```

它们分别表示：

- `mkdir`: 创建目录
- `touch`: 创建空文件
- `cp`: 复制
- `mv`: 移动或重命名
- `rm`: 删除

你可以把 Linux 初学理解成：“围绕文件和进程做管理”。

如果再往下拆一层，这些命令后面跟的内容叫参数，也就是“你要操作谁”：

- `mkdir notes`
  `notes` 是目录名，表示创建一个叫 `notes` 的目录
- `touch notes/todo.txt`
  `notes/todo.txt` 是文件路径，表示在 `notes` 目录里创建或更新时间戳为 `todo.txt` 的文件
- `cp notes/todo.txt notes/todo.bak`
  第一个参数 `notes/todo.txt` 是源文件
  第二个参数 `notes/todo.bak` 是目标文件
- `mv notes/todo.bak notes/archive.txt`
  第一个参数 `notes/todo.bak` 是原路径
  第二个参数 `notes/archive.txt` 是新路径
- `rm notes/archive.txt`
  `notes/archive.txt` 是要删除的目标文件

可以先记住这个规律：

- `mkdir`、`touch`、`rm` 后面通常跟“目标路径”
- `cp`、`mv` 后面通常跟“源路径 + 目标路径”
- 像 `-p`、`-r` 这种以 `-` 开头的内容通常是选项，不是参数

## 5. 重定向和管道为什么重要

这是从“会敲命令”进入“会组合命令”的关键。

### 5.1 重定向

把命令输出写入文件：

```bash
echo "hello linux" > hello.txt
```

含义：

- `>` 会覆盖写入

追加写入：

```bash
echo "second line" >> hello.txt
```

含义：

- `>>` 会追加，不覆盖原内容

### 5.2 管道

把前一个命令的输出，交给后一个命令继续处理：

```bash
ps aux | grep ssh
```

含义：

- `ps aux` 列出进程
- `grep ssh` 从结果里筛出带 `ssh` 的行

学习管道的核心，不是记住某一条组合，而是理解“输出可以继续喂给下一个命令”。

## 6. 文本处理为什么是 Linux 基础能力

Linux 里大量信息都以文本形式存在：

- 配置文件是文本
- 日志是文本
- 命令输出大多也是文本

所以你要学会看文本、筛文本、统计文本。

常见命令：

- `cat`: 直接输出全文
- `less`: 分页查看，适合长文件
- `head`: 看前几行
- `tail`: 看后几行
- `grep`: 关键词搜索
- `wc`: 统计行数、单词数、字符数

例如查看日志末尾：

```bash
tail -n 20 /var/log/syslog
```

如果想持续追踪新日志：

```bash
tail -f /var/log/syslog
```

如果要搜索错误关键词：

```bash
grep -i "error" /var/log/syslog
```

这里的 `-i` 表示忽略大小写。

## 7. 权限模型是 Linux 的核心

Linux 不只是“能不能打开文件”，而是“谁能对什么做什么”。

### 7.1 看懂 `ls -l`

执行：

```bash
ls -l
```

可能看到：

```text
-rw-r--r-- 1 zcy staff 120 Apr 21 10:00 notes.txt
```

这串内容可以这样拆：

- 第 1 位 `-`: 文件类型，`-` 表示普通文件，`d` 表示目录
- 接下来的 9 位：权限位
- `zcy`: 文件所有者
- `staff`: 文件所属组

权限位分三组：

- `rw-`: 所有者权限
- `r--`: 所属组权限
- `r--`: 其他用户权限

每组三个字符分别表示：

- `r`: 读
- `w`: 写
- `x`: 执行

### 7.2 权限的实际意义

对普通文件来说：

- `r`: 可以读取内容
- `w`: 可以修改内容
- `x`: 可以把它当程序执行

对目录来说：

- `r`: 可以查看目录中的名字
- `w`: 可以在目录中增删改条目
- `x`: 可以进入目录

### 7.3 `chmod` 怎么理解

例如：

```bash
chmod 644 notes.txt
chmod 755 script.sh
```

数字权限的含义：

- `r = 4`
- `w = 2`
- `x = 1`

所以：

- `6 = 4 + 2 = rw-`
- `4 = r--`
- `5 = 4 + 1 = r-x`

`644` 表示：

- 所有者：读写
- 组用户：只读
- 其他用户：只读

`755` 表示：

- 所有者：读写执行
- 组用户：读执行
- 其他用户：读执行

### 7.4 `sudo` 是什么

`sudo` 不是命令增强版，而是“临时以更高权限执行一条命令”。

例如：

```bash
sudo apt update
```

这表示以管理员权限执行包索引更新。

原则是：

- 能不用 `sudo` 就别用
- 只在确实需要系统级修改时使用

## 8. 进程、作业和服务

### 8.1 进程是什么

程序是放在磁盘上的文件，进程是程序运行起来后的状态。

例如你打开一个 `nginx` 服务，它运行起来后就会有一个或多个进程。

查看进程：

```bash
ps aux
```

或者：

```bash
top
```

### 8.2 如何结束进程

先查进程：

```bash
ps aux | grep nginx
```

再结束：

```bash
kill 12345
```

这里的 `12345` 是进程号 PID。

如果进程不响应，可以强制结束：

```bash
kill -9 12345
```

但要知道，`-9` 是强制信号，不适合乱用，因为它不给程序清理机会。

### 8.3 前台和后台

如果你执行：

```bash
sleep 300
```

终端会被占住，这叫前台运行。

如果改成：

```bash
sleep 300 &
```

它会放到后台。

查看后台任务：

```bash
jobs
```

把后台任务拉回前台：

```bash
fg
```

### 8.4 服务是什么

服务可以理解为“长期在后台运行、为系统或用户提供能力的程序”。

在 Linux 上常见服务有：

- `sshd`: 远程登录
- `nginx`: Web 服务
- `cron`: 定时任务

基于 `systemd` 的系统通常用：

```bash
systemctl status ssh
systemctl start nginx
systemctl stop nginx
```

## 9. 网络基础必须会哪些

初学阶段不用一开始就深挖网络协议，但要先把几个词分清：

- `IP`: 设备在网络中的地址
- `端口`: 同一台机器上不同服务的入口编号
- `DNS`: 把域名解析成 IP
- `SSH`: 远程登录协议

### 9.1 连通性检查

```bash
ping 8.8.8.8
ping www.baidu.com
```

如果 `ping IP` 通，但 `ping 域名` 不通，常常说明 DNS 有问题。

### 9.2 测试 HTTP 请求

```bash
curl -I https://example.com
```

这里的 `-I` 表示只看响应头。

### 9.3 查看端口监听

```bash
ss -lnt
```

含义：

- `l`: 只看监听中的连接
- `n`: 数字方式显示，不做名称解析
- `t`: 只看 TCP

### 9.4 远程登录

```bash
ssh user@192.168.1.10
```

这表示用用户 `user` 登录远程机器 `192.168.1.10`。

## 10. 包管理和环境变量

### 10.1 为什么包管理重要

在 Linux 里安装软件，通常不是双击安装包，而是通过包管理器。

例如 Ubuntu 或 Debian 常见：

```bash
sudo apt update
sudo apt install curl
```

Red Hat 系常见：

```bash
sudo dnf install curl
```

### 10.2 `PATH` 是什么

当你输入：

```bash
ls
```

Shell 会去 `PATH` 环境变量列出的目录里找 `ls` 这个可执行文件。

查看 `PATH`：

```bash
echo $PATH
```

查看命令实际位置：

```bash
which ls
```

如果一个命令提示 `command not found`，通常从这几个方向排查：

1. 软件没安装
2. 可执行文件不在 `PATH` 中
3. 命令名写错了

## 11. Shell 脚本为什么值得学

当你发现一串命令总要重复执行时，就可以把它写成脚本。

最小 Bash 脚本示例：

```bash
#!/bin/bash
echo "Hello Linux"
```

保存为 `hello.sh` 后：

```bash
chmod +x hello.sh
./hello.sh
```

带变量的例子：

```bash
#!/bin/bash
name="zcy"
echo "Hello, $name"
```

带参数的例子：

```bash
#!/bin/bash
echo "First arg: $1"
```

执行：

```bash
./script.sh linux
```

输出：

```text
First arg: linux
```

## 12. 入门阶段最该建立的思维

初学 Linux，最重要的不是立刻记住所有命令，而是建立下面几个思维：

- 任何问题先看路径和当前目录
- 看不到内容时先判断是文件不存在还是权限不够
- 程序出问题先看进程、日志、端口
- 命令不会时先查 `man`、`--help`、`which`
- 复杂任务不要手工重复，想想能不能写成脚本

## 13. 一组建议你现在就练的命令

如果你今天刚开始学，先只练这 12 个：

- `pwd`
- `ls -l`
- `cd`
- `mkdir`
- `touch`
- `cp`
- `mv`
- `rm`
- `cat`
- `grep`
- `find`
- `chmod`

把这 12 个练熟以后，你对 Linux 命令行就已经入门了。
