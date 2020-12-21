# 碎片化的RHEL8知识2

## 通配符

通配符:针对不确定的文档名称，以特殊字符表示

`*`代表多个字符，`?`代表单个字符

`[a-z]/[0-9]`匹配在单个数字或字母之间的字符

`{1,2,a,z}`匹配在这个括号内的字符

## `|`管道符

`|`：将管道符前的命令结果交给后面的命令再次处理

```bash
head -10 [txt]
# 查看文本前10行
tail -5 [txt]
# 查看文本后5行
head -20 [txt]|tail -16
# 输出前20行，在输出前面文本的后16行，即5-20行
```

## `grap`查找行

`grep`命令全称是 `Globally search a Regular Expression and Print`(全局搜索正则表达式并输出)

语法：`grep [选项] 过滤条件 文本文件`

**常用选项：**

- `-v` 取反
- `-i` 忽略大小写
- `-n` 显示行号
- `-c` 统计行数

常用过滤条件

- `关键词` 包含关键词
- `^关键词` 以关键词开头
- `关键词$` 以关键词结尾
- `^$` 匹配空行(与 `-v`一起使用排除空行)

## 重定向

标准输入(STDIN 文件描述为 0)：默认从键盘输入

标准输出(STDOUT 文件描述符为 1)：默认输出到屏幕

错误输出(STDERR 文件描述符为2)：默认输出到屏幕

**输入重定向**

- `命令 < 文件` 将文件作为命令的标准输入
- `命令 << 分界符` 从标准输入中读入，直到遇见分界符才停止

**输出重定向**

- `>` 覆盖重定向，`>>`追加重定向
- `2>` 只重定向错误输出
- `&>` 重定向标准输出和错位输出

## `find`搜索文件

`find`命令就是查找的意思

**语法：**`find /文档路径 条件1 选项 条件2 ...`

**常用选项：**

- `-a` and 逻辑和，代表 `-a`前后的条件同时成立才匹配
- `-o` or 逻辑或，代表 `-o`前后的条件只要成立其中一个就匹配

> *默认状态 `-a`，默认多个查找条件*

- `-type` 按照文档类型查找，文件 `f`、目录 `d`、快捷方式 `l`
- `-name` 按照文档名查找，支持通配符
- `-iname` 功能同上但是忽略大小写
- `-size` 按照文件的大小进行查找，`+`表示大于指定大小，`-`则表示小于指定大小，可选单位 `k`、`M`、`G`、`T`

> 注意 `-size -1M`和 `-size -1024k`的区别，前者仅匹配空文件

- `-user` 按照文档的所有者查找文档
- `-group` 按照文档的所属组查找文档
- `-maxdepth` 限制递归查找的层数，不能单独使用
- `-mtime` 按修改时间进行查找，`+2`查找两天前
- `-exec` 对前面的搜索结果执行一条命令(类似管道)

```bash
[root@localhost ~]# find /etc/ -maxdepth 1 -type f -exec cp {} /mnt/    \;
# 在/etc/目录下查找 查找一层 文档类型为文件 
# 将搜索到的文件复制到/mnt/目录下
```

## 管理用户与组

`user`用户：登录到系统的账号

`group`组：把想要具有相同权限的用户放到相同的组，批量管理

**用户的分类**

- 管理员用户：root，拥有最高的权限
- 普通用户
- 系统用户：系统自带或通过安装软件自动生成的用户

**组分类**

- 基本组：随着用户创建会自动生成并自动加入一个和用户名相同的组
- 附加(从属)组：除了基本组，用户还可以加入其他组(包含其他用户的基本组)

**uid：**类似身份证号，是唯一标识这个系统里一个用户的编号

- 0：root
- 1-999：系统用户
- 1000-65535：普通用户

用户家目录

- 管理员(root)家目录：/root
- 普通目录：/home/用户名，创建了用户以后就自动创建

### `useradd`添加(创建)用户

语法：`useradd [选项] 用户名`

**常用选项：**

- `-u` 创建用户时指定用户的uid号
- `-d` 创建用户时指定家目录路径
- `-s` 创建用户时指定用户使用的登录shell
- `-G` 创建用户时为其指定一个附加组

### 修改用户的密码

#### 1#交互式修改密码

```bash
[root@localhost ~]# passwd zgyd
Changing password for user zgyd.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
```

#### 2#非交互式修改密码

```bash
[root@localhost ~]# echo 1919810|passwd --stdin zgyd 
Changing password for user zgyd.
passwd: all authentication tokens updated successfully.
```

## `su`临时切换用户

`su`命令的意思是 `swith user`(切换用户)，用于变更为其他用户进行操作，除了 root以外，都需要输入使用者的密码才能登录

**语法：**`su 用户名`

## 用户的信息与密码

 `/etc/passwd`存放有所有用户的基本信息

每一个用户占用一行

```bash
[root@localhost ~]# head -1 /etc/passwd
root:x:0:0:root:/root:/bin/bash
# 用户名:密码占位符:UID:GID:用户描述信息:用户家目录:解释器
```

 `/etc/shadow`文件中存放着用户相关的密码信息

```bash
[root@localhost ~]# head -1 /etc/shadow
root:$6$baLQ7lHGan5N4ilU$E0gqjrOW5W7ta9hFTLvEbK0gVPOXTAPWShtagFt0diyxvGWbechaXFW6dD7zcIdR9A289a2YfiScOeLHALynx.::0:99999:7:::
# 用户名:密码加密字符串:上次密码修改时间的时间戳
```

## `usermod`修改用户属性

`usermod`的意思是 `user modify`(修改用户账号)，主要用于修改用户的属性，例uid、附加组、家目录、解释器等

**语法：**`usermod [选项] 用户名`

**常用选项：**

- `-u` 修改uid
- `-d` 修改家目录路径
- `-g` 修改所属组
- `-G` 修改附加组
- `-s` 修改登录 shell

也可以手动直接修改 `etc/passwd`文件，更改用户的相关信息

## `userdel`删除用户

`userdel`的意思是 `user delete`(删除用户)，主要用于删除一个用户

**语法：**`userdel [-r] 用户名`

**常用选项：**

- `-r` 删除用户的家目录

*尽量不要使用，家目录数据还是比较宝贵的*

## 用户组管理

`/etc/group`中存放的是组的相关信息

```bash
[root@localhost ~]# grep zgyd /etc/group
zgyd:x:10086:
# 组名:组的密码:组的gid标识:组成员列表(可空)
```

## `groupadd`新建组

**语法：**`groupadd [选项] 组名`

**常用选项：**

- `-g` 指定组id

## `gpasswd`修改组成员

**语法：**`gpasswd [选项] 用户名 组名`

**常用选项：**

- `-a` 添加用户到组
- `-d` 从组中删除用户

组管理操作不是立即生效的，需要重新登录才会生效

## `groupdel`删除组

**语法：**`groupdel 组名`

## 权限管理

### Linux文档权限

**基本权限类别：**

- 读取 `r` --`read`允许查看
- 写入 `w` --`write`允许修改
- 可执行 `x`-- `execute`允许运行和切换

**权限的使用对象：**

- 所有者 `u`-- `user`拥有此文件/目录的用户
- 所属组 `g`-- `group`拥有此文件/目录的组
- 其他用户 `o`-- `other`除所有者，所有组以外的用户

**对于文本文件的权限：**

- `r`-- 可以使用`cat`等命令查看
- `w`-- 可以使用 `vim`等编辑器写入
- `x`-- 可以在 Shell中运行

**对于目录的权限：**

- `r`-- 可以使用`ls`命令查看
- `w`-- 可以使用 `rm`、`mv`、`cp`、`mkdir`等命令更改目录内的内容(不包括目录本身的属性)
- `x`-- 可以在使用 `cd`命令切换到该目录

*如果父目录没有权限，那子目录和子文件有权限也无意义*

### 查看权限

查看文档权限可以使用下列命令

- `ls -l` 查看工作目录内的全部权限
- `ls -ld` 查看指定的目录的权限

权限的格式：

文档类型-所有者权限-所属组权限-其他人权限 链接数 所有者 所属组

- 开头为 `d`为目录
- 开头为 `-`为文本文件
- 开头为 `l`为快捷方式

```bash
# ll命令等同于 ls -l命令
[root@localhost ~]# ll
total 12
-rw-r--r--. 1 root root   14 Nov 27 00:12 2.txt
-rw-------. 1 root root 1403 Nov 25 23:28 anaconda-ks.cfg
drwxr-xr-x. 2 root root   19 Nov 27 00:12 Desktop
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Documents
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Downloads
-rw-r--r--. 1 root root 1558 Nov 25 23:31 initial-setup-ks.cfg
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Music
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Pictures
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Public
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Templates
drwxr-xr-x. 2 root root    6 Nov 25 23:31 Videos
[root@localhost ~]# ll -d
dr-xr-x---. 15 root root 4096 Dec  1 06:12 .
```

### `chmod`设置权限

`chmod`命令的意思是 `change file mode bits`(改变文件权限标识)

**语法：**`chmod 归属关系 [+、-、=] 权限类型 文档`

```bash
[root@localhost ~]# chmod o+w 1
# 将其他人添加读权限
[root@localhost ~]# chmod u-w 1
# 将所有者移除写权限
[root@localhost ~]# chmod g=--- 1
# 将所属组可的权限设置为空(---表示没有读写执行权限)
[root@localhost ~]# chmod u=r 1
# 将所有者权限设置为只读
[root@localhost ~]# chmod ugo=rwx 1
# 将所有权限设置为可读写执行
[root@localhost ~]# chmod g-w,o-x 1
# 删除组用户的写权限，删除其他人的执行权限
[root@localhost ~]# chmod 774 1
# 将权限设置为 rwxrwxr-x
```

**常用选项：**

- `-R` 递归设置(包含目录本身以及目录下所有文档一起设置权限)

***注意：只有 root才能运行 `chmod`命令***

### `chown`设置文档归属

`chown`命令表示 `change file owner and group`(变更文档的所有者和组)

**常用选项：**

- `-R` 递归设置(包含目录本身以及目录下所有文档一起设置权限)

默认情况下文档的所有者和所属组分别是：文件的创建者及其基本组

如果是 root用户执行 `cp`命令，那么复制品的所有者和所属组会变成 root用户及 root组

如果想复制后保持原归属关系可以使用 `cp -p`命令

```bash
[root@localhost ~]# chown -R zgyd 1
# 修改所有者
[root@localhost ~]# chown -R root:root 1
# 修改所属组
[root@localhost ~]# chown -R root:root 1
# 修改所有者和所属组
```

### 特殊权限

#### 1# Set GID

**命令：**`chmod g+s /文档路径` & `chmod g-s /文档目录`

附加在所属组的执行权限 `x`(执行)位上，所属组的执行标识会变成 `s`或 `S`

`s`代表所属组的执行权限为 `x`，`S`代表所属组的执行权限为 `-`

**适用范围：**目录

**功能：**Set GID可以使目录下新增的文档新建时自动设置与父目录相同的所属组，也就是让新建的子文档自动继承父目录的所属组

#### 2#Set UID

**命令：**`chmod u+s /文档目录`& `chmod u-s /文档目录`

附加在所有者的执行权限 `x`(执行)位上，所有者的执行标识会变成 `s`或 `S`

`s`代表所有者的执行权限为 `x`，`S`代表所有者的执行权限为 `-`

**适用范围：**所有可执行文件

**功能：**可以让使用者具有文件所有者的身份以及部分权限，让用户暂时拥有所有者的权限

#### 3#Sticky Bit

**命令：**`chmod o+t /文档目录 `& `chmod o-t /文档目录 `

附加在其他用户的执行权限 `x`(执行)位上，其他用户的执行标识会变成 `t`或 `T`

`t`代表设置附加权限前其他用户执行权限为 `x`，`T`代表设置附加权限前其他用户的执行权限为 `-`

**适用范围：**适用于开放了 `w`权限的目录

**功能：**可以防止用户滥用 `w`权限(禁止用户操作别人的文件)，只有文档的所有者才能操作自己的文档，其他人无法操作

### acl 策略

acl 策略：访问控制列表，

**功能：**能够针对个别用户，个别组设置独立的权限

**优势：**可以实现更精确的权限控制

#### `setfacl`修改访问控制列表

`setfacl`意为 `set file access control lists`(设置文件访问控制列表)

```bash
setfacl -m u:zgyd:rwx /root/1.txt
# 将文件的cal中用户zgyd的权限修改为rwx
setfacl -R -m g:zgyd:r /root/114514
# 将目录的cal中组zgyd的权限递归修改为r
setfacl -x u:zgyd:x /root/1.txt
# 删除文件的cal中用户zgyd的x权限
setfacl -R -b /root/114514
# 将目录的cal递归删除 
```

**常用选项：**

- `-m` modify 表示调整、修改
- `-R` 递归设置权限
- `-x` 删除用户或组的相关权限
- `-b` 删除所有acl

#### `getfac`查看文档acl权限

`getfacl`意为 `get file access control lists`(获取文件访问控制列表)

