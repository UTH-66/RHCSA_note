# RHCSA

关于RHCSA题目的解析与说明

## 虚拟机1需要执行的任务

### 1.网络配置

给虚拟机1设置以下网络参数  
主机名：`server0.example.com`  
IP地址：`172.25.0.11`  
子网掩码：`255.255.255.0`  
网关：`172.25.0.254`  
名称服务器：`172.25.254.254`

<details>
<summary>Open</summary>
<pre>
[root@localhost ~]# <code>nmcli connection modify 'Wired connection 1' ipv4.method manual ipv4.addresses 172.25.0.11/24 ipv4.gateway 172.25.0.254 ipv4.dns 172.25.254.254 connection.autoconnect yes</code>
[root@localhost ~]# <code>nmcli connection up 'Wired connection 1'</code>
[root@localhost ~]# <code>hostnamectl set-hostname server0.example.com</code>
</pre>
</details>

### 2.配置yum

yum存储库已可以从  
`http://classroom.example.com/BaseOS`  
`http://classroom.example.com/AppStream`  
使用配置您的系统，以将这些位置用作默认存储库

<details>
<summary>Open</summary>
<pre>
root@server0 ~]# <code>vi /etc/yum.repo.d/base.repo</code>
<code>#添加下面的语句
[BaseOS]
name=this is BaseOS.repo
baseurl=http://classroom.example.com/BaseOS
enabled=1
gpgcheck=0</code>
[root@server0 ~]# <code>cp /etc/yum.repo.d/{base.repo,app.repo}</code>
[root@server0 ~]# <code>vi /etc/yum.repo.d/app.repo</code>
<code>#修改为
[AppStream]
name=this is AppStream.repo
baseurl=http://classroom.example.com/AppStream
enabled=1
gpgcheck=0</code>
[root@server0 ~]# <code>yum clean all</code>
[root@server0 ~]# <code>yum repolist</code>
</pre>
</details>

<details>
<summary>不推荐的方法</summary>
<pre>
<p>注意:像这样的方法需要在VM中打开浏览器访问classroom的yum库查找包</p>
root@server0 ~]# <code>rpm -ivh http://classroom.example.com/BaseOS/Packages/dnf-utils-4.0.2.2-3.el8.noarch.rpm</code>
[root@server0 ~]# <code>yum-config-manager --add-repo http://classroom.example.com/BaseOS</code>
[root@server0 ~]# <code>yum-config-manager --add-repo http://classroom.example.com/AppStream</code>
[root@server0 ~]# <code>vi /etc/yum.conf</code>
[root@server0 ~]# <code>yum clean all</code>
[root@server0 ~]# <code>yum repolist</code>
</pre>
</details>

### 3.配置SELinux

非标准端口 82 上运行的 Web 服务器在提供内容时遇到问题。根据需要调试并解决问题，使其满足以下条件： 

1. 系统上的 Web 服务器能够提供 `/var/www/html` 中所有现有的 HTML 文件
2. Web 服务器在端口 82 上提供此内容
3. Web 服务器在系统启动时自动启动

（注：不要删除或以其他方式改动现有的 文件内容）

<details>
<summary>Open</summary>
<pre>
[root@server0 ~]# <code>semanage port -a -t http_port_t -p tcp 82</code>
[root@server0 ~]# <code>chcon --reference=/var/www/html /var/www/html/file1</code>
[root@server0 ~]# <code>ls -l /var/www/html/file1 -Z</code>
[root@server0 ~]# <code>systemctl restart httpd</code>
[root@server0 ~]# <code>systemctl enable httpd</code>
</pre>
</details>

### 4. 配置用户账户

创建下列用户、组和组成员资格

1. 名为 sysmgrs的组
2. 用户 Natasha ，作为次要组从属于 sysmgrs
3. 用户 Harry ，作为次要组还从属于 sysmgrs
4. 用户 Sarah ，无权访问系统上的交互式 shell 且不是 sysmgrs 的成员
5. Natasha 、 Harry 和 Sarah 的密码应当都是 123

<details>
<summary>Open</summary>
<pre>
[root@server0 ~]# <code>groupadd sysmgrs</code>
[root@server0 ~]# <code>useradd -G sysmgrs Natasha</code>
[root@server0 ~]# <code>useradd -G sysmgrs Harry</code>
[root@server0 ~]# <code>useradd -s /bin/false Sarah</code>
[root@server0 ~]# <code>echo 123|passwd --stdin Natasha</code>
[root@server0 ~]# <code>echo 123|passwd --stdin Harry</code>
[root@server0 ~]# <code>echo 123|passwd --stdin Sarah</code>
</pre>
</details>

### 5. 配置 `cron`计划任务

配置 `cron`作业，该作业在本地时间每周三 14:23 运行并执行以下命令：

`logger "EX200 in progress"`，以用户 Natasha身份运行

<details>
<summary>Open</summary>
<pre>
[root@server ~]# <code>crontab -e -u Natasha</code>
<code># 添加下面这句
23 14 * * 3 logger "EX200 in progress"</code>
</pre>
</details>

### 6.创建目录

1. 创建目录 `/home/managers`
2. `/home/managers`的组用权是 sysmgrs
3. 目录应当可被 sysmgrs的成员读取、写入和访问，但任何其他用户不具这些权限。
4. `/home/managers` 中创建的文件自动将组所有权设置到 sysmgrs 组

<details>
<summary>Open</summary>
<pre>
[root@server ~]# <code>mkdir /home/managers</code>
[root@server ~]# <code>chown :sysmgrs /home/managers</code>
[root@server ~]# <code>chmod g=rwx,o=- /home/managers</code>
[root@server ~]# <code>chmod g+s /home/managers</code>
</code></pre>
</details>

### 7.配置NTP时间同步

配置您的系统，使其成为 `classroom.example.com`的 NTP客户端。

<details>
<summary>Open</summary>
<pre>
[root@server ~]# <code>systemctl status chronyd</code>
[root@server ~]# <code>vim /etc/chrony.conf</code>
<code># 添加下面这行
server classroom.exmple.com iburst</code>
[root@server ~]# <code>systemctl restart chronyd</code>
</pre>
</details>

### 8. 配置`autofs`

配置`autofs` ，以按照如下所述自动挂载远程用户的主目录： 

1. 从 `classroom.example.com(172.25.0.254)` NFS 导出 /rhel8 到您的系统。此文件系统包含为用户user1 预配置的主目录 
2. user1 的主目录是 `classroom.example.com:/rhel8/user1`
3. user1 的主目录应自动挂载到本地 `/home/rhel8` 下的 `/home/rhel8/user1`
4. 主目录必须可供其用户写入
5. user1 的密码是 123

<details>
<summary>Open</summary>
<pre>
[root@server0 ~]# <code>yum -y install nfs-utils</code>
[root@server0 ~]# <code>yum -y install autofs</code>
[root@server0 ~]# <code>vim /etc/auto.master</code>
<code># 添加下面这行
/home/rhel8     /etc/auto.rule</code>
[root@server0 ~]# <code>vim /etc/auto.rule</code>
<code># 添加下面这行
user1  -rw   classroom.example.com:/rhel8/user1</code>
[root@server0 ~]# <code>systemctl enable --now autofs.service</code>
</pre>
</details>

### 9. 设置文件权限

将文件 `/etc/fstab`复制到 `/var/tmp/fstab` 。配置 `/var/tmp/fstab`的权限以满足如下条件：

1. 文件 `/var/tmp/fstab` 自 root 用户所有

2. 文件 `/var/tmp/fstab` 属于组 root

3. 文件 `/var/tmp/fstab` 应不能被任何人执行

4. 用户 Natasha能够读取和写入 `/var/tmp/fstab`

5. 用户 Harry无法写入或读取 `/var/tmp/fstab`

6. 所有其他用户（当前或未来）能够读取 `/var/tmp/fstab`

<details>
<summary>Open</summary>
<pre>
[root@server0 ~]# <code>cp /etc/fstab /var/tmp/fstab</code>
[root@server0 ~]# <code>setfacl -m u:Natasha:rw /var/tmp/fstab</code>
[root@server0 ~]# <code>setfacl -m u:Harry:- /var/tmp/fstab</code>
</pre>
</details>

### 10. 创建用户账户

1. 创建用户 user2，其用户 ID为 `3388`
2. 用户密码为 123

<details>
<summary>Open</summary>
<pre>
[root@server0 ~]# <code>useradd -u 3388 user2</code>
[root@server0 ~]# <code>echo 123 | passwd --stdin user2</code>
</pre>
</details>

### 11. 查找文件

查找Harry 所有的文件并将其复制到 `/root/dfiles`目录下

<details>
<summary>Open</summary>
<pre>
[root@server0 ~]# <code>mkdir /root/dfiles</code>
[root@server0 ~]# <code>find / -user Harry -type f -exec cp -a {} /root/dfiles \;</code>
</pre>
</details>

### 12. 查找字符串

查找文件 `/etc/passwd`中包含字符串 `re`的所有行。将所有这些行的副本按原始顺序放在文件 `/root/files`中

<details>
<summary>Open</summary>
<pre>
[root@server0 ~]# <code>grep re /etc/passwd > /root/files</code>
</pre>
</details>
### 13. 创建归档

创建一个名为 `/root/books.tar.gz`的 tar归档，其应包含 `/usr/local`的 tar归档，其应包含 `/usr/local`的内容。该 tar归 档必须使用 `gzip`进行压缩

<details>
<summary>Open</summary>
<pre>
[root@server0 ~]# <code>tar -cPzf /root/books.tar.gz /usr/local</code>
</pre>
</details>


## 虚拟机2需要执行的任务

### 1. 破解密码

将 `server1.example.com`的 root密码设置为 114514

<details>
<summary>Open</summary>
<pre>
开机内核选择页面按<code>e</code>键
在<code>linux</code>所在行的行尾加<code>rd.break console=tty0</code>
按快捷键<code>ctrl + x</code>
<code># 进入紧急救援模式后的操作</code>
switch_root:/# <code>mount -o remount,rw / /sysroot</code>
switch_root:/# <code>chroot /sysroot</code>
sh-4.4# <code>echo 114514|passwd --stdin root</code>
sh-4.4# <code>touch /.autorelabel</code>
sh-4.4# <code>exit</code>
switch_root:/# <code>reboot</code>
</pre>
</details>

### 2. 配置 yum仓库

yum存储库已可以从  
`http://classroom.example.com/BaseOS`  
`http://classroom.example.com/AppStream`  
使用配置您的系统，以将这些位置用作默认存储库

<details>
<summary>Open</summary>
<pre>
root@server0 ~]# <code>vi /etc/yum.repo.d/base.repo</code>
<code>#添加下面的语句
[BaseOS]
name=this is BaseOS.repo
baseurl=http://classroom.example.com/BaseOS
enabled=1
gpgcheck=0</code>
[root@server0 ~]# <code>cp /etc/yum.repo.d/{base.repo,app.repo}</code>
[root@server0 ~]# <code>vi /etc/yum.repo.d/app.repo</code>
<code>#修改为
[AppStream]
name=this is AppStream.repo
baseurl=http://classroom.example.com/AppStream
enabled=1
gpgcheck=0</code>
[root@server0 ~]# <code>yum clean all</code>
[root@server0 ~]# <code>yum repolist</code>
</pre>
</details>

### 3. 调整逻辑卷大小

将逻辑卷 vo及其文件系统的大小调整到 180 MiB确保文件系统内容保持不变。

*注：分区大小很少与请求的大小完全 相同，因此可以接受范围为 167 MiB 到 193 MiB 的大小*

<details>
<summary>Open</summary>
<pre>
[root@server1 ~]# <code>df -Th</code>
[root@server1 ~]# <code>vgs myvol</code>
<code># 如果vg不够大,就先扩容 #</code>
[root@server1 ~]# <code>lkblk</code>
[root@server1 ~]# <code>fdisk</code>
[root@server1 ~]# <code>pvcreate /dev/sdb2</code>
[root@server1 ~]# <code>vgextend myvol /dev/sdb2</code>
[root@server1 ~]# <code>vgs</code>
<code>######################</code>
[root@server1 ~]# <code>lvextend -L 180M /dev/myvol/vo</code>
[root@server1 ~]# <code>resize2fs /dev/myvol/vo</code>
</pre>
</details>

### 4. 添加交换分区

向您的系统添加一个额外的交换分区 567MiB 。

- 交换分区应在系统启动时自动挂载
- 不要删除或以任何方式改动系统上 的任何现有交换分区

<details>
<summary>Open</summary>
<pre>
[root@server1 ~]# <code>lsblk</code>
[root@server1 ~]# <code>fdisk /dev/vdb</code>
[root@server1 ~]# <code>mkswap /dev/vdb2</code>
[root@server1 ~]# <code>vim /dev/fstab</code>
<code>#添加下面这条语句
/dev/vdb2	swap	swap	defaults	0 0</code>
[root@server1 ~]# <code>swapon -a</code>
</pre>
</details>

### 5. 创建逻辑卷

1. 逻辑卷取名为 np，属于 npgroup卷组，大小为 45 个 PE
2. npgroup卷组中逻辑卷的 PE大小应当为 20 MiB
3. 使用 ext3文件系统格式化新逻辑卷。该逻辑卷应在系统启动时自动挂载到 `/mnt/np`下

<details>
<summary>Open</summary>
<pre>
[root@server1 ~]# <code>lsblk</code>
[root@server1 ~]# <code>fdisk /dev/vdb</code>
[root@server1 ~]# <code>pvcreate /dev/vdb3</code>
[root@server1 ~]# <code>vgcreate -s 20M npgroup /dev/vdb3</code>
[root@server1 ~]# <code>lvcreate -l 45 -n np npgroup</code>
[root@server1 ~]# <code>mkfs.ext3 /dev/npgroup/np</code>
[root@server1 ~]# <code>mkdir /mnt/np</code>
[root@server1 ~]# <code>vim /etc/fstab</code>
<code># 添加下面这行
/dev/npgroup/np            /mnt/np                 ext3	defaults	1 2</code>
[root@server1 ~]# <code>mount -a /mnt/np</code>
</pre>
</details>

### 6. 创建VDO卷

根据如下要求，创建新的 VDO 卷：

1. 使用未分区的磁盘
2. 该卷的名称为 vdoname
3. 该卷的逻辑大小为 80G
4. 该卷使用 xfs文件系统格式化
5. 该卷挂载到 `/vbark`下

<details>
<summary>Open</summary>
<pre>
[root@server1 ~]# <code>lsblk</code>
[root@server1 ~]# <code>yum install vdo</code>
[root@server1 ~]# <code>vdo create --name=vdoname --device=/dev/vdc --vdoLogicalSize=80G</code>
[root@server1 ~]# <code>mkfs.xfs -K /dev/mapper/vdoname</code>
[root@server1 ~]# <code>udevadm settle</code>
[root@server1 ~]# <code>mkdir /vbark</code>
[root@server1 ~]# <code>vim /etc/fstab</code>
<code># 添加下面这行
/dev/mapper/vdoname /vbark xfs defaults,x-systemd.requires=vdo.service 0 0</code>
[root@server1 ~]# <code>mount -a</code>
</pre>
</details>

### 7. 配置系统调优

系统选择建议的 tuned 配置集并将它设为默认设置 

<details>
<summary>Open</summary>
<pre>
[root@server1 ~]# <code>tuned-adm recommend</code>
[root@server1 ~]# <code>tuned-adm profile virtual-guest</code>
</pre>
</details>
