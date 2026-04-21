# 实验 01：文件查看、搜索和日志处理

这个实验的目标是把这一章的命令串成一个完整工作流。

完成后你应该能做到：

- 创建一组练习文件
- 判断文件类型和查看文件元数据
- 选择合适的方式查看短文件和长文件
- 在文本里搜索关键字
- 在目录里查找指定文件
- 对文本做基础统计、排序和去重

## 实验准备

建议先进入一个干净的练习目录：

```bash
cd ~
mkdir -p linux-text-lab/{logs,notes,data}
cd linux-text-lab
pwd
```

## 第 1 步：创建练习文件

```bash
echo "Linux filesystem note" > notes/intro.txt
echo "INFO app start" > logs/app.log
echo "WARN cache miss" >> logs/app.log
echo "ERROR database timeout" >> logs/app.log
echo "INFO retry success" >> logs/app.log
printf "tom\namy\ntom\nbob\namy\n" > data/names.txt
ls -l notes logs data
```

你应该观察：

- 三个目录都已经有内容
- `logs/app.log` 有多行日志
- `data/names.txt` 有重复名字

## 第 2 步：识别文件和元数据

```bash
file notes/intro.txt
file logs/app.log
stat logs/app.log
```

你应该理解：

- `file` 更像快速识别
- `stat` 更像详细属性查看

## 第 3 步：分别用不同方式查看内容

```bash
cat notes/intro.txt
head -n 2 logs/app.log
tail -n 2 logs/app.log
less logs/app.log
```

观察：

- `cat` 适合短文件
- `head` 和 `tail` 更适合快速抓重点
- `less` 更适合分页看长文件

进入 `less` 后试试：

- `/ERROR` 搜索
- `q` 退出

## 第 4 步：搜索日志中的错误

```bash
grep "ERROR" logs/app.log
grep -in "error" logs/app.log
```

你应该观察：

- 第二条命令能忽略大小写并显示行号

如果输出像这样：

```text
3:ERROR database timeout
```

说明第 3 行包含错误信息。

## 第 5 步：查找文件和目录

```bash
find . -name "*.log"
find . -type f -name "*.txt"
find . -type d -name "logs"
```

你应该理解：

- `-name` 用来按名称匹配
- `-type f` 只查普通文件
- `-type d` 只查目录

## 第 6 步：统计和整理文本

```bash
wc -l logs/app.log
sort data/names.txt
sort data/names.txt | uniq
sort data/names.txt | uniq -c
```

你应该观察：

- `wc -l` 能快速告诉你日志有多少行
- `sort` 只是排序，不去重
- `uniq` 处理的是相邻重复行
- `uniq -c` 能看到每个名字出现了几次

## 第 7 步：模拟日志持续追加

先打开一个终端运行：

```bash
tail -f logs/app.log
```

再在另一个终端追加一行：

```bash
echo "ERROR payment timeout" >> logs/app.log
```

你应该看到第一边的 `tail -f` 立刻出现新日志。

结束时按：

```text
Ctrl + C
```

## 第 8 步：做一次完整的小排查

假设你只知道“程序报错了”，你可以按这个顺序做：

```bash
find . -name "*.log"
tail -n 5 logs/app.log
grep -in "error" logs/app.log
wc -l logs/app.log
```

这组动作分别回答：

- 日志文件在哪
- 最近几行写了什么
- 错误出现在哪一行
- 整个日志大概多大

## 实验完成标准

如果你已经能做到下面这些，说明这个实验达标了：

- 能根据文件长短选择 `cat`、`less`、`head` 或 `tail`
- 能区分“找文件”和“找内容”
- 能用 `grep` 找到关键字并看行号
- 能用 `find` 按名称和类型查路径
- 能用 `sort`、`uniq`、`wc` 做基础整理

## 实验后复盘

建议你自己再回答这 5 个问题：

1. 哪种查看方式最适合日志文件，为什么？
2. 为什么 `grep -in` 比单独 `grep` 更适合排查？
3. `find` 和 `grep` 在你这次实验里各自解决了什么问题？
4. 为什么 `uniq` 前面经常要先接 `sort`？
5. 如果明天给你一个很长的日志文件，你第一步会先敲什么？
