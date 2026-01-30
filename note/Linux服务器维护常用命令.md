# Linux 服务器维护常用命令词典

### 一、用户与用户组管理

| 命令/语法                          | 说明                                                  | 示例                                                       |
| ---------------------------------- | ----------------------------------------------------- | ---------------------------------------------------------- |
| `sudo usermod -d 新目录 -m 用户名` | 修改用户家目录，-m 自动创建新目录并迁移原数据（必加） | `sudo usermod -d /data/user1 -m user1`                     |
| `sudo passwd 用户名`               | 修改系统用户密码（需提权）                            | `sudo passwd user1`                                        |
| `sudo smbpasswd -a 用户名`         | 将已有系统用户加入Samba数据库并设置初始密码           | `sudo smbpasswd -a user1`                                  |
| `sudo smbpasswd 用户名`            | 修改已有Samba用户密码（立即生效）                     | `sudo smbpasswd user1`                                     |
| `sudo adduser 用户名 sudo`         | 将用户加入sudo组（赋予sudo权限，友好型）              | `sudo adduser user1 sudo`                                  |
| `sudo usermod -aG sudo 用户名`     | 将用户追加到sudo组（不覆盖原有组，通用型）            | `sudo usermod -aG sudo user1`                              |
| `sudo useradd 用户名`              | 创建新系统用户（基础版，需手动配置家目录等）          | （-m自动创建家目录）`sudo useradd -m -d /home/user2 user2` |
| `sudo userdel -r 用户名`           | 删除用户并同步删除家目录（-r 避免残留文件）           | `sudo userdel -r user2`                                    |
| `id 用户名`                        | 查看用户UID、GID及所属所有组信息                      | `id user1`                                                 |
| `su 用户名`                        | 切换到指定用户环境（- 同步加载用户环境变量）          | `su user1`                                                 |
| `groups 用户名`                    | 查看用户所属的所有用户组                              | `groups user1`                                             |
| `sudo groupadd 组名`               | 创建新用户组                                          | `sudo groupadd dev_group`                                  |
| `sudo usermod -g 主组 用户名`      | 修改用户的主组（覆盖原有主组）                        | `sudo usermod -g dev_group user1`                          |

### 二、文件权限与归属管理

| 命令/语法                           | 说明                                                  | 示例                                           |
| ----------------------------------- | ----------------------------------------------------- | ---------------------------------------------- |
| `sudo chown -R 属主:属组 目标路径`  | 递归修改文件/目录的属主和属组                         | `sudo chown -R ljb:ljb /media/bsj/HDD2/hw/ljb` |
| `sudo chmod -R 权限值 目标路径`     | 递归修改文件/目录权限（755=属主读写执行，其他读执行） | `sudo chmod -R 755 /media/bsj/HDD2/hw/ljb`     |
| `chgrp 属组 目标路径`               | 单独修改文件/目录的属组                               | `sudo chgrp dev_group /data/file.txt`          |
| `chmod u+s 目标文件`                | 给文件添加SUID权限（执行时继承文件属主权限）          | `sudo chmod u+s /usr/bin/passwd`               |
| `chmod g+s 目标目录`                | 给目录添加SGID权限（目录下新文件继承目录属组）        | `sudo chmod g+s /data/dev`                     |
| `getfacl 目标路径`                  | 查看文件/目录的ACL权限（精细权限控制）                | `getfacl /data/file.txt`                       |
| `setfacl -m u:用户名:权限 目标路径` | 给指定用户添加ACL权限                                 | `sudo setfacl -m u:user1:rwx /data/file.txt`   |

### 三、文件操作与管理

| 命令/语法                               | 说明                                      | 示例                                |
| --------------------------------------- | ----------------------------------------- | ----------------------------------- |
| `find 查找路径 -name 文件名`            | 按名称查找文件/目录                       | `find /data -name "*.log"`          |
| `find 查找路径 -user 用户名`            | 按属主查找文件/目录                       | `find /opt -user user1`             |
| `cat 文件名`                            | 查看文件全部内容（适合小文件）            | `cat /etc/hosts`                    |
| `less 文件名`                           | 分页查看大文件（支持上下翻页、搜索）      | `less /var/log/syslog`              |
| `tail -f 日志文件`                      | 实时跟踪日志文件更新（核心运维命令）      | `tail -f /var/log/nginx/access.log` |
| `head -n 行数 文件名`                   | 查看文件前N行内容                         | `head -n 10 /etc/passwd`            |
| `cp -r 源路径 目标路径`                 | 递归复制目录/文件（-r 用于目录）          | `cp -r /data/backup /tmp/`          |
| `mv 源路径 目标路径`                    | 移动/重命名文件/目录                      | `mv /data/file.txt /data/new.txt`   |
| `mkdir -p 目录路径`                     | 递归创建多级目录（-p 避免层级不存在报错） | `mkdir -p /data/dev/log`            |
| `rm -rf 目标路径`                       | 强制递归删除文件/目录（谨慎使用）         | `sudo rm -rf /tmp/unused`           |
| `tar -zcvf 压缩包名.tar.gz 源路径`      | 压缩并打包文件（gzip格式）                | `tar -zcvf data.tar.gz /data/dev`   |
| `tar -zxvf 压缩包名.tar.gz -C 解压路径` | 解压gzip格式压缩包（-C 指定解压目录）     | `tar -zxvf data.tar.gz -C /tmp/`    |
| `zip -r 压缩包名.zip 源路径`            | 压缩为zip格式（需先装zip包）              | `zip -r data.zip /data/dev`         |
| `unzip 压缩包名.zip -d 解压路径`        | 解压zip格式压缩包                         | `unzip data.zip -d /tmp/`           |

### 四、存储与挂载管理

| 命令/语法                                                    | 说明                                          | 示例                                                         |
| ------------------------------------------------------------ | --------------------------------------------- | ------------------------------------------------------------ |
| `sudo umount -f /mnt/挂载点`                                 | 强制卸载挂载点（适用于NFS等无法正常卸载场景） | `sudo umount -f /mnt/nfs_mount`                              |
| `sudo mount -t nfs -o nolock 服务端IP:服务端目录 本地挂载点` | 挂载NFS共享目录                               | `sudo mount -t nfs -o nolock 192.168.100.4:/opt /opt`        |
| `mount`                                                      | 查看所有已挂载的文件系统                      | `mount`                                                      |
| `df -h`                                                      | 查看磁盘空间使用情况（-h 人性化显示单位）     | `df -h`                                                      |
| `du -sh 目录路径`                                            | 查看目录总大小（-s 汇总，-h 人性化单位）      | `du -sh /data`                                               |
| `sudo mount /dev/sdb1 /mnt/usb`                              | 挂载硬盘/U盘分区到指定目录                    | `sudo mount /dev/sdb1 /mnt/usb`                              |
| `sudo blkid`                                                 | 查看磁盘分区UUID（用于fstab永久挂载）         | `sudo blkid`                                                 |
| `sudo vim /etc/fstab`                                        | 配置开机自动挂载（编辑后需执行mount -a验证）  | 编辑后添加：`UUID=xxx /data ext4 defaults 0 2`               |
| `sudo mount -a`                                              | 验证fstab配置并加载所有未挂载的项             | `sudo mount -a`                                              |
| `free -h`                                                    | 查看内存/swap使用情况（-h 人性化单位）        | `free -h`                                                    |
| `swapon -s`                                                  | 查看swap分区使用情况                          | `swapon -s`                                                  |
| `sudo fallocate -l 10G /swapfile`                            | 创建10G的swap文件（应急扩容swap）             | 后续需执行：`sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile` |

### 五、系统基础运维

| 命令/语法                                        | 说明                                           | 示例                                             |
| ------------------------------------------------ | ---------------------------------------------- | ------------------------------------------------ |
| `sudo -i`                                        | 切换到root用户环境（加载root所有环境变量）     | `sudo -i`                                        |
| `sudo apt update`                                | 更新Debian/Ubuntu系软件包索引                  | `sudo apt update`                                |
| `sudo apt upgrade`                               | 升级已安装的软件包（Debian/Ubuntu系）          | （-y 自动确认）`sudo apt upgrade -y`             |
| `sudo apt install 软件包名`                      | 安装Debian/Ubuntu系软件包                      | `sudo apt install nginx`                         |
| `sudo apt remove 软件包名`                       | 卸载Debian/Ubuntu系软件包（保留配置）          | `sudo apt remove nginx`                          |
| `sudo apt purge 软件包名`                        | 彻底卸载软件包（删除配置）                     | `sudo apt purge nginx`                           |
| `sudo dpkg -i 软件包.deb`                        | 安装本地deb格式软件包                          | `sudo dpkg -i nginx_1.24.0-1_amd64.deb`          |
| `sudo yum install 软件包名`                      | 安装RHEL/CentOS系软件包                        | `sudo yum install nginx -y`                      |
| `sudo reboot`                                    | 重启服务器                                     | `sudo reboot`                                    |
| `sudo shutdown -h now`                           | 立即关机                                       | `sudo shutdown -h now`                           |
| `sudo shutdown -h 20:00`                         | 定时关机（20:00）                              | `sudo shutdown -h 20:00`                         |
| `uname -a`                                       | 查看系统内核、架构等信息                       | `uname -a`                                       |
| `hostname`                                       | 查看主机名                                     | `hostname`                                       |
| `sudo hostname 新主机名`                         | 临时修改主机名（重启失效）                     | `sudo hostname server-01`                        |
| `lsb_release -a`                                 | 查看系统发行版信息（Debian/Ubuntu系）          | `lsb_release -a`                                 |
| `who`                                            | 查看当前登录系统的用户                         | `who`                                            |
| `last`                                           | 查看用户登录历史记录                           | `last`                                           |
| `history`                                        | 查看当前用户执行过的命令历史                   | `history`                                        |
| `sudo sync && echo 3 > /proc/sys/vm/drop_caches` | 清理系统页缓存、目录项和inodes（临时释放内存） | `sudo sync && echo 3 > /proc/sys/vm/drop_caches` |

### 六、网络管理

| 命令/语法                                            | 说明                                                         | 示例                                                 |
| ---------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| `ip a`                                               | 查看所有网卡IP、MAC等信息（替代ifconfig）                    | `ip a`                                               |
| `ping -c 4 目标IP/域名`                              | 测试网络连通性（-c 指定发包数）                              | `ping -c 4 192.168.100.4`                            |
| `traceroute 目标IP/域名`                             | 跟踪数据包路由路径                                           | `traceroute www.baidu.com`                           |
| `mtr 目标IP/域名`                                    | 结合ping和traceroute，实时监控路由丢包                       | `mtr www.baidu.com`                                  |
| `netstat -tulnp`                                     | 查看监听的端口及对应进程（t=TCP,u=UDP,l=监听,n=数字显示,p=进程） | `netstat -tulnp`                                     |
| `ss -tulnp`                                          | 替代netstat，更快查看端口/进程信息                           | `ss -tulnp`                                          |
| `sudo ufw enable`                                    | 启用Ubuntu防火墙（ufw）                                      | `sudo ufw enable`                                    |
| `sudo ufw allow 22/tcp`                              | 允许22端口TCP流量（SSH）                                     | `sudo ufw allow 22/tcp`                              |
| `sudo ufw status`                                    | 查看ufw防火墙规则状态                                        | `sudo ufw status`                                    |
| `sudo iptables -L -n`                                | 查看iptables防火墙规则（n=数字显示）                         | `sudo iptables -L -n`                                |
| `sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT` | 允许80端口TCP入站流量                                        | `sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT` |
| `ssh 用户名@服务器IP`                                | 远程连接服务器                                               | `ssh user1@192.168.100.4`                            |
| `scp 本地文件 用户名@服务器IP:远程路径`              | 本地文件上传到服务器                                         | `scp /data/file.txt user1@192.168.100.4:/data/`      |
| `scp 用户名@服务器IP:远程文件 本地路径`              | 从服务器下载文件到本地                                       | `scp user1@192.168.100.4:/data/file.txt /tmp/`       |

### 七、进程管理

| 命令/语法         | 说明                                                     | 示例                                     |
| ----------------- | -------------------------------------------------------- | ---------------------------------------- |
| `ps -ef`          | 查看所有进程（e=所有进程，f=全格式）                     | （过滤nginx进程）`ps -ef | grep nginx`   |
| `ps aux`          | 查看进程资源占用（a=所有用户，u=用户维度，x=无终端进程） | （按CPU占用降序）`ps aux | sort -k 3 -r` |
| `top`             | 实时监控进程资源占用（CPU、内存）                        | （按P排序CPU，按M排序内存）`top`         |
| `htop`            | 增强版top（需先安装）                                    | `htop`                                   |
| `kill -9 进程ID`  | 强制终止进程（9=强制信号）                               | `kill -9 1234`                           |
| `killall 进程名`  | 终止所有同名进程                                         | `killall nginx`                          |
| `pkill 进程名`    | 按进程名终止进程（支持模糊匹配）                         | `pkill nginx`                            |
| `lsof -i :端口号` | 查看占用指定端口的进程                                   | `lsof -i :80`                            |
|                   |                                                          |                                          |