# Shell And Commands 常用命令

这一章重点不是命令数量，而是把最常用的命令练到熟。

## 1. `mkdir`

作用：创建目录。

```bash
mkdir notes
mkdir project backup
mkdir -p project/logs/archive
```

说明：

- `-p`：父目录不存在时一并创建

常见理解：

- `mkdir notes`：创建一个目录
- `mkdir a b`：一次创建多个目录

## 2. `touch`

作用：创建空文件，或更新时间戳。

```bash
touch todo.txt
touch notes.txt tasks.txt
touch notes/todo.txt
```

适合：

- 先造一个空文件方便练习
- 批量创建若干测试文件

## 3. `cp`

作用：复制文件或目录。

```bash
cp notes.txt notes.bak
cp notes/todo.txt backup/todo.txt
cp -r project project-backup
cp -i notes.txt notes.bak
```

说明：

- `-r`：递归复制目录
- `-i`：覆盖前询问

你应该记住：

- 第一个参数通常是源路径
- 第二个参数通常是目标路径
- 复制完成后，原文件还在

## 4. `mv`

作用：移动文件，或给文件改名。

```bash
mv old.txt new.txt
mv report.txt archive/report.txt
mv file.txt backup/
mv -i notes.bak notes.txt
```

说明：

- `-i`：覆盖前询问

你应该记住：

- 同目录下经常是在改名
- 跨目录时经常是在移动
- 原路径通常会消失

## 5. `rm`

作用：删除文件或目录。

```bash
rm old.txt
rm -r old_dir
rm -i notes.txt
rm *.log
```

说明：

- `-r`：递归删除目录
- `-i`：删除前询问

提醒：

- `rm` 删除后通常不会进回收站
- 执行通配符删除前，最好先用 `ls` 看匹配结果

## 6. `echo`

作用：输出文本或变量内容。

```bash
echo hello
echo $HOME
echo "user is $USER"
echo 'user is $USER'
```

你可以顺便观察：

- 双引号里的变量会展开
- 单引号里的内容更接近原样输出

## 7. `history`

作用：查看命令历史。

```bash
history
history | tail -n 10
```

适合：

- 找回刚才执行过的命令
- 复盘自己做过哪些操作

## 8. `type`

作用：查看一个名字在 Shell 里是什么。

```bash
type cd
type ls
type -a python3
```

适合：

- 区分内建命令和外部命令
- 查看同名命令是否有多个来源

## 9. 通配符练习

### `*`

```bash
ls *.md
rm *.tmp
cp *.txt backup/
```

含义：匹配任意长度字符。

### `?`

```bash
ls file?.txt
```

含义：匹配单个字符。

例如可能匹配：

- `file1.txt`
- `fileA.txt`

### `[abc]`

```bash
ls report-[ab].txt
```

含义：匹配括号中任意一个字符。

## 10. 重定向

### `>`

```bash
echo "hello" > hello.txt
ls > list.txt
```

作用：把标准输出写入文件，存在则覆盖。

### `>>`

```bash
echo "line 1" > app.log
echo "line 2" >> app.log
```

作用：把标准输出追加到文件末尾。

### `2>`

```bash
ls not_exist.txt 2> error.txt
```

作用：把错误输出写入文件。

## 11. 管道 `|`

```bash
ls -l | grep notes
history | tail -n 5
cat app.log | grep ERROR
```

作用：把前一个命令的标准输出交给后一个命令继续处理。

你先形成直觉就够：

- 左边负责产生数据
- 右边负责进一步筛选、统计或查看

## 12. 命令组合

### `;`

```bash
pwd; ls
```

作用：顺序执行，不管前一条成功还是失败。

### `&&`

```bash
mkdir demo && cd demo
```

作用：前一条成功后，才执行后一条。

### `||`

```bash
cd demo || echo "demo not found"
```

作用：前一条失败后，才执行后一条。

## 13. 这一章建议反复练的命令组合

```bash
mkdir notes
touch notes/todo.txt
cp notes/todo.txt notes/todo.bak
mv notes/todo.bak notes/archive.txt
echo "task 1" > notes/todo.txt
echo "task 2" >> notes/todo.txt
ls notes
ls notes | grep todo
rm notes/archive.txt
history | tail -n 5
```

## 14. 练这一章时要特别观察什么

- 同一个命令后面跟不同选项，会发生什么变化
- 同一个路径写法，加不加引号是否影响结果
- `cp` 和 `mv` 执行后，原文件是否还在
- `>` 和 `>>` 到底是覆盖还是追加
- 管道左边输出的内容，右边到底拿到了什么
