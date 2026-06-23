# Ubuntu 22\.04\.5 LTS 完整部署\+基础命令手册

## 一、文档说明

1、系统版本：**Ubuntu 22\.04\.5 LTS Server 服务端版（无桌面，生产推荐）**

2、适用场景：VMware虚拟机部署、物理服务器装机、云服务器重装系统

3、核心内容：系统图文化部署步骤、装机初始化优化、APT源更换、用户权限、文件、网络、磁盘、服务、防火墙全套基础命令

4、默认账号：装机自定义普通用户 \+ root超级管理员

---

## 二、前置准备资源

### 2\.1 系统镜像

官方镜像下载地址：https://releases\.ubuntu\.com/22\.04\.5/

镜像文件名：**ubuntu\-22\.04\.5\-live\-server\-amd64\.iso**

### 2\.2 硬件最低配置要求

- CPU：2核及以上

- 内存：4G及以上（最小2G，2G容易安装报错）

- 磁盘：40G及以上

- 网卡：NAT模式/桥接模式，保证联网

---

## 三、VMware虚拟机Ubuntu22\.04\.5装机部署全过程

### 3\.1 虚拟机新建配置

1. VMware新建虚拟机→自定义\(高级\)→兼容性默认最新版本

2. 选择：稍后安装操作系统→客户机操作系统：Linux→版本：Ubuntu 64位

3. 自定义虚拟机名称\+存储位置，建议英文路径，禁止中文文件夹

4. CPU：分配2核4线程，内存：4GB

5. 网络类型：桥接模式（局域网互通）/NAT模式（仅上网）

6. 磁盘：新建虚拟磁盘40G，存储为单个文件

7. 完成虚拟机创建，挂载ubuntu\-22\.04\.5\-live\-server\-amd64\.iso镜像，开机启动

### 3\.2 系统可视化装机步骤（重点配置）

1. 语言选择：English（推荐，中文容易出现终端乱码）→Enter确认

2. 键盘布局：默认English\(US\)，无需修改

3. 网络配置：自动DHCP获取IP，联网后直接下一步，静态IP后文初始化修改

4. 代理配置：默认空，不配置代理

5. 镜像仓库：装机默认官方源，装机完成后更换国内清华源

6. 磁盘分区【生产必选】：**Use an entire disk**全盘自动分区，新手不要手动分区

7. 勾选：Set up this disk as an LVM group 开启LVM，方便后期扩容磁盘

8. 确认分区：选择Done→确认Continue写入磁盘（数据清空）

9. 用户信息配置（牢记账号密码）：
            

    - Your name：自定义用户名全称

    - Server name：主机名（服务器名称）

    - Username：登录用户名（小写英文，例如ubuntu）

    - Password：登录密码，复杂度达标

10. 关键选项：**Install OpenSSH server 勾选安装SSH服务**（远程连接必备，必须勾选）

11. 预装软件：其余默认不勾选，等待系统安装完成

12. 安装结束选择Reboot Now重启，弹出移除镜像回车确认

### 3\.3 装机后首次登录

输入装机设置用户名\+密码登录系统，登录后默认普通用户权限

---

## 四、系统装机后初始化配置（必做）

### 4\.1 切换root管理员账号

Ubuntu22\.04默认root未启用，先设置root密码

```bash
# 设置root密码，输入两次自定义密码即可
sudo passwd root
# 切换root管理员登录
su root
```

### 4\.2 更换清华APT国内源（解决下载慢、超时）

1. 备份原有官方源文件

```bash
cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

1. 编辑源文件，清空原有内容，粘贴22\.04专属清华源

```bash
vim /etc/apt/sources.list
```

粘贴以下全部清华源内容：

```bash
# 默认注释了源码镜像以提高 apt 更新速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
```

1. 更新软件缓存、升级系统补丁

```bash
# 更新源索引
apt update
# 升级系统全部软件包
apt upgrade -y
```

### 4\.3 固定静态IP地址（生产必备）

Ubuntu22\.04采用netplan管理网络，配置文件路径固定：/etc/netplan/

```bash
# 查看网卡名称
ip addr
# 编辑netplan配置文件，文件名每个人略有区别
vim /etc/netplan/00-installer-config.yaml
```

yaml严格缩进，复制修改网段即可：

```bash
network:
  ethernets:
    ens33:  # 替换为自己的网卡名称
      addresses: [192.168.1.100/24] # 静态IP/子网掩码
      gateway4: 192.168.1.1 # 网关
      nameservers:
        addresses: [223.5.5.5,114.114.114.114] # DNS
  version: 2
```

```bash
# 生效网络配置
netplan apply
```

### 4\.4 安装基础工具包

```bash
apt install vim net-tools lrzsz tree curl wget -y
```

---

## 五、Ubuntu22\.04 全套分类基础命令

### 5\.1 系统信息查看命令

```bash
# 查看系统版本（确认22.04.5）
lsb_release -a
# 查看内核版本
uname -r
# 查看开机运行时长、负载
uptime
# 查看主机名、修改主机名
hostname
hostnamectl set-hostname ubuntu-server
# 关机、重启、注销
poweroff      # 关机
reboot        # 重启
exit          # 退出终端/退出root
```

### 5\.2 用户与权限管理命令

```bash
# 新增普通用户
useradd testuser
# 设置用户密码
passwd testuser
# 删除用户
userdel -r testuser
# 查看当前登录用户
whoami
# 临时提权执行命令（普通用户专用）
sudo 命令
# 文件权限修改
chmod 755 文件名    # 修改读写执行权限
chown root:root 文件名 # 修改文件所属用户组
```

### 5\.3 文件目录基础命令

```bash
# 切换目录
cd /home        # 进入home目录
cd ~            # 回到当前用户家目录
cd ..           # 返回上一级目录
# 查看目录文件
ls              # 普通查看
ls -lh          # 人性化查看大小权限
tree            # 树形展示目录
# 创建/删除
mkdir 文件夹名        # 新建文件夹
mkdir -p a/b/c       # 递归创建多级文件夹
rm 文件名             # 删除文件
rm -rf 文件夹         # 强制删除文件夹（慎用）
# 复制移动
cp 文件 目标路径
mv 文件 目标路径/重命名文件
# 查看文件内容
cat 文件名       # 查看小文件
less 文件名      # 分页查看大文件
tail -f 日志文件 # 实时监控日志输出
```

### 5\.4 网络运维命令

```bash
# 查看网卡IP、网卡信息
ip addr
ifconfig
# 测试网络连通
ping www.baidu.com
# 测试端口连通性
curl 192.168.1.100:22
# 查看端口占用
ss -tulnp
# DNS测试解析
nslookup www.baidu.com
```

### 5\.5 磁盘空间命令

```bash
# 查看整机磁盘使用率
df -h
# 查看文件夹占用大小
du -sh 文件夹路径
# 查看磁盘分区
fdisk -l
```

### 5\.6 进程资源命令

```bash
# 查看CPU内存进程
top
# 查找指定进程
ps -ef | grep ssh
# 杀死进程
kill -9 进程ID
```

### 5\.7 APT软件安装卸载命令

```bash
# 更新软件源索引
apt update
# 升级系统所有软件
apt upgrade -y
# 安装软件
apt install 软件名 -y
# 卸载软件（保留配置）
apt remove 软件名
# 彻底卸载软件（删除配置）
apt purge 软件名
# 搜索仓库软件
apt search 软件名
```

### 5\.8 防火墙命令（ufw默认防火墙）

Ubuntu22\.04默认防火墙ufw，默认关闭

```bash
# 查看防火墙状态
ufw status
# 开启防火墙
ufw enable
# 关闭防火墙
ufw disable
# 放行指定端口
ufw allow 22/tcp
# 禁止指定端口
ufw deny 80
# 删除端口规则
ufw delete allow 22
```

### 5\.9 SSH远程连接常用配置

```bash
# 查看ssh服务状态
systemctl status ssh
# 开机自启ssh
systemctl enable ssh
# 重启ssh服务
systemctl restart ssh
# 编辑ssh配置允许root远程登录
vim /etc/ssh/sshd_config
# 修改：PermitRootLogin yes  保存重启ssh
```

### 5\.10 系统服务统一管理命令

```bash
# 所有服务通用格式
systemctl start 服务名      # 启动服务
systemctl stop 服务名       # 停止服务
systemctl restart 服务名    # 重启服务
systemctl status 服务名     # 查看状态
systemctl enable 服务名     # 开机自启
systemctl disable 服务名    # 取消开机自启
```

---

## 六、常见装机报错解决方案

1. 安装卡住报错：内存不足，虚拟机调整内存≥4G

2. apt下载超时：已更换清华源，重启网络即可

3. root无法远程ssh：修改sshd\_config开启PermitRootLogin yes

4. netplan配置报错：yaml文件空格严格对齐，禁止使用Tab制表符

> （注：部分内容可能由 AI 生成）
