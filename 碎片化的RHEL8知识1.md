# 碎片化的RHEL8知识1

## bash的快捷键

- `ctrl + c`废弃当前编辑的命令行(结束正在运行的命令)
- `Esc + .`和 `Alt + .`粘贴上个命令的参数
- `ctrl + l` 清空整个屏幕
- `ctrl + u` 剪切至行首
- `ctrl + x` 剪切至行尾
- `ctrl + w` 往回剪切一部分(以空格界定)
- `ctrl + y`粘贴剪切的文本
- `ctrl + shift + t` 新开一个终端

## `date`查看系统时间

`date`命令的意思是日期，默认为显示

语法：`date +'[参数][参数]...'`

```bash
[root@localhost ~]# date
Thu Nov 26 23:51:51 CST 2020
```

**`date`命令可选的参数如下：**

- `%F`：日期（YYYY-mm-dd）
- `%R`：时间（HH:MM）
- `%Y`：年
- `%m`：月
- `%d`：日
- `%H`：时
- `%M`：分
- `%S`：秒

使用方法如下

```bash
[root@localhost ~]# date +'%F %R'
2020-11-26 23:52
[root@localhost ~]# date +'%Y-%m-%d %H:%M:%S'
2020-11-26 23:52:40
[root@localhost ~]# date +'%F %R:%S'
2020-11-26 23:52:47
```

## `cal`显示日历

`cal`命令是单词 `calendar`(日历)的缩写

语法：`cal -[参数]`

```bash
[root@localhost ~]# cal
    November 2020   
Su Mo Tu We Th Fr Sa
 1  2  3  4  5  6  7
 8  9 10 11 12 13 14
15 16 17 18 19 20 21
22 23 24 25 26 27 28
29 30 
```

**常用选项：**

- `m` 以星期一作为一周的第一天
- `3` 显示前后三个月的日历
- `y` 显示一年的日历

```bash
[root@localhost ~]# cal -m
    November 2020   
Mo Tu We Th Fr Sa Su
                   1
 2  3  4  5  6  7  8
 9 10 11 12 13 14 15
16 17 18 19 20 21 22
23 24 25 26 27 28 29
30 
[root@localhost ~]# cal -m3
    October 2020          November 2020         December 2020   
Mo Tu We Th Fr Sa Su  Mo Tu We Th Fr Sa Su  Mo Tu We Th Fr Sa Su
          1  2  3  4                     1      1  2  3  4  5  6
 5  6  7  8  9 10 11   2  3  4  5  6  7  8   7  8  9 10 11 12 13
12 13 14 15 16 17 18   9 10 11 12 13 14 15  14 15 16 17 18 19 20
19 20 21 22 23 24 25  16 17 18 19 20 21 22  21 22 23 24 25 26 27
26 27 28 29 30 31     23 24 25 26 27 28 29  28 29 30 31         
                      30 
```

## `pwd`显示当前目录

`pwd`命令是 `print work directory`(打印工作目录)的缩写，会输出当前用户所在的目录

```bash
[root@localhost ~]# pwd
/root
```

## `wc`统计文本

`wc`命令的是 `Word Count`(字数)的缩写，这个命令能够计算一个文件内行数、单词数与字节数

语法：`wc -[参数][参数]...`

```bash
[root@localhost ~]# wc /etc/passwd
  46  104 2534 /etc/passwd
# 行数 词数 字节数
```

**常用参数：**

- `-l` 统计行数
- `-w` 统计单词数（以空格为界限）
- `-c` 统计字节数

```bash
[root@localhost ~]# wc -w /etc/passwd
104 /etc/passwd
# 统计单词数
[root@localhost ~]# wc -c /etc/passwd
2534 /etc/passwd
# 统计字节数
```

## 关机与重启

关机命令: `poweroff`, `shutdown -h now`, `init 0`(改变运行状态，不推荐使用)

重启命令: `reboot`, `shutdown -r now`, `init 6`(改变运行状态，不推荐使用)

```bash
# 延时关机或重启
[root@localhost ~]# shutdown -h 10:30
# 在10:30挂起系统并关机
[root@localhost ~]# shutdown -r 60
# 在60分后挂起系统并重启
```

**补充：**

切换运行级别: RHEL6引入(7、8也兼容，但是不推荐使用)

- `init 0` 关机
- `init 1` 单用户模式(修复模式/破解密码模式)
- `init 2` 多用户模式，字符界面(不支持网络)
- `init 3`完全多用户模式，字符界面(支持网络)
- `init 4` 安全模式
- `init 5`图形界面
- `init 6` 重启

## `ls`查看文档信息

`ls`命令是 `list files`(列出文件)的缩写，这个命令用于显示指定工作目录下之内容(列出目前工作目录所含之文件及子目录)

语法：`ls -[参数][参数]... /文档路径`

```bash
[root@localhost ~]# ls ./
anaconda-ks.cfg  Documents  initial-setup-ks.cfg  Pictures  Templates
Desktop          Downloads  Music                 Public    Videos
```

**在bash中 `ls`命令会用不同的颜色输出文档名：**

- 白色：文本文件
- 绿色：可执行文件
- 红色：压缩包、iso光盘镜像
- 蓝色：目录
- 蓝绿色：快捷方式

**常用选项：**

- `-l` 以长格式显示文档的详细属性信息
- `-d` 一般与 `-l`一起用，查看目录本身的详细属性信息
- `-h` 必须与 `-l`一起用，查看属性信息时以人类易读的单位显示大小
- `-a` 显示文件名以.开头的隐藏文档

**补充：**

命令 `ll`实际上相当于是命令 `ls -l`

```bash
[root@localhost ~]# ll
total 8
-rw-------. 1 root root 1403 Nov 25 23:28 anaconda-ks.cfg
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Desktop
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Documents
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Downloads
-rw-r--r--. 1 root root 1558 Nov 25 23:31 initial-setup-ks.cfg
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Music
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Pictures
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Public
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Templates
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Videos
```

## `cd`切换工作目录

`cd`命令是 `change directory`(更改目录)的缩写，用于切换当前工作目录

语法：`cd /文档目录`

```bash
[root@localhost ~]# cd Desktop/
[root@localhost Desktop]# 
```

**常用用法：**

- `cd ..` 切换到上一级目录(..父目录，.当前目录)
- `cd -` 切换到上一次所在的目录
- `cd ~`或 `cd` 切换到当前用户的家目录

## `cp`复制文档到指定位置

`cp`命令是 `copy`(复制)的缩写，主要用于复制文件和目录

语法：`cp -[选项][选项]... /一个或多个文档路径 /目标文档路径`

```bash
[root@localhost ~]# cp 1.txt Desktop/
```

**常用选项：**

- `-r` 递归复制，除了复制目录本身还复制目录里的内容，复制目录时必须要有

**使用技巧：**

1. 询问是否覆盖同名文档
   在命令前加上 `\`
   `\cp -r /文档路径 /目标文档路径`
2. `cp`命令支持两个以上的参数
   `cp -r /文档路径1 /文档路径2 /文档路径3 /文档路径4 /文档路径5(目标文档路径)`
   永远将最后一个参数作为目标文档路径，会将前面输入的文档都复制到最后一个目录中
3. 与 `.`连用代表复制到当前目录下
   `cp 文档名 .`
4. 复制并重命名
   `cp /文档路径/文档名 /目标文档路径/新文档名`
   1. `/目标文档路径`内本来没有名为 `/新文档名`的文档
      将文档复制到 `/目标文档路径`并重命名为 `/新文档名`
   2. `/目标文档路径`里本来有一个名为 `/新文档名`的文件
      将文档复制到 `/目标文档路径`并重命名为 `/新文档名`，询问是否覆盖同名文件
   3. `/目标文档路径`里本来有一个名为 `/新文档名`的目录
      将文档复制到 `/目标文档路径/新文档名`这个目录下

## `mv`移动文档

`mv`命令是 `move`(移动)的缩写，用来为文件或目录改名、或将文件或目录移入其它位置。

语法：`mv /原文档路径 /目标路径`

**常用用法：重命名**

目标路径不变的移动就是重命名

```bash
[root@localhost ~]# mv 1.txt 2.txt
```

## `rm`删除文档

`rm`命令是 `remove`(移除)的缩写，用于删除一个文件或者目录

语法：`rm -[选项] /被删除文档路径`

```bash
[root@localhost ~]# rm 2.txt
```

**常用选项：**

- `-r` 递归删除，除了删除目录本身，还删除目录里的内容，删除目录时必须要有
- `-f` 强制删除，不提示（一般删除目录时需要使用）

```bash
[root@localhost ~]# rm -rf /*
# 千万别用，会死的(>д<)
```

## `cat`查看文件

`cat`命令是 `concatenate`(连接)的缩写，用于连接文件并打印到标准输出设备上。

语法：`cat -[选项] 文本[文件]`

常用选项：

- `-n` 显示文本行号

## `touch`创建文件

`touch`命令是“触碰”的意思，用于修改文件或者目录的时间属性，包括存取时间和更改时间。若文件不存在，系统会建立一个新的文件。

## `mkdir`创建目录

`mkdir`命令是 `make directory`(生成目录)的缩写，用于创建目录。

**常用选项：**

- `-p` 递归创建子文目录

## 获取帮助

有两种方式可以获取命令的帮助信息

- help帮助
  在命令后添加参数 `--help`或是 `-h`就可以获取关于命令的说明
- man手册
  语法为 `man [命令名]`，即可查看man手册中的信息

## man手册的使用

`man`命令是 `Manual pages`的缩写，是在Unix或类Unix操作系统在线软件文档的一种普遍的形式。

滚动屏幕：向下 `j` `↓` `Page Down`，向上 `k` `↑` `Page Up`

搜索模式：`/关键词`,回车，`n`下一个，`N`上一个

放大字体：`Ctrl Shift +`

缩小字体：`Ctrl -`

按 `q`退出

## vim的使用

命令模式中的快捷键

1.移动光标：

- 键盘上下左右键、Home键、End键

2.行间跳转：

- 到全文的第一行(`1G`或`gg`)、到全文的最后一行(`G`)、到全文的第10行(`10G`)

3.复制、粘贴：

- 复制1行（`yy`）、复制3行（`3yy`）

- 粘贴到光标之后（小写`p`）

- 粘贴到光标之后 (大写`P`)


4.删除(实际为剪切)：

- 删除单个字符(`x`或`Delete`)

- 删除到行首(`d^`)、删除到行尾(`d$`)

- 删除1行（`dd`）、删除3行（`3dd`）


5.查找关键词：

- 搜索（`/word`）切换结果（`n`、`N`）


6.撤销操作:

- 撤销最近的一次操作(`u`)

- 取消前一次撤销操作`Ctrl+r`