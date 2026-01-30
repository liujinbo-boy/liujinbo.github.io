 **Linux 服务器维护常用命令手册**，便于快速查阅：

---

## 一、用户与用户组管理

▶ **修改用户家目录**  
`sudo usermod -d /data/user1 -m user1`  
→ `-m` 自动创建新目录并迁移原数据

▶ **修改系统用户密码**  
`sudo passwd user1`

▶ **Samba 用户初始化**  
`sudo smbpasswd -a user1`  
→ 将系统用户加入 Samba 数据库并设初始密码

▶ **修改 Samba 用户密码**  
`sudo smbpasswd user1`  
→ 立即生效

▶ **赋予 sudo 权限（Debian/Ubuntu 友好方式）**  
`sudo adduser user1 sudo`

▶ **赋予 sudo 权限（通用方式）**  
`sudo usermod -aG sudo user1`  
→ `-aG` 追加到组，不覆盖原有组

▶ **创建新用户（自动创建家目录）**  
`sudo useradd -m -d /home/user2 user2`

▶ **删除用户及家目录**  
`sudo userdel -r user2`  
→ `-r` 避免残留文件

▶ **查看用户 UID/GID/所属组**  
`id user1`

▶ **切换用户（加载环境变量）**  
`su - user1`

▶ **查看用户所属所有组**  
`groups user1`

▶ **创建用户组**  
`sudo groupadd dev_group`

▶ **修改用户主组**  
`sudo usermod -g dev_group user1`  
→ 覆盖原有主组

---

## 二、文件权限与归属管理

▶ **递归修改属主和属组**  
`sudo chown -R ljb:ljb /media/bsj/HDD2/hw/ljb`

▶ **递归修改权限（755）**  
`sudo chmod -R 755 /media/bsj/HDD2/hw/ljb`  
→ 755 = 属主 rwx，组/其他 rx

▶ **单独修改属组**  
`sudo chgrp dev_group /data/file.txt`

▶ **添加 SUID 权限**  
`sudo chmod u+s /usr/bin/passwd`  
→ 执行时继承文件属主权限

▶ **添加 SGID 权限（目录）**  
`sudo chmod g+s /data/dev`  
→ 新文件自动继承目录属组

▶ **查看 ACL 精细权限**  
`getfacl /data/file.txt`

▶ **设置 ACL 权限**  
`sudo setfacl -m u:user1:rwx /data/file.txt`  
→ 为指定用户授权

---

## 三、文件操作与管理

▶ **按名称查找文件**  
`find /data -name "*.log"`

▶ **按属主查找文件**  
`find /opt -user user1`

▶ **查看小文件内容**  
`cat /etc/hosts`

▶ **分页查看大文件**  
`less /var/log/syslog`  
→ 支持搜索、翻页

▶ **实时跟踪日志**  
`tail -f /var/log/nginx/access.log`  
→ 核心运维命令

▶ **查看文件前 N 行**  
`head -n 10 /etc/passwd`

▶ **递归复制目录**  
`cp -r /data/backup /tmp/`

▶ **移动/重命名文件**  
`mv /data/file.txt /data/new.txt`

▶ **递归创建多级目录**  
`mkdir -p /data/dev/log`

▶ **强制递归删除（⚠️ 谨慎）**  
`sudo rm -rf /tmp/unused`

▶ **压缩为 gzip 格式**  
`tar -zcvf data.tar.gz /data/dev`

▶ **解压到指定目录**  
`tar -zxvf data.tar.gz -C /tmp/`

▶ **压缩为 zip 格式**  
`zip -r data.zip /data/dev`

▶ **解压 zip 文件**  
`unzip data.zip -d /tmp/`

---

## 四、存储与挂载管理

▶ **查看磁盘空间**  
`df -h`  
→ `-h` 人性化单位（GB/MB）

▶ **查看目录总大小**  
`du -sh /data`

▶ **查看内存/swap 使用**  
`free -h`

▶ **查看 swap 分区状态**  
`swapon -s`

▶ **查看分区 UUID**  
`sudo blkid`  
→ 用于 fstab 永久挂载

▶ **挂载硬盘分区**  
`sudo mount /dev/sdb1 /mnt/usb`

▶ **强制卸载挂载点**  
`sudo umount -f /mnt/nfs_mount`  
→ 适用于 NFS 卡死场景

▶ **挂载 NFS 共享**  
`sudo mount -t nfs -o nolock 192.168.100.4:/opt /opt`

▶ **查看所有挂载**  
`mount`

▶ **配置开机自动挂载**  
`sudo vim /etc/fstab`  
→ 添加：`UUID=xxx /data ext4 defaults 0 2`

▶ **验证 fstab 配置**  
`sudo mount -a`

▶ **创建 swap 文件（应急）**  
`sudo fallocate -l 10G /swapfile`  
→ 后续执行：`chmod 600` → `mkswap` → `swapon`

---

## 五、系统基础运维

▶ **切换到 root 环境**  
`sudo -i`  
→ 加载完整 root 环境变量

▶ **更新软件包索引（Debian/Ubuntu）**  
`sudo apt update`

▶ **升级已安装软件包**  
`sudo apt upgrade -y`

▶ **安装软件包**  
`sudo apt install nginx`  
`sudo yum install nginx -y`（RHEL/CentOS）

▶ **卸载软件（保留配置）**  
`sudo apt remove nginx`

▶ **彻底卸载（删除配置）**  
`sudo apt purge nginx`

▶ **安装本地 deb 包**  
`sudo dpkg -i nginx_1.24.0.deb`

▶ **重启服务器**  
`sudo reboot`

▶ **立即关机**  
`sudo shutdown -h now`

▶ **定时关机**  
`sudo shutdown -h 20:00`

▶ **查看内核与架构**  
`uname -a`

▶ **查看/设置主机名**  
`hostnamectl`（推荐）  
`sudo hostname server-01`（临时）

▶ **查看发行版信息**  
`lsb_release -a`（Debian/Ubuntu）

▶ **查看当前登录用户**  
`who`

▶ **查看登录历史**  
`last`

▶ **查看命令历史**  
`history`

▶ **清理缓存释放内存（⚠️ 仅诊断用）**  
`sudo sync && echo 3 > /proc/sys/vm/drop_caches`

---

## 六、网络管理

▶ **查看网卡信息**  
`ip a`  
→ 替代 ifconfig

▶ **测试连通性**  
`ping -c 4 192.168.100.4`

▶ **路由追踪**  
`traceroute www.baidu.com`

▶ **实时路由监控**  
`mtr www.baidu.com`  
→ ping + traceroute 合体

▶ **查看端口监听（推荐）**  
`ss -tulnp`

▶ **查看端口监听（传统）**  
`netstat -tulnp`

▶ **查看端口占用进程**  
`lsof -i :80`

▶ **启用 UFW 防火墙**  
`sudo ufw enable`

▶ **开放 SSH 端口**  
`sudo ufw allow 22/tcp`

▶ **查看防火墙规则**  
`sudo ufw status`  
`sudo iptables -L -n`（iptables）

▶ **远程登录**  
`ssh user1@192.168.100.4`

▶ **上传文件**  
`scp /data/file.txt user1@192.168.100.4:/data/`

▶ **下载文件**  
`scp user1@192.168.100.4:/data/file.txt /tmp/`

---

## 七、进程管理

▶ **查看指定进程**  
`ps -ef | grep nginx`

▶ **查看进程资源占用**  
`ps aux`

▶ **实时监控进程**  
`top`  
→ 按 `P` 按 CPU 排序，按 `M` 按内存排序

▶ **增强版监控（需安装）**  
`htop`

▶ **强制终止进程**  
`kill -9 1234`

▶ **终止同名进程**  
`killall nginx`  
`pkill nginx`（支持模糊匹配）

▶ **查看端口占用进程**  
`lsof -i :80`

### 