#  服务器运维手册

系统用户添加 + Samba 映射 + NFS 服务搭建

# 

## 第一章 添加部门人员系统账号

（以用户 ljb 为例）

### 1.1 创建系统用户并指定家目录
```shell
# -m：自动创建家目录；-s：指定登录shell；-d：自定义家目录路径
sudo useradd -m -s /bin/bash -d /media/bsj/HDD2/hw/ljb ljb
```
### 1.2 设置用户密码
```shell
sudo passwd ljb
# 执行后按提示输入密码（建议与Samba密码区分）
```



## 第二章 Samba 服务安装与共享配置

（Windows 访问支持）

### 2.1 安装 Samba 依赖包

（Debian/Ubuntu）

```shell
sudo apt update && sudo apt install -y samba samba-common-bin
```

### 2.2 配置 Samba 共享目录
#### 2.2.1 编辑核心配置文件
```shell
sudo vim /etc/samba/smb.conf
```

#### 2.2.2 添加共享规则

在文件末尾追加

```ini
[ljb]
comment = Shared directory for department
path = /media/bsj/HDD2/hw/ljb
browseable = yes
writable = yes
valid users = ljb
create mask = 0755
directory mask = 0775
force user = ljb
force group = ljb
valid users = ljb
```
> ⚠️ 注意：配置中不要保留注释，避免语法解析错误

### 2.3 添加 Samba 授权用户

（需与系统用户同名）

```shell
# 将系统用户 ljb 加入 Samba 用户库，设置独立访问密码
sudo smbpasswd -a ljb
```

### 2.4 配置共享目录权限
```shell
# 授予 ljb 用户对共享目录的读写执行权限
sudo chmod -R 775 /media/bsj/HDD2/hw/ljb
# 绑定目录所属用户组为 ljb
sudo chown -R ljb:ljb /media/bsj/HDD2/hw/ljb
```

### 2.5 启动并设置 Samba 开机自启
```shell
# 重启 Samba 服务使配置生效
sudo systemctl restart smbd nmbd
# 设置开机自动启动
sudo systemctl enable smbd nmbd
```

### 2.6 Samba 服务验证与问题排查
```shell
# 1. 检查服务运行状态（active(running) 为正常）
sudo systemctl status smbd nmbd

# 2. 验证 Samba 用户是否存在（替换 ljb 为目标用户名）
sudo pdbedit -L -v | grep ljb

# 3. 本地测试访问（替换 ljb 为目标用户名）
smbclient //localhost/ljb -U ljb
# 输入密码后进入交互界面，说明认证成功
```

---



## 第三章 NFS 文件系统搭建

（跨设备 Linux 共享）

> 📌 核心概念：NFS 分「服务端」（提供共享目录）和「客户端」（挂载共享目录）

### 3.1 NFS 服务端配置
#### 3.1.1 安装 NFS 服务端软件
```shell
sudo apt update && sudo apt install -y nfs-kernel-server
```

#### 3.1.2 创建共享目录

（示例路径 /nfs，可自定义）

```shell
# 创建目录（-p 确保父目录不存在时自动创建）
sudo mkdir -p /nfs
# 设置目录权限（允许客户端读写）
sudo chmod -R 755 /nfs
sudo chown -R nobody:nogroup /nfs
```

#### 3.1.3 配置 NFS 共享规则
```shell
# 编辑 exports 配置文件
sudo vim /etc/exports
```

在文件末尾添加以下规则（二选一，根据网络环境选择）：
```ini
# 示例1：仅允许 192.168.1.0/24 网段设备访问（推荐，安全性更高）
/nfs 192.168.1.0/24(rw,sync,no_root_squash,no_subtree_check)

# 示例2：允许所有设备访问（测试环境使用，不推荐生产环境）
# /nfs *(rw,sync,no_root_squash,no_subtree_check)
```

#### 3.1.4 规则参数说明
| 参数            | 作用说明                                  |
|-----------------|-------------------------------------------|
| rw              | 允许客户端读写（ro 为只读权限）           |
| sync            | 数据实时写入磁盘，保证数据一致性（推荐）  |
| no_root_squash  | 允许客户端 root 用户保持权限（谨慎使用）  |
| no_subtree_check| 关闭子目录检查，提升 NFS 服务稳定性       |

#### 3.1.5 启动 NFS 服务并设置自启
```shell
# 重新导出/etc/exports中的所有共享目录
sudo exportfs -r
# 启动 NFS 服务
sudo systemctl start nfs-kernel-server
# 设置开机自启
sudo systemctl enable nfs-kernel-server
```

#### 3.1.6 开放防火墙端口

（客户端可访问的关键）

```shell
# UFW 防火墙（Ubuntu 默认）放行 NFS 服务
sudo ufw allow nfs
# 重启防火墙使规则生效
sudo ufw reload
```
> 📌 备选方案：测试环境可直接关闭防火墙 `sudo ufw disable`

#### 3.1.7 服务端验证共享是否生效

```shell
# 列出本地已共享的 NFS 目录
showmount -e localhost
# 输出包含 "/nfs 192.168.1.0/24" 即为配置成功
```

#### 3.1.8 服务端问题排查
```shell
# 若共享未生效或客户端无法连接，重启 NFS 服务
sudo systemctl restart nfs-kernel-server
```

### 3.2 NFS 客户端配置
#### 3.2.1 安装 NFS 客户端工具
```shell
sudo apt update && sudo apt install -y nfs-common
```

#### 3.2.2 创建本地挂载点

（自定义路径）

```shell
sudo mkdir -p /mnt/nfs_mount
```

#### 3.2.3 临时挂载 NFS 共享

（重启后失效）

```shell
# 格式：sudo mount -t nfs -o nolock 服务端IP:共享目录 客户端挂载点
sudo mount -t nfs -o nolock 192.168.100.4:/nfs /mnt/nfs_mount
```
> 替换说明：
> - `192.168.100.4`：替换为你的 NFS 服务端实际 IP
> - `/nfs`：服务端共享目录路径
> - `/mnt/nfs_mount`：客户端本地挂载点路径

#### 3.2.4 版本兼容性说明
```shell
# 查看服务端支持的 NFS 版本
cat /proc/fs/nfsd/versions
```
⚠️ 注意：Linux 内核 6.x 不支持 NFS v2，需确保服务端和客户端版本一致（推荐 v3/v4）



# 

## 第四章 Linux 内核版本降级操作

#### 4.1 查看系统支持安装的内核版本

```Shell
# 筛选通用版内核镜像包（generic 为通用内核标识）
apt-cache search linux-image | grep generic
```

#### 4.2 验证目标内核版本的可安装性

以 `6.11.0-29-generic` 版本为例，检查系统是否可安装该版本：

```Shell
apt policy linux-image-6.11.0-29-generic
```

输出示例（若 Candidate 为 none 表示无可用安装源，需更换版本）：

```Plain
linux-image-6.11.0-29-generic:
  Installed: (none)
  Candidate: (none)
  Version table:
```

#### 4.3 查看系统已安装的内核版本

```Shell
# 列出已安装的内核镜像文件，提取版本号
ls /boot/vmlinuz-* | sed 's|/boot/vmlinuz-||'
```

#### 4.4 安装指定版本内核

```Shell
# 1. 更新软件源
sudo apt update

# 2. 安装目标内核镜像和头文件（替换为实际要安装的版本）
sudo apt install linux-image-6.11.0-29-generic linux-headers-6.11.0-29-generic

# 3. 更新 GRUB 配置（预加载内核信息）
sudo update-grub
```

#### 4.5 配置 GRUB 启动项

（设置默认内核+显示启动菜单）

##### 4.5.1 编辑 GRUB 核心配置文件

```Shell
sudo vim /etc/default/grub
```

##### 4.5.2 修改配置项

（替换为目标内核版本）

| 原配置项                    | 新配置项                                              | 配置作用                  |
| --------------------------- | ----------------------------------------------------- | ------------------------- |
| `GRUB_DEFAULT=0`            | `GRUB_DEFAULT="Ubuntu, with Linux 6.11.0-29-generic"` | 设置默认启动指定内核      |
| `GRUB_TIMEOUT_STYLE=hidden` | `GRUB_TIMEOUT_STYLE=menu`                             | 显示 GRUB 启动选择菜单    |
| `GRUB_TIMEOUT=0`            | `GRUB_TIMEOUT=5`                                      | 菜单停留 5 秒，支持手动选 |

修改后的完整核心配置示例：

```TOML
GRUB_DEFAULT="Ubuntu, with Linux 6.11.0-29-generic"
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`( . /etc/os-release; echo ${NAME:-Ubuntu} ) 2>/dev/null || echo Ubuntu`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""
```



#### 4.6 生效 GRUB 配置并验证

```Shell
# 1. 重新生成 GRUB 配置文件
sudo update-grub
```

输出验证：配置成功会看到类似以下内容，说明目标内核已被识别

```Plain
Found linux image: /boot/vmlinuz-6.14.0-37-generic
Found linux image: /boot/vmlinuz-6.11.0-29-generic
...
done
```

**重要注意事项**：执行sudo update-grub后，若出现如下警告（实际操作中已触发），需调整GRUB_DEFAULT配置值：

```Plain
Warning: Please don't use old title `Ubuntu, with Linux 6.11.0-29-generic' for GRUB_DEFAULT, use `Advanced options for Ubuntu>Ubuntu, with Linux 6.11.0-29-generic' (for versions before 2.00) or `gnulinux-advanced-b10358f4-4f92-45fb-81e2-bf90b01c801b>gnulinux-6.11.0-29-generic-advanced-b10358f4-4f92-45fb-81e2-bf90b01c801b' (for 2.00 or later)
```

解决方案：根据警告提示，将/etc/default/grub文件中的GRUB_DEFAULT配置替换为对应格式（二选一，适配自身GRUB版本），修改后重新执行sudo update-grub生效：

```TOML
# 适配 GRUB 2.00 之前版本
GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.11.0-29-generic"

# 适配 GRUB 2.00 及之后版本（实际日志中提示的格式）
GRUB_DEFAULT="gnulinux-advanced-b10358f4-4f92-45fb-81e2-bf90b01c801b>gnulinux-6.11.0-29-generic-advanced-b10358f4-4f92-45fb-81e2-bf90b01c801b"
```

#### 4.7 重启系统并确认内核版本

```Shell
# 1. 重启系统使内核生效
sudo reboot

# 2. 重启后验证当前内核版本
uname -a
```

输出示例（包含目标版本号即为降级成功）：

```Plain
Linux ubuntu 6.11.0-29-generic #31-Ubuntu SMP Tue Aug 1 12:37:51 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```



## 第五章 开机自动挂载操作

Linux 系统实现磁盘/目录开机自动挂载的核心是编辑 `/etc/fstab` 配置文件，该文件定义了开机时需自动挂载的文件系统规则，每行配置格式为：  

```
[设备/共享标识] [本地挂载点] [文件系统类型] [挂载参数] [dump备份标识] [fsck检查优先级]
```

#### 5.1 本地 EXT4 格式磁盘开机自动挂载

配置示例

```Plain
UUID=4b817199-22f7-4055-8467-20c087f5def4 /Disk-12TB    ext4  defaults  0   1
```

字段说明

| 字段           | 取值/解释                                                    |
| -------------- | ------------------------------------------------------------ |
| 设备标识       | `UUID=xxx`：用磁盘UUID（而非`/dev/sdX`）避免设备名变动导致挂载失败（UUID可通过`blkid`命令查询） |
| 本地挂载点     | `/Disk-12TB`：自定义目录（需提前用`mkdir -p /Disk-12TB`创建） |
| 文件系统类型   | `ext4`：对应磁盘的实际文件系统格式                           |
| 挂载参数       | `defaults`：包含rw（读写）、auto（开机自动挂载）等默认参数   |
| dump备份标识   | `0`：不启用dump工具备份（1为启用）                           |
| fsck检查优先级 | `1`：开机时fsck磁盘检查的最高优先级（0为不检查）             |

#### 5.2 NFS 共享目录开机自动挂载

配置示例

```Plain
192.168.10.45:/media/bsj/HDD2/nfs  /nfs/192.168.10.45  nfs  nolock,_netdev,x-systemd.automount,x-systemd.requires=network-online.target  0  0
```

字段说明

| 字段             | 取值/解释                                                    |
| ---------------- | ------------------------------------------------------------ |
| NFS共享标识      | `192.168.10.45:/media/bsj/HDD2/nfs`：NFS服务端IP + 共享目录路径 |
| 本地挂载点       | `/nfs/192.168.10.45`：客户端本地挂载目录（需提前创建）       |
| 文件系统类型     | `nfs`：对应NFS文件系统                                       |
| 挂载参数（关键） | `nolock`：关闭NFS锁机制，提升兼容性<br>`_netdev`：标识挂载依赖网络，先启网再挂载<br>`x-systemd.automount`：systemd按需挂载（访问时才挂载）<br>`x-systemd.requires=network-online.target`：确保网络完全就绪后挂载 |
| dump备份标识     | `0`：NFS目录无需dump备份                                     |
| fsck检查优先级   | `0`：NFS目录无需fsck检查                                     |

#### 关键注意事项

1. 配置前需确保挂载点目录已创建（如`mkdir -p /nfs/192.168.10.45`）；
2. 配置完成后执行`mount -a`验证（无报错则配置有效），避免开机挂载失败；
3. NFS自动挂载需客户端已安装`nfs-common`，且服务端NFS服务正常、网络可达。

