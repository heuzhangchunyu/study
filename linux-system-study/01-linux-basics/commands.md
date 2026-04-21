# Linux Basics 常用命令

这一章的命令目标很明确：先会观察系统、确认自己在哪、知道怎么查帮助。

## 1. `pwd`

作用：显示当前所在目录。

```bash
pwd
```

示例输出：

```text
/home/zcy/projects
```

你应该理解：

- 输出的是当前工作目录
- 后面很多相对路径，都是以这里为参照

## 2. `ls`

作用：列出目录内容。

```bash
ls
ls -l
ls -la
```

说明：

- `-l`：详细格式显示
- `-a`：包含隐藏文件

推荐先练：

```bash
ls /
ls -la ~
```

## 3. `cd`

作用：切换目录。

```bash
cd /tmp
cd ~
cd ..
cd -
```

说明：

- `cd ~`：回家目录
- `cd ..`：回上一级
- `cd -`：回到上一次所在目录

## 4. `whoami`

作用：查看当前用户。

```bash
whoami
```

适合用来回答一个很基础的问题：我现在到底是哪个用户在操作系统。

## 5. `id`

作用：查看当前用户的用户 ID、组 ID 和所属组。

```bash
id
```

你现在不需要把输出全背下来，但要知道系统里“用户”和“组”是重要概念。

## 6. `uname`

作用：查看系统内核或系统信息。

```bash
uname
uname -a
```

说明：

- `uname`：只显示内核名
- `uname -a`：显示更多系统信息

如果你在真正的 Linux 环境里练，这条命令很常用。

## 7. `hostname`

作用：查看主机名。

```bash
hostname
```

它常和用户、路径一起出现在提示符里。

## 8. `echo`

作用：输出文本或环境变量内容。

```bash
echo hello
echo $SHELL
echo $PATH
echo $HOME
```

你这一章重点先用它观察环境变量。

## 9. `which`

作用：查看命令大概率会从哪里找到。

```bash
which ls
which bash
which curl
```

说明：

- 它常用来快速确认某个命令文件的位置
- 如果一个命令有多个来源，更完整的查看方式通常是 `type -a`

## 10. `type`

作用：查看一个名字在 Shell 里到底是什么。

```bash
type ls
type cd
type -a python3
```

这条命令很有价值，因为它能帮助你区分：

- 这是外部程序
- 这是 Shell 内建命令
- 这个名字是否有多个来源

## 11. `man`

作用：查看命令手册。

```bash
man ls
man pwd
```

常用操作：

- `Space`：向下翻页
- `b`：向上翻页
- `/keyword`：搜索
- `q`：退出

## 12. `--help`

作用：查看命令的简要帮助。

```bash
ls --help
cp --help
grep --help
```

适合快速确认：

- 这个命令能做什么
- 常用选项叫什么
- 基本语法长什么样

## 13. 第一章建议你一定跑一遍的命令

按顺序练一遍：

```bash
whoami
pwd
ls
ls -la
cd /
pwd
ls
cd ~
echo $SHELL
echo $PATH
which ls
type cd
man pwd
```

## 14. 练这一章命令时要观察什么

不要只是“命令跑通了”，而要有意识地看这些点：

- 我当前在哪个目录
- 当前用户是谁
- 当前系统用的是什么 Shell
- 命令是系统自带的，还是后装的
- 我不知道的命令，能不能自己查帮助
