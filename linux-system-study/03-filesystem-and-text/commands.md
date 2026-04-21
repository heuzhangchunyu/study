# Filesystem And Text 常用命令

这一章重点是把“文件在哪里”和“文件里写了什么”这两件事练熟。

## 1. `ls -l`

作用：用详细格式查看文件和目录信息。

```bash
ls -l
ls -lh
ls -la
```

说明：

- `-l`：详细格式
- `-h`：人类可读大小
- `-a`：显示隐藏文件

适合：

- 看文件类型、大小、修改时间
- 看隐藏文件

## 2. `file`

作用：粗略判断文件类型。

```bash
file notes.txt
file /bin/ls
file image.png
```

适合：

- 判断一个文件更像文本、程序还是图片
- 遇到陌生文件时先做快速识别

## 3. `stat`

作用：查看更详细的文件元数据。

```bash
stat notes.txt
stat logs/app.log
```

适合：

- 看文件大小
- 看访问时间、修改时间、状态变化时间
- 看权限和 inode 等详细属性

注意：

- 不同系统上 `stat` 输出格式会有差异

## 4. `cat`

作用：直接输出文件全部内容。

```bash
cat hello.txt
cat notes.txt tasks.txt
```

适合：

- 查看短文件
- 把几个小文件内容连续输出

不太适合：

- 特别长的文件
- 持续增长的日志

## 5. `less`

作用：分页查看长文件。

```bash
less app.log
less /etc/services
```

常用操作：

- `Space`：向下翻页
- `b`：向上翻页
- `/keyword`：搜索
- `q`：退出

适合：

- 长文件
- 日志浏览
- 边看边搜索

## 6. `head`

作用：查看文件开头部分。

```bash
head app.log
head -n 5 app.log
```

说明：

- 默认通常看前 10 行
- `-n`：指定显示行数

适合：

- 快速确认文件格式
- 先看看开头写了什么

## 7. `tail`

作用：查看文件结尾部分。

```bash
tail app.log
tail -n 20 app.log
tail -f app.log
```

说明：

- `-n`：指定显示行数
- `-f`：持续跟踪文件末尾的新内容

适合：

- 看日志最后几行
- 跟踪日志追加输出

提醒：

- `tail -f` 会一直运行，通常按 `Ctrl + C` 退出

## 8. `grep`

作用：按关键词筛选文本。

```bash
grep "error" app.log
grep -i "error" app.log
grep -n "error" app.log
grep -in "error" app.log
grep -r "TODO" .
```

说明：

- `-i`：忽略大小写
- `-n`：显示行号
- `-r`：递归搜索目录

适合：

- 在日志里找错误
- 在项目里找关键字
- 快速定位某段文本出现的位置

## 9. `find`

作用：按条件查找文件或目录。

```bash
find . -name "*.log"
find . -type f
find . -type d -name "backup"
find /var/log -name "*.log"
```

说明：

- `-name`：按名称匹配
- `-type f`：只找普通文件
- `-type d`：只找目录

适合：

- 找某类文件
- 找某个目录
- 在不清楚位置时全局查路径

## 10. `wc`

作用：统计行数、单词数、字节数。

```bash
wc notes.txt
wc -l app.log
wc -w article.txt
wc -c image.bin
```

说明：

- `-l`：行数
- `-w`：单词数
- `-c`：字节数

适合：

- 快速看日志有多少行
- 粗略统计文本规模

## 11. `sort`

作用：排序文本行。

```bash
sort names.txt
sort app.log
sort -r names.txt
```

说明：

- `-r`：反向排序

适合：

- 先整理文本
- 为后续 `uniq` 做准备

## 12. `uniq`

作用：处理相邻重复行。

```bash
uniq names.txt
sort names.txt | uniq
sort names.txt | uniq -c
```

说明：

- `-c`：统计重复次数

提醒：

- `uniq` 通常要和 `sort` 一起用，效果更稳定

## 13. `cut`

作用：按分隔规则取出一部分文本字段。

```bash
cut -d ':' -f 1 /etc/passwd
cut -d ',' -f 2 users.csv
```

说明：

- `-d`：指定分隔符
- `-f`：指定字段位置

适合：

- 从结构化文本里抽取某一列

## 14. 这一章建议反复练的命令组合

```bash
cat app.log
head -n 5 app.log
tail -n 5 app.log
grep -in "error" app.log
find . -name "*.log"
wc -l app.log
sort names.txt | uniq
ls -l
file app.log
stat app.log
```

## 15. 练这一章时要特别观察什么

- 文件很多时，哪种查看方式最省力
- 你是在找文件本身，还是找文件内容
- `grep -n` 加上行号后，排查是不是更顺
- `sort` 后再接 `uniq`，结果和直接 `uniq` 有什么差别
- `tail -f` 和普通 `tail` 的体验差在哪里
