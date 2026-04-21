# 实验 01：文件操作、重定向和管道

这个实验的目标是把这一章最核心的命令串成一个完整流程。

完成后你应该能做到：

- 创建目录和文件
- 复制、移动、删除文件
- 使用通配符匹配多个文件
- 使用 `>` 和 `>>` 写入文件
- 使用 `|` 把一个命令的结果交给另一个命令处理

## 实验准备

先进入一个你愿意练习的目录，比如家目录下的临时学习区。

建议先执行：

```bash
cd ~
mkdir -p linux-shell-lab
cd linux-shell-lab
pwd
```

## 第 1 步：创建练习目录和文件

```bash
mkdir notes backup
touch notes/todo.txt
touch notes/day1.txt notes/day2.txt notes/day3.txt
ls -l notes
```

你应该观察：

- `notes` 和 `backup` 被创建出来了
- `touch` 可以一次创建多个文件

## 第 2 步：写入内容

```bash
echo "task 1: learn shell" > notes/todo.txt
echo "task 2: practice cp" >> notes/todo.txt
cat notes/todo.txt
```

你应该观察：

- 第一条命令是覆盖写入
- 第二条命令是追加写入

如果最终文件内容有两行，说明操作正确。

## 第 3 步：复制和改名

```bash
cp notes/todo.txt backup/todo.txt
cp notes/todo.txt notes/todo.bak
mv notes/todo.bak notes/archive.txt
ls -l notes backup
```

你应该观察：

- `backup/todo.txt` 是复制出来的新文件
- `notes/archive.txt` 原本来自 `notes/todo.bak`
- `notes/todo.bak` 已经不存在了

## 第 4 步：练习通配符

先看匹配结果：

```bash
ls notes/*.txt
ls notes/day?.txt
```

你应该理解：

- `notes/*.txt` 会匹配 `notes` 目录下所有 `.txt` 文件
- `notes/day?.txt` 会匹配像 `day1.txt`、`day2.txt` 这种单字符占位文件

试着复制一批文件：

```bash
cp notes/day?.txt backup/
ls backup
```

## 第 5 步：用管道筛结果

```bash
ls notes | grep todo
ls backup | grep day
```

你应该理解：

- `ls notes` 先产生文件名列表
- `grep todo` 再从结果中筛出包含 `todo` 的项

## 第 6 步：体验命令组合

```bash
mkdir logs && cd logs
pwd
cd ..
cd missing-dir || echo "directory not found"
```

你应该理解：

- `&&` 适合把“前一步成功后才继续”的操作串起来
- `||` 适合在失败时给出补充提示

## 第 7 步：删除前先确认

先看：

```bash
ls notes/*.txt
```

再删：

```bash
rm notes/archive.txt
ls notes
```

这里想让你形成的习惯是：

- 先确认匹配范围
- 再执行删除

## 第 8 步：查看最近命令

```bash
history | tail -n 10
```

看看你刚才做的实验步骤是否都能在历史记录里找到。

## 实验完成标准

如果你已经能做到下面这些，说明这个实验达标了：

- 能自己完成一轮文件创建、复制、移动和删除
- 能说出 `cp` 和 `mv` 的区别
- 能理解 `>`、`>>` 和 `|` 的作用
- 能看懂 `*`、`?` 的基本匹配逻辑
- 能把几条命令组合成一个小流程

## 实验后复盘

建议你自己再回答这 5 个问题：

1. 我在哪一步最容易输错命令？
2. 哪个命令我只是照着敲，还没有真正理解？
3. 哪个地方最能体现“Shell 在先处理，再执行命令”？
4. 如果目录名里有空格，我哪几步必须改写？
5. 以后执行删除命令前，我应该先做什么？
