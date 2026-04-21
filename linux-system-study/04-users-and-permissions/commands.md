# Users And Permissions 常用命令

这一章重点是把“我是谁、这个文件属于谁、谁能做什么”这三件事练熟。

## 1. `whoami`

作用：查看当前用户。

```bash
whoami
```

适合：

- 快速确认当前是谁在执行命令

## 2. `id`

作用：查看用户 ID、组 ID 和所属组。

```bash
id
id zcy
```

适合：

- 看当前用户身份信息
- 看某个用户属于哪些组

## 3. `groups`

作用：查看用户所属组。

```bash
groups
groups zcy
```

适合：

- 快速确认当前用户是否在某个组里

## 4. `ls -l`

作用：用详细格式查看权限、所有者和所属组。

```bash
ls -l
ls -ld mydir
```

说明：

- `-l`：详细格式
- `-d`：查看目录本身，而不是目录内容

适合：

- 看文件权限位
- 看目录权限位
- 看所有者和所属组

## 5. `chmod`

作用：修改权限位。

```bash
chmod 644 notes.txt
chmod 755 run.sh
chmod u+x run.sh
chmod g-w shared.txt
chmod -R 755 scripts
```

说明：

- 数字写法适合整体设置
- 符号写法适合局部增减
- `-R`：递归处理目录及其内容

常见理解：

- `644`：普通文件常见权限
- `755`：目录和可执行脚本常见权限

## 6. `chown`

作用：修改所有者，必要时也可以一起修改所属组。

```bash
sudo chown zcy notes.txt
sudo chown zcy:staff notes.txt
sudo chown -R www-data:www-data /var/www/html
```

说明：

- 通常需要更高权限
- `-R`：递归修改

适合：

- 修正文件归属
- 调整项目目录所有者

## 7. `chgrp`

作用：修改所属组。

```bash
chgrp staff notes.txt
chgrp -R staff shared
```

说明：

- `-R`：递归修改

适合：

- 保持所有者不变，只调整组归属

## 8. `sudo`

作用：以更高权限执行一条命令。

```bash
sudo ls /root
sudo chown root:root file.txt
sudo apt update
```

适合：

- 系统级操作
- 需要管理员权限的修改

提醒：

- 不要把 `sudo` 当默认前缀
- 普通用户能完成的事，尽量不要提权

## 9. `umask`

作用：查看或设置当前 Shell 的默认权限掩码。

```bash
umask
umask -S
```

适合：

- 理解为什么新文件和新目录的默认权限不是全开放

初学阶段先看值和含义，不建议随手长期修改。

## 10. `stat`

作用：查看文件更详细的属性信息。

```bash
stat notes.txt
stat mydir
```

适合：

- 辅助查看权限和时间信息
- 需要更细的元数据时使用

提醒：

- 不同系统上的输出格式会有差异

## 11. `su`

作用：切换用户身份。

```bash
su -
su zcy
```

适合：

- 在某些环境里切换到其他用户

提醒：

- 初学阶段更常用的是 `sudo`
- `su` 通常需要目标用户密码，使用场景和 `sudo` 不完全一样

## 12. 权限相关最常见的观察组合

```bash
whoami
id
groups
ls -l file.txt
ls -ld dir
stat file.txt
```

这组命令分别在回答：

- 我是谁
- 我属于哪些组
- 这个文件和目录权限长什么样
- 这个对象的详细属性是什么

## 13. 这一章建议反复练的命令组合

```bash
whoami
id
groups
touch notes.txt
ls -l notes.txt
chmod 644 notes.txt
chmod u+x run.sh
mkdir scripts
chmod 755 scripts
ls -ld scripts
umask
```

## 14. 练这一章时要特别观察什么

- `ls -l` 的三组权限分别对应谁
- 文件和目录的 `x` 权限区别
- 改权限后，`ls -l` 输出怎么变化
- `chmod` 改完后，行为是不是和预期一致
- 什么时候真的需要 `sudo`
