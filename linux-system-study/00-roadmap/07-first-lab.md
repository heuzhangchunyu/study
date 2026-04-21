# Linux 第一轮综合实验

这份实验的目标是把零散命令串起来。你做完后，不只是“见过命令”，而是完成了一次完整练习。

建议在真正的 Linux 环境中做。如果你暂时没有 Linux 主机，目录、文本、权限、脚本这几部分也可以先在本地终端练。

## 实验目标

完成后你应该能做到：

- 创建目录和文件
- 写入和查看文本
- 搜索内容和查找文件
- 修改权限
- 启动和结束一个后台任务
- 发起一次网络请求
- 写一个简单脚本

## 第 1 步：创建实验目录

```bash
mkdir -p ~/linux-lab/{notes,logs,scripts,backup}
cd ~/linux-lab
pwd
ls -l
```

你应该理解：

- `-p` 可以连父目录一起建
- `{a,b,c}` 是一种快速展开写法
- `pwd` 用来确认当前位置

## 第 2 步：创建笔记和日志文件

```bash
echo "Linux lab start" > notes/intro.txt
echo "INFO application started" > logs/app.log
echo "WARN cache miss" >> logs/app.log
echo "ERROR database timeout" >> logs/app.log
```

检查结果：

```bash
cat notes/intro.txt
cat logs/app.log
```

你应该看到：

```text
INFO application started
WARN cache miss
ERROR database timeout
```

这里你会学到：

- `>` 覆盖写入
- `>>` 追加写入

## 第 3 步：查看文件头尾和搜索错误

```bash
head -n 2 logs/app.log
tail -n 2 logs/app.log
grep -in "error" logs/app.log
```

解释：

- `head -n 2`: 看前 2 行
- `tail -n 2`: 看后 2 行
- `grep -in`: 忽略大小写并显示行号

如果搜索结果是：

```text
3:ERROR database timeout
```

说明第 3 行包含错误信息。

## 第 4 步：查找文件

```bash
find . -name "*.log"
find . -type d -name "scripts"
```

你应该理解：

- `-name` 按名称匹配
- `*.log` 匹配所有 `.log` 文件
- `-type d` 表示只查目录

## 第 5 步：复制、移动和删除

```bash
cp logs/app.log backup/app.log.bak
mv notes/intro.txt notes/intro-day1.txt
ls -l notes backup
rm backup/app.log.bak
```

操作后检查：

```bash
ls -l backup
```

如果目录为空，说明删除成功。

## 第 6 步：练习权限

先创建脚本：

```bash
echo 'echo "running lab script"' > scripts/run.sh
ls -l scripts/run.sh
```

此时大概率没有执行权限。给它加上：

```bash
chmod 755 scripts/run.sh
ls -l scripts/run.sh
./scripts/run.sh
```

你应该观察 `ls -l` 输出中权限位的变化，比如从：

```text
-rw-r--r--
```

变成：

```text
-rwxr-xr-x
```

## 第 7 步：练习后台任务和进程

启动一个后台任务：

```bash
sleep 300 &
jobs
```

再找出它的进程：

```bash
ps aux | grep sleep
```

结束它：

```bash
kill <PID>
```

把 `<PID>` 替换成你查到的进程号。

你应该理解：

- `&` 会让命令放到后台
- `jobs` 管理当前 Shell 的后台任务
- `ps aux` 用来看系统进程

## 第 8 步：做一次网络请求

```bash
curl -I https://example.com
```

你会看到类似：

```text
HTTP/2 200
content-type: text/html
```

这说明：

- 请求已经发出
- 远程站点有正常响应

如果你在 Linux 主机上，还可以继续：

```bash
ss -lnt
```

查看当前机器有哪些 TCP 端口在监听。

## 第 9 步：查看空间占用

```bash
du -sh .
df -h
```

理解：

- `du -sh .`: 当前目录占了多大空间
- `df -h`: 各文件系统整体还有多少空间

## 第 10 步：写一个简单备份脚本

创建脚本文件：

```bash
cat > scripts/backup_logs.sh <<'EOF'
#!/bin/bash
cp logs/app.log backup/app.log.$(date +%Y%m%d%H%M%S).bak
echo "backup finished"
EOF
```

给权限并执行：

```bash
chmod +x scripts/backup_logs.sh
./scripts/backup_logs.sh
ls -l backup
```

你应该看到 `backup` 目录下多出一个带时间戳的备份文件。

这个脚本学到的是：

- 脚本首行 `#!/bin/bash` 的作用
- 变量替换和命令替换的基本感觉
- 如何把重复操作写成自动化脚本

## 复盘问题

做完实验后，建议你自己回答下面几个问题：

1. 为什么 `chmod +x` 后脚本才能直接执行？
2. 为什么 `grep -in "error"` 比只用 `grep` 更适合排查日志？
3. `du -sh` 和 `df -h` 分别回答的是什么问题？
4. 如果 `./scripts/run.sh` 提示没有权限，你第一反应该查什么？
5. 如果 `curl -I https://example.com` 失败，可能是网络、DNS 还是目标站点问题？

## 达标标准

如果你能不看提示，独立完成下面这几件事，就说明你已经有了比较扎实的 Linux 入门手感：

- 自己创建一套练习目录
- 自己构造一份日志并用 `grep` 搜索
- 自己给脚本加执行权限并运行
- 自己启动一个后台任务再结束它
- 自己写一个最简单的备份脚本
