# Linux 常用命令操作示例

这份文档按“命令做什么 -> 怎么敲 -> 会看到什么 -> 什么时候用”的方式整理。

## 1. 目录和路径

### `pwd`

作用：显示当前所在目录。

```bash
pwd
```

示例输出：

```text
/home/zcy/linux-study
```

适用场景：你不知道自己当前在哪个目录时。

### `ls`

作用：列出目录内容。

```bash
ls
ls -l
ls -la
```

说明：

- `-l`: 详细格式
- `-a`: 包含隐藏文件

示例：

```bash
ls -la /etc
```

### `cd`

作用：切换目录。

```bash
cd /var/log
cd ~
cd ..
cd -
```

说明：

- `cd ~`: 回家目录
- `cd ..`: 回上一级
- `cd -`: 回到上一次所在目录

## 2. 文件和目录操作

### `mkdir`

作用：创建目录。

```bash
mkdir notes
mkdir -p project/logs/archive
```

说明：

- `-p` 会连父目录一起创建

### `touch`

作用：创建空文件，或更新文件时间戳。

```bash
touch notes.txt
touch a.txt b.txt c.txt
```

### `cp`

作用：复制文件或目录。

```bash
cp notes.txt notes.bak
cp -r project project-backup
```

说明：

- 复制目录常用 `-r`

### `mv`

作用：移动文件，或给文件改名。

```bash
mv old.txt new.txt
mv report.txt archive/report.txt
```

### `rm`

作用：删除文件或目录。

```bash
rm old.txt
rm -r old_dir
rm -rf temp_dir
```

提醒：

- `rm -rf` 很危险，初学阶段一定先用 `ls` 确认路径再删

### 一组连贯示例：命令后面的参数是什么意思

下面这组命令很适合拿来理解“参数”：

```bash
mkdir notes
touch notes/todo.txt
cp notes/todo.txt notes/todo.bak
mv notes/todo.bak notes/archive.txt
rm notes/archive.txt
```

逐条拆开：

- `mkdir notes`
  `notes` 是目录名，表示创建这个目录
- `touch notes/todo.txt`
  `notes/todo.txt` 是文件路径，表示在 `notes` 目录里创建 `todo.txt`
- `cp notes/todo.txt notes/todo.bak`
  `notes/todo.txt` 是源文件
  `notes/todo.bak` 是复制后的目标文件
- `mv notes/todo.bak notes/archive.txt`
  `notes/todo.bak` 是原文件路径
  `notes/archive.txt` 是移动后或重命名后的新路径
- `rm notes/archive.txt`
  `notes/archive.txt` 是要删除的文件路径

这里顺便区分两个概念：

- 参数：命令要处理的对象，比如 `notes/todo.txt`
- 选项：控制命令行为的附加开关，比如 `-p`、`-r`、`-f`

## 3. 输出重定向和管道

### 覆盖写入 `>`

```bash
echo "hello" > hello.txt
```

效果：如果文件不存在就创建，存在就覆盖原内容。

### 追加写入 `>>`

```bash
echo "second line" >> hello.txt
```

效果：把内容接在文件后面。

### 管道 `|`

```bash
ps aux | grep nginx
ls -l /etc | grep ssh
```

效果：把前一个命令的结果交给后一个命令处理。

## 4. 查看文本

### `cat`

作用：直接显示文件内容。

```bash
cat hello.txt
```

适合：短文件。

### `less`

作用：分页查看长文件。

```bash
less /var/log/syslog
```

常用操作：

- `Space`: 向下翻页
- `b`: 向上翻页
- `/keyword`: 搜索
- `q`: 退出

### `head`

作用：查看文件开头。

```bash
head hello.txt
head -n 5 hello.txt
```

### `tail`

作用：查看文件末尾。

```bash
tail hello.txt
tail -n 20 /var/log/syslog
tail -f /var/log/syslog
```

说明：

- `-f` 适合持续追日志

## 5. 搜索和统计

### `grep`

作用：按关键词筛选文本。

```bash
grep "error" app.log
grep -i "error" app.log
grep -n "error" app.log
grep -r "TODO" .
```

说明：

- `-i`: 忽略大小写
- `-n`: 显示行号
- `-r`: 递归搜索

### `find`

作用：按名称、类型、时间等条件查找文件。

```bash
find . -name "*.log"
find /var/log -type f
find . -type d -name "archive"
```

### `wc`

作用：统计。

```bash
wc -l app.log
wc -w article.txt
wc -c file.bin
```

说明：

- `-l`: 行数
- `-w`: 单词数
- `-c`: 字节数

## 6. 用户和权限

### `whoami`

作用：查看当前用户。

```bash
whoami
```

### `id`

作用：查看用户 ID、组 ID 和所属组。

```bash
id
```

### `chmod`

作用：修改权限。

```bash
chmod 644 notes.txt
chmod 755 deploy.sh
chmod u+x deploy.sh
```

说明：

- `u+x`: 给所有者增加执行权限

### `chown`

作用：修改所有者或所属组。

```bash
sudo chown zcy:zcy notes.txt
sudo chown -R www-data:www-data /var/www/html
```

说明：

- 通常需要管理员权限
- `-R` 表示递归修改

## 7. 进程和任务

### `ps`

作用：查看进程。

```bash
ps aux
ps aux | grep python
```

### `top`

作用：动态查看系统资源和进程状态。

```bash
top
```

适合：看哪个进程最占 CPU 或内存。

### `kill`

作用：向进程发送信号。

```bash
kill 12345
kill -9 12345
```

提醒：

- 优先尝试普通 `kill`
- `kill -9` 只在必要时用

### `jobs` `bg` `fg`

作用：管理当前 Shell 里的后台任务。

```bash
sleep 300 &
jobs
fg
```

## 8. 网络

### `ping`

作用：测试连通性。

```bash
ping 8.8.8.8
ping example.com
```

### `curl`

作用：发送网络请求。

```bash
curl https://example.com
curl -I https://example.com
curl -O https://example.com/file.txt
```

说明：

- `-I`: 只看响应头
- `-O`: 按远程文件名保存

### `ssh`

作用：远程登录。

```bash
ssh user@192.168.1.10
ssh -p 2222 user@192.168.1.10
```

说明：

- `-p`: 指定端口

### `ss`

作用：查看网络连接和监听端口。

```bash
ss -lnt
ss -tunp
```

说明：

- `-l`: 监听
- `-t`: TCP
- `-u`: UDP
- `-n`: 数字显示
- `-p`: 显示进程

## 9. 磁盘和空间

### `df`

作用：查看文件系统整体空间使用情况。

```bash
df -h
```

说明：

- `-h`: 人类可读格式

### `du`

作用：查看目录或文件占用大小。

```bash
du -sh .
du -sh /var/log/*
```

### `lsblk`

作用：查看块设备信息。

```bash
lsblk
```

适合：看磁盘、分区、挂载点关系。

## 10. 包管理和环境变量

### `apt`

Ubuntu 或 Debian 常见：

```bash
sudo apt update
sudo apt install tree
apt list --installed | grep tree
```

### `which`

作用：查命令在哪。

```bash
which ls
which python3
```

### `echo $PATH`

作用：查看命令搜索路径。

```bash
echo $PATH
```

## 11. 一组推荐你现在就练的完整例子

### 例子 1：创建学习目录并写入内容

```bash
mkdir -p ~/linux-demo/logs
echo "linux study" > ~/linux-demo/notes.txt
echo "day1" >> ~/linux-demo/notes.txt
cat ~/linux-demo/notes.txt
```

你会学到：

- 如何创建目录
- 如何写文件
- 如何查看文件

### 例子 2：筛日志中的错误

```bash
grep -i "error" ~/linux-demo/logs/app.log
```

如果文件不存在，你可以先模拟一个：

```bash
echo "INFO start" > ~/linux-demo/logs/app.log
echo "ERROR database down" >> ~/linux-demo/logs/app.log
grep -in "error" ~/linux-demo/logs/app.log
```

### 例子 3：创建并执行脚本

```bash
cat > hello.sh <<'EOF'
#!/bin/bash
echo "Hello Linux"
EOF
chmod +x hello.sh
./hello.sh
```

你会学到：

- 脚本文件长什么样
- 为什么要加执行权限
- 为什么执行脚本常用 `./`
