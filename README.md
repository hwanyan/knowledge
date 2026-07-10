# 记录实战过程中开发与运维相关的知识库

## 目录

- [1. Linux 命令行提示符解析](#1-linux-命令行提示符解析)
- [2. 如何查看和修改主机名](#2-如何查看和修改主机名)
- [3. hostnamectl 命令的工作原理](#3-hostnamectl-命令的工作原理)
- [4. 利用终端工具连接云服务器后，终端提示符显示为 "bash-5.2$" 这种格式的原因](#4-利用终端工具连接云服务器后终端提示符显示为-bash-52-这种格式的原因)
- [5. 如何查看云服务器的基本信息](#5-如何查看云服务器的基本信息)
- [6. 新云服务器的必要基础配置](#6-新云服务器的必要基础配置)
- [7. sudo tee 命令的作用](#7-sudo-tee-命令的作用)
- [8. SSH 免密登录的原理](#8-ssh-免密登录的原理)
- [9. 服务器防火墙详解](#9-服务器防火墙详解)
- [10. 什么是代理和反向代理](#10-什么是代理和反向代理)
- [11. Context 传递登录态的技术方案](#11-context-传递登录态的技术方案)
- [12. JWT 签名（signToken）与验签（VerifyToken）原理](#12-jwt-签名signtoken与验签verifytoken原理)
- [13. MCP 协议的两种传输方式（SSE Transport / Streamable HTTP）](#13-mcp-协议的两种传输方式sse-transport--streamable-http)
- [14. 在服务器上编译基于 go 实现的后端代码并部署的流程](#14-服务器上编译基于-go-实现的后端代码并部署的流程)
- [15. 在服务器上编译基于 vue 实现的前端代码并部署的流程](#15-服务器上编译基于-vue-实现的前端代码并部署的流程)
- [16. PostgreSQL 安装与配置（CentOS 8）](#16-postgresql-安装与配置centos-8)
- [17. pgvector 扩展安装（CentOS 8 + PostgreSQL 16）](#17-pgvector-扩展安装centos-8--postgresql-16)
- [18. Nginx proxy_pass 末尾斜杠的差别（反代路径改写规则）](#18-nginx-proxy_pass-末尾斜杠的差别反代路径改写规则)
- [19. 多后端服务共存时的 URL 路径分层与网关前缀最佳实践](#19-多后端服务共存时的-url-路径分层与网关前缀最佳实践)
- [20. 反代到后端为什么不用 HTTP/2？](#20-反代到后端为什么不用-http2)
- [21. 仅 loopback 的 gRPC 服务与网关反代的意义及实现](#21-仅-loopback-的-grpc-服务与网关反代的意义及实现)
- [22. 前端调用 gRPC 服务还是 HTTP 网关？两者有何区别？](#22-前端调用-grpc-服务还是-http-网关两者有何区别)
- [23. grpc-gateway 路由注册机制与双端口架构](#23-grpc-gateway-路由注册机制与双端口架构)
- [24. 契约与部署拓扑解耦与资源导向的 API 路径设计](#24-契约与部署拓扑解耦与资源导向的-api-路径设计)
- [25. Go 编译期接口实现断言（`var _ Iface = (*T)(nil)`）](#25-Go-编译期接口实现断言var-_-iface--tnil)
- [26. 查看云服务器运行状态的常用命令速查](#26-查看云服务器运行状态的常用命令速查)
- [27. journalctl 单个服务日志过多的清理与重置方法](#27-journalctl-单个服务日志过多的清理与重置方法)
- [28. 在云服务器上把应用注册为 systemd 系统级服务](#28-在云服务器上把应用注册为-systemd-系统级服务)
- [29. 为什么生产环境中要用低权限用户跑业务进程](#29-为什么生产环境中要用低权限用户跑业务进程)
- [30. 云服务器上查看当前有哪些用户](#30-云服务器上查看当前有哪些用户)

---

## 1. Linux 命令行提示符解析
### 问题
在 Linux 系统中，输入命令行的前面那一堆 `[xxx@yyy ~]` 的含义是什么？

### 解答

在 Linux 系统中，命令行提示符（Prompt）的格式 `[xxx@yyy ~]` 各部分含义如下：

#### 各部分说明

- **xxx**：当前登录的用户名
- **@**：分隔符，表示 "at"（在）
- **yyy**：主机名（hostname），即当前计算机的名称
- **~**：当前工作目录
  - `~` 表示用户的家目录（home directory）
  - 如果在其他目录，会显示完整路径或相对路径
  - 例如：`/home/xxx/documents` 或 `/var/log`

#### 完整示例解读

```bash
[root@localhost ~]$
```

- **用户名**：root（超级管理员）
- **主机名**：localhost
- **当前目录**：~ (即 `/root`)
- **$**：普通用户提示符（`#` 表示 root 用户）

#### 提示符末尾符号

- `$`：表示当前是普通用户权限
- `#`：表示当前是 root 用户（超级管理员）权限

#### 自定义提示符

提示符格式由环境变量 `PS1` 控制，可以通过修改 `~/.bashrc` 或 `~/.bash_profile` 文件来自定义显示内容。

**常用 PS1 变量：**
- `\u`：当前用户名
- `\h`：主机名
- `\w`：完整工作目录路径
- `\W`：当前工作目录名称
- `\$`：根据用户权限显示 `$` 或 `#`

**示例自定义：**
```bash
# 在 ~/.bashrc 中添加
export PS1="[\u@\h \W]\$ "
```

---

## 2. 如何查看和修改主机名

### 问题
如何在 Linux 系统中查看和修改主机名？

### 解答

#### 查看主机名

有多种方法可以查看当前系统的主机名：

**方法一：使用 `hostname` 命令**
```bash
# 查看主机名
hostname

# 查看完整的主机名（FQDN - Fully Qualified Domain Name）
hostname -f
```

**方法二：使用 `hostnamectl` 命令（推荐）**
```bash
# 显示详细的主机名信息
hostnamectl

# 仅显示主机名
hostnamectl --static
```

**方法三：查看配置文件**
```bash
# CentOS/RHEL 7+ 系统
cat /etc/hostname

# 旧版系统（CentOS/RHEL 6）
cat /etc/sysconfig/network
```

#### 修改主机名

##### 临时修改（重启后失效）

```bash
# 使用 hostname 命令临时修改
sudo hostname new-hostname
```

这种方法修改后立即生效，但重启系统后会恢复到原来的主机名。

##### 永久修改（推荐方法）

**方法一：使用 `hostnamectl` 命令（适用于 CentOS/RHEL 7+、Ubuntu 16.04+）**

```bash
# 设置新的主机名
sudo hostnamectl set-hostname new-hostname

# 验证修改
hostnamectl
```

**方法二：手动编辑配置文件**

对于 **CentOS/RHEL 7+** 系统：
```bash
# 编辑 /etc/hostname 文件
sudo vim /etc/hostname
# 将内容改为新的主机名

# 编辑 /etc/hosts 文件（可选但推荐）
sudo vim /etc/hosts
# 添加或修改：
# 127.0.0.1   new-hostname
```

对于 **Ubuntu/Debian** 系统：
```bash
# 编辑 /etc/hostname 文件
sudo vim /etc/hostname

# 编辑 /etc/hosts 文件
sudo vim /etc/hosts
# 修改包含旧主机名的行为新主机名
```

对于 **CentOS/RHEL 6** 系统：
```bash
# 编辑 /etc/sysconfig/network 文件
sudo vim /etc/sysconfig/network
# 修改或添加：
# HOSTNAME=new-hostname
```

#### 修改后的操作

1. **重启网络服务或系统使修改生效：**
   ```bash
   # 对于使用 hostnamectl 的系统，通常立即生效
   # 如果需要，可以重启系统
   sudo reboot
   
   # 或者重新登录终端
   ```

2. **验证修改是否成功：**
   ```bash
   hostname
   hostnamectl
   ```

#### 注意事项

- 主机名命名规范：
  - 只能包含字母（a-z, A-Z）、数字（0-9）和连字符（-）
  - 不能以连字符开头或结尾
  - 长度通常不超过 64 个字符
  - 建议使用小写字母
  - 不要使用特殊字符和空格

- 修改主机名后，建议同时更新 `/etc/hosts` 文件，避免某些应用程序出现问题

- 在云服务器上修改主机名时，部分云平台的管理工具可能会在重启后重置主机名，需要查看云平台的相关文档

---

## 3. hostnamectl 命令的工作原理

### 问题
利用 `sudo hostnamectl set-hostname new-hostname` 更改服务器名称的本质是修改 `/etc/hostname` 文件么？

### 解答

是的，但不完全准确。`sudo hostnamectl set-hostname new-hostname` 的本质**不仅仅是**修改 `/etc/hostname` 文件，它做的事情更多。

#### `hostnamectl` 命令的实际操作

当你执行 `hostnamectl set-hostname` 时，它会：

1. **修改 `/etc/hostname` 文件**
   - 将新主机名写入该文件（持久化存储）

2. **通过 systemd 的 `hostnamed` 服务更新内核主机名**
   - 调用系统调用 `sethostname()` 立即更新运行时的主机名
   - 这等同于执行 `hostname new-hostname` 命令

3. **更新 systemd 的内部状态**
   - 通过 D-Bus 与 `systemd-hostnamed` 服务通信
   - 更新静态主机名、瞬态主机名和美化主机名（pretty hostname）

4. **可能触发相关服务的更新**
   - 某些依赖主机名的服务可能会收到通知

#### 与手动修改的区别

**手动修改 `/etc/hostname`：**
```bash
echo "new-hostname" | sudo tee /etc/hostname
```
- ✅ 修改了配置文件（重启后生效）
- ❌ **不会**立即更新当前运行中的主机名
- ❌ 需要重启系统或手动执行 `hostname new-hostname` 才能生效

**使用 `hostnamectl`：**
```bash
sudo hostnamectl set-hostname new-hostname
```
- ✅ 修改了配置文件（持久化）
- ✅ 立即更新运行时主机名（无需重启）
- ✅ 通过 systemd 规范化管理

#### 验证方法

你可以通过以下命令验证：

```bash
# 查看运行时主机名（内核中的）
hostname

# 查看配置文件中的主机名
cat /etc/hostname

# 查看所有主机名信息
hostnamectl status
```

#### 总结

`hostnamectl set-hostname` 的本质是**同时修改持久化配置和运行时状态**，它是一个更加完整和规范的解决方案，而不仅仅是简单地修改 `/etc/hostname` 文件。这就是为什么推荐使用 `hostnamectl` 而不是手动编辑文件的原因。

---

## 4. 利用终端工具连接云服务器后，终端提示符显示为 "bash-5.2$" 这种格式的原因

### 问题
为什么终端工具连接云服务器后，输入命令的地方显示 `bash-5.2$` 这种格式？

### 解答

当终端工具通过 SSH 连接到云服务器后，终端显示 `bash-5.2$` 这种简化的提示符，主要有以下几个原因：

#### 原因分析

**1. 使用了默认的 Bash 提示符**

当你通过 Remote-SSH 连接服务器时，启动的是一个**非登录 Shell**（non-login shell），这种 Shell 可能：
- 没有加载完整的用户配置文件（如 `～/.bashrc`、`~/.bash_profile`、`~/.profile`）
- 没有完整的用户配置文件（如 `～/.bashrc`、`~/.bash_profile`、`~/.profile`）
- 如果 `~/.bashrc` 中没有自定义 `PS1` 变量，就会使用 Bash 的默认提示符

**2. Bash 默认提示符格式**

`bash-5.2$` 的含义：
- **bash-5.2**：当前使用的 Shell 类型和版本号（Bash 5.2）
- **$**：普通用户权限标识（root 用户会显示 `#`）

这是 Bash 在没有自定义 `PS1` 环境变量时的默认行为。

**3. 配置文件未生效**

可能的情况：
- 服务器上的 `~/.bashrc` 文件不存在或为空
- `~/.bashrc` 中没有设置 `PS1` 变量
- 终端连接工具启动的 Shell 没有正确读取配置文件

#### 如何自定义提示符

如果你想在终端工具连接服务器后显示更友好的提示符（如 `[user@hostname ~]$`），可以通过以下方法：

**方法一：编辑 `~/.bashrc` 文件（推荐）**

```bash
# 在云服务器上编辑 ~/.bashrc 文件
vim ~/.bashrc

# 添加以下内容到文件末尾
export PS1="[\u@\h \W]\$ "

# 保存后，使配置生效
source ~/.bashrc
```

**方法二：使用更丰富的提示符**

```bash
# 彩色提示符示例
export PS1="\[\e[32m\]\u@\h\[\e[m\]:\[\e[34m\]\w\[\e[m\]\$ "

# 或者显示完整路径和时间
export PS1="[\u@\h \w \t]\$ "
```

**方法三：检查并创建配置文件**

```bash
# 检查是否存在 .bashrc
ls -la ~/.bashrc

# 如果不存在，创建一个
touch ~/.bashrc

# 添加基本的 PS1 配置
echo 'export PS1="[\u@\h \W]\$ "' >> ~/.bashrc

# 使配置生效
source ~/.bashrc
```

#### 验证配置

修改后，你可以：

1. **在当前终端验证：**
   ```bash
   source ~/.bashrc
   ```
2. **检查当前 PS1 值：**
   ```bash
   echo $PS1
   ```

#### 登录 Shell vs 非登录 Shell

# 登录Shell（通过ssh、su -、console登录）
```
/etc/profile
    ↓
/etc/profile.d/*.sh
    ↓
~/.bash_profile    # 如果存在
    ↓  
~/.bash_login      # 如果.bash_profile不存在
    ↓
~/.profile         # 如果前两个都不存在
    ↓
~/.bashrc          # 通常在上面某个文件中被显式调用
```

# 非登录Shell（终端内新开标签、su、执行脚本）
```
~/.bashrc
    ↓
/etc/bashrc        # 通常在.bashrc中被调用
```

这就是为什么建议将 `PS1` 配置写入 `~/.bashrc` 文件，这样无论是登录 Shell 还是非登录 Shell 都能生效。

#### 总结

`bash-5.2$` 是 Bash 的默认提示符，表示你正在使用 Bash 5.2 版本。要自定义提示符，只需在 `~/.bashrc` 文件中设置 `PS1` 环境变量即可。

---

## 5. 如何查看云服务器的基本信息

### 问题
拿到一台新的云服务器后，如何查看这台服务器的基本信息？

### 解答

查看云服务器的基本信息是管理服务器的第一步，以下是常用的查看方法：

#### 1. 查看系统信息

**查看操作系统版本**

```bash
# 查看系统发行版信息（适用于大多数 Linux 发行版）
cat /etc/os-release

# 查看内核版本
uname -r

# 查看完整系统信息
uname -a

# CentOS/RHEL 系统
cat /etc/redhat-release

# Ubuntu/Debian 系统
lsb_release -a
```

**查看系统架构**

```bash
# 查看系统架构（x86_64、aarch64 等）
arch

# 或使用
uname -m
```

#### 2. 查看 CPU 信息

```bash
# 查看详细 CPU 信息
cat /proc/cpuinfo

# 查看 CPU 型号
cat /proc/cpuinfo | grep "model name" | head -1

# 查看 CPU 核心数
lscpu | grep "^CPU(s):"

# 或者
nproc

# 查看完整 CPU 架构信息
lscpu
```

#### 3. 查看内存信息

```bash
# 查看内存使用情况（易读格式）
free -h

# 查看详细内存信息
cat /proc/meminfo

# 查看总内存大小
free -h | grep Mem | awk '{print $2}'
```

#### 4. 查看磁盘信息

```bash
# 查看磁盘分区和使用情况
df -h

# 查看所有磁盘和分区
lsblk

# 查看磁盘详细信息
fdisk -l

# 查看磁盘 I/O 统计
iostat
```

#### 5. 查看网络信息

```bash
# 查看网络接口信息
ip addr
# 或
ifconfig

# 查看公网 IP（如果有）
curl ifconfig.me
# 或
curl ip.sb

# 查看内网 IP
hostname -I

# 查看网络接口详细信息
ip link show

# 查看路由信息
ip route
# 或
route -n
```

#### 6. 查看主机名

```bash
# 查看主机名
hostname

# 查看详细主机名信息（systemd 系统）
hostnamectl
```

#### 7. 查看系统运行时间和负载

```bash
# 查看系统运行时间和平均负载
uptime

# 查看系统启动时间
who -b
```

#### 8. 查看已登录用户

```bash
# 查看当前登录用户
who

# 查看当前用户信息
whoami

# 查看所有用户
cat /etc/passwd

# 查看登录历史
last
```

#### 9. 使用综合工具一次性查看

**安装并使用 `neofetch`（推荐）**

```bash
# CentOS/RHEL
sudo yum install -y neofetch

# Ubuntu/Debian
sudo apt install -y neofetch

# 运行查看系统信息
neofetch
```

**使用 `htop` 查看实时系统状态**

```bash
# 安装 htop
sudo yum install -y htop   # CentOS/RHEL
sudo apt install -y htop   # Ubuntu/Debian

# 运行
htop
```

#### 10. 一键查看脚本示例

可以创建一个脚本一次性查看所有关键信息：

```bash
#!/bin/bash
echo "========== 系统信息 =========="
cat /etc/os-release | grep PRETTY_NAME
echo ""

echo "========== CPU 信息 =========="
lscpu | grep "Model name"
lscpu | grep "^CPU(s):"
echo ""

echo "========== 内存信息 =========="
free -h
echo ""

echo "========== 磁盘信息 =========="
df -h
echo ""

echo "========== 网络信息 =========="
echo "内网 IP: $(hostname -I)"
echo "公网 IP: $(curl -s ifconfig.me)"
echo ""

echo "========== 系统运行时间 =========="
uptime
```

保存为 `system_info.sh`，然后执行：

```bash
chmod +x system_info.sh
./system_info.sh
```

#### 总结

查看云服务器基本信息的核心命令：
- **系统版本**：`cat /etc/os-release`、`uname -a`
- **CPU**：`lscpu`、`nproc`
- **内存**：`free -h`
- **磁盘**：`df -h`、`lsblk`
- **网络**：`ip addr`、`curl ifconfig.me`
- **综合工具**：`neofetch`、`htop`

---

## 6. 新云服务器的必要基础配置

### 问题
在拿到一台新的云服务器时，要做哪些必要的基础配置？

### 解答

新云服务器的基础配置关系到服务器的安全性、稳定性和易用性。以下是按优先级排列的必要配置步骤：

#### 第一步：更新系统

**更新软件包是首要任务，确保系统安全**

```bash
# CentOS/RHEL 系统
sudo yum update -y

# Ubuntu/Debian 系统
sudo apt update && sudo apt upgrade -y
```

#### 第二步：修改主机名

**设置一个有意义的主机名便于识别**

```bash
# 查看当前主机名
hostname

# 修改主机名
sudo hostnamectl set-hostname your-server-name

# 验证
hostnamectl
```

#### 第三步：创建普通用户并配置 sudo 权限

**避免直接使用 root 用户，提高安全性**

```bash
# 创建新用户
sudo adduser username

# 为用户设置密码
sudo passwd username

# 将用户添加到 sudo 组
# CentOS/RHEL
sudo usermod -aG wheel username

# Ubuntu/Debian
sudo usermod -aG sudo username

# 验证 sudo 权限
su - username
sudo whoami  # 应该输出 root
```

#### 第四步：配置 SSH 安全

**禁用 root 登录和密码登录，使用 SSH 密钥认证**

**1. 配置 SSH 密钥登录**

```bash
# 在本地机器生成 SSH 密钥对（如果还没有）
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 将公钥复制到服务器
ssh-copy-id username@server_ip

# 或手动添加
# 在服务器上创建 .ssh 目录
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# 创建 authorized_keys 文件并粘贴公钥
vim ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

**2. 修改 SSH 配置**

```bash
# 备份原配置
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# 编辑 SSH 配置
sudo vim /etc/ssh/sshd_config

# 修改以下配置项：
# Port 22                          # 可以改为其他端口，如 2222
# PermitRootLogin no               # 禁止 root 直接登录
# PasswordAuthentication no        # 禁用密码登录（确保密钥登录已配置）
# PubkeyAuthentication yes         # 启用公钥认证
# PermitEmptyPasswords no          # 禁止空密码

# 重启 SSH 服务使配置生效
sudo systemctl restart sshd
```

**注意**：修改 SSH 配置前，请确保已经配置好密钥登录，并在另一个终端保持连接，避免锁死自己。

#### 第五步：配置防火墙

**只开放必要的端口**

**使用 firewalld（CentOS/RHEL 7+）**

```bash
# 启动并设置开机自启
sudo systemctl start firewalld
sudo systemctl enable firewalld

# 查看当前状态
sudo firewall-cmd --state

# 允许 SSH（默认端口 22）
sudo firewall-cmd --permanent --add-service=ssh

# 如果修改了 SSH 端口（例如 2222）
sudo firewall-cmd --permanent --add-port=2222/tcp

# 允许 HTTP 和 HTTPS（如果需要）
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

# 重载防火墙规则
sudo firewall-cmd --reload

# 查看已开放的服务和端口
sudo firewall-cmd --list-all
```

**使用 ufw（Ubuntu/Debian）**

```bash
# 安装 ufw（如果未安装）
sudo apt install -y ufw

# 允许 SSH（在启用防火墙前必须先允许）
sudo ufw allow ssh
# 或指定端口
sudo ufw allow 2222/tcp

# 允许 HTTP 和 HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 启用防火墙
sudo ufw enable

# 查看状态
sudo ufw status verbose
```

#### 第六步：设置时区和时间同步

**确保服务器时间准确**

```bash
# 查看当前时区
timedatectl

# 设置时区为上海（中国标准时间）
sudo timedatectl set-timezone Asia/Shanghai

# 启用 NTP 时间同步
sudo timedatectl set-ntp true

# 验证
timedatectl
```

#### 第七步：配置自动安全更新（可选但推荐）

**CentOS/RHEL**

```bash
# 安装 yum-cron
sudo yum install -y yum-cron

# 编辑配置
sudo vim /etc/yum/yum-cron.conf
# 修改：apply_updates = yes

# 启动并设置开机自启
sudo systemctl start yum-cron
sudo systemctl enable yum-cron
```

**Ubuntu/Debian**

```bash
# 安装 unattended-upgrades
sudo apt install -y unattended-upgrades

# 启用自动更新
sudo dpkg-reconfigure -plow unattended-upgrades
```

#### 第八步：安装常用工具

```bash
# CentOS/RHEL
sudo yum install -y vim git wget curl net-tools htop

# Ubuntu/Debian
sudo apt install -y vim git wget curl net-tools htop
```

#### 第九步：配置交换空间（Swap）

**如果内存较小，建议配置 Swap**

```bash
# 检查是否已有 swap
free -h

# 创建 2GB 的 swap 文件
sudo fallocate -l 2G /swapfile

# 设置权限
sudo chmod 600 /swapfile

# 格式化为 swap
sudo mkswap /swapfile

# 启用 swap
sudo swapon /swapfile

# 设置开机自动挂载
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 验证
free -h
```

#### 第十步：配置系统日志和监控（可选）

**启用系统日志**

```bash
# 确保 rsyslog 正在运行
sudo systemctl status rsyslog
sudo systemctl enable rsyslog
```

**查看重要日志位置**

```bash
# 系统日志
/var/log/messages        # CentOS/RHEL
/var/log/syslog          # Ubuntu/Debian

# SSH 登录日志
/var/log/secure          # CentOS/RHEL
/var/log/auth.log        # Ubuntu/Debian

# 使用 journalctl 查看 systemd 日志
sudo journalctl -xe
```

#### 第十一步：安装 Fail2Ban（防暴力破解）

```bash
# CentOS/RHEL（需要 EPEL 源）
sudo yum install -y epel-release
sudo yum install -y fail2ban

# Ubuntu/Debian
sudo apt install -y fail2ban

# 创建本地配置文件
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# 编辑配置
sudo vim /etc/fail2ban/jail.local

# 启动并设置开机自启
sudo systemctl start fail2ban
sudo systemctl enable fail2ban

# 查看状态
sudo fail2ban-client status
```

#### 配置检查清单

完成配置后，建议检查以下内容：

- [ ] 系统已更新到最新版本
- [ ] 主机名已修改为有意义的名称
- [ ] 已创建普通用户并配置 sudo 权限
- [ ] SSH 密钥登录已配置且测试通过
- [ ] SSH 已禁用 root 登录和密码登录
- [ ] 防火墙已启用并配置必要端口
- [ ] 时区和时间同步已正确配置
- [ ] 常用工具已安装
- [ ] Swap 已配置（如果需要）
- [ ] Fail2Ban 已安装并运行

#### 安全建议

1. **定期备份**：配置好服务器后，立即创建快照或备份
2. **最小权限原则**：只开放必要的端口和服务
3. **定期审查**：定期检查系统日志和登录记录
4. **及时更新**：保持系统和软件包的及时更新
5. **监控告警**：配置监控工具，及时发现异常

#### 总结

新云服务器的基础配置核心步骤：
1. 更新系统
2. 修改主机名
3. 创建普通用户
4. 配置 SSH 安全（密钥登录 + 禁用 root）
5. 配置防火墙
6. 设置时区和时间同步
7. 安装常用工具
8. 配置 Swap（可选）
9. 安装 Fail2Ban（推荐）

按照这些步骤完成配置后，你的云服务器将具备基本的安全性和可用性。

---

## 7. sudo tee 命令的作用

### 问题
`sudo tee` 命令的作用是什么？

### 解答

`tee` 命令是一个非常实用的 Linux 工具，它从标准输入读取数据，然后**同时**写入到标准输出和一个或多个文件中。结合 `sudo` 使用时，可以解决一个常见的权限问题。

#### tee 命令的基本原理

**命令格式：**
```bash
command | tee [选项] 文件名
```

**工作流程：**
```
输入数据 → tee → ┬→ 标准输出（屏幕）
                   └→ 写入文件
```

#### 基本用法示例

**示例 1：同时显示和保存输出**

```bash
# 将 ls 的结果同时显示在屏幕上并保存到文件
ls -la | tee file_list.txt

# 结果：
# 1. 屏幕上显示文件列表
# 2. 同时将列表写入 file_list.txt
```

**示例 2：追加模式**

```bash
# 使用 -a 参数追加内容，而不是覆盖
echo "新内容" | tee -a existing_file.txt
```

**示例 3：同时写入多个文件**

```bash
# 将输出同时写入多个文件
echo "测试内容" | tee file1.txt file2.txt file3.txt
```

#### sudo tee 的重要用途

**为什么需要 `sudo tee`？**

在 Linux 中，以下写法是**错误的**：

```bash
# ❌ 错误示例：重定向不会以 sudo 权限执行
sudo echo "content" > /etc/some_file

# 原因：虽然 echo 以 sudo 执行，但重定向符号 > 是由当前 shell 处理的
# shell 没有 sudo 权限，因此无法写入需要 root 权限的文件
```

**正确的做法是使用 `sudo tee`：**

```bash
# ✅ 正确示例：使用 tee 以 sudo 权限写入文件
echo "content" | sudo tee /etc/some_file

# tee 命令本身以 sudo 权限运行，因此可以写入受保护的文件
```

#### 常见使用场景

**场景 1：修改系统配置文件**

```bash
# 向 /etc/hosts 文件追加内容
echo "127.0.0.1 myapp.local" | sudo tee -a /etc/hosts

# 覆盖 /etc/hostname 文件
echo "new-hostname" | sudo tee /etc/hostname
```

**场景 2：静默模式（不显示输出）**

```bash
# 使用 > /dev/null 隐藏屏幕输出，只写入文件
echo "content" | sudo tee /etc/file > /dev/null

# 或者使用 -a 参数追加
echo "content" | sudo tee -a /etc/file > /dev/null
```

**场景 3：在脚本中使用**

```bash
#!/bin/bash
# 配置脚本示例

# 备份原文件
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# 追加新配置
echo "PermitRootLogin no" | sudo tee -a /etc/ssh/sshd_config > /dev/null
echo "PasswordAuthentication no" | sudo tee -a /etc/ssh/sshd_config > /dev/null

echo "SSH 配置已更新"
```

**场景 4：创建多行内容文件**

```bash
# 使用 cat 和 tee 创建多行文件
cat << 'EOF' | sudo tee /etc/myconfig.conf
[section1]
option1 = value1
option2 = value2

[section2]
option3 = value3
EOF
```

#### tee 命令的选项

常用选项：

| 选项 | 说明 |
|------|------|
| `-a` | 追加到文件末尾，而不是覆盖 |
| `-i` | 忽略中断信号（SIGINT） |
| `--help` | 显示帮助信息 |
| `--version` | 显示版本信息 |

#### 与其他方法的对比

**方法 1：使用重定向（普通权限）**
```bash
echo "content" > file.txt
# 优点：简单直接
# 缺点：无法同时显示输出；无法处理需要 sudo 的文件
```

**方法 2：使用 sudo tee**
```bash
echo "content" | sudo tee file.txt
# 优点：可以处理需要 sudo 的文件；同时显示输出
# 缺点：语法稍复杂
```

**方法 3：使用 sudo sh -c（不推荐）**
```bash
sudo sh -c 'echo "content" > /etc/file'
# 优点：可以处理需要 sudo 的文件
# 缺点：语法更复杂；安全性较低；不显示输出
```

#### 实用技巧

**技巧 1：管道链中使用 tee 保存中间结果**

```bash
# 在处理流程中保存中间结果
cat access.log | grep "ERROR" | tee errors.log | wc -l

# 流程：
# 1. 过滤出错误日志
# 2. 保存到 errors.log
# 3. 继续传递给 wc 统计行数
```

**技巧 2：调试脚本时保存输出**

```bash
# 同时显示脚本输出并保存到日志
./my_script.sh 2>&1 | tee script_output.log

# 2>&1 将标准错误重定向到标准输出
# tee 同时保存所有输出到文件
```

**技巧 3：实时查看并保存日志**

```bash
# 实时查看命令输出并保存
tail -f /var/log/syslog | tee current_log.txt
```

#### 总结

**`tee` 命令的核心作用：**
- 从标准输入读取数据
- 同时输出到屏幕和文件
- 可以输出到多个文件

**`sudo tee` 的关键用途：**
- 解决需要 root 权限写入文件的问题
- 替代失效的 `sudo echo "content" > /protected_file` 写法
- 在保持输出可见性的同时写入受保护的文件

**记忆口诀：**
- 需要写入受保护文件时，用 `sudo tee`
- 需要追加内容时，加上 `-a` 参数
- 不想看到屏幕输出时，重定向到 `/dev/null`

---

## 8. SSH 免密登录的原理

### 问题
SSH 免密登录是如何工作的？请用生动且直观的例子来解释其原理。

### 解答

SSH 免密登录的原理可以用一个**"钥匙和锁"**的生动比喻来理解。让我们通过一个完整的故事来讲解。

#### 🎭 故事比喻：门卫与访客的身份认证

想象你住在一个高级小区，有一个严格的门卫负责安全管理。

**传统密码登录方式（每次都要报暗号）：**

```
你（客户端）          门卫（服务器）
    |                      |
    |----"我想进门"-------->|
    |                      |
    |<---"暗号是什么？"-----|
    |                      |
    |----"芝麻开门"-------->|
    |                      |（验证暗号）
    |<---"暗号正确，进来吧"|
    |                      |
   进门
```

**问题：**
- 每次都要记住并说出暗号（密码）
- 暗号可能被偷听
- 暗号可能被暴力破解

#### 🔑 免密登录方式（钥匙和锁的配对）

现在，门卫给你一种更安全的方案：**公私钥配对**。

**第一步：制作一把特殊的锁和钥匙**

```bash
# 在你的电脑上（客户端）生成密钥对
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 这会生成两个文件：
# ~/.ssh/id_rsa       （私钥 = 你的钥匙，只有你有）
# ~/.ssh/id_rsa.pub   （公钥 = 你的锁，可以给别人）
```

**比喻：**
- **私钥（id_rsa）**：你的专属钥匙，像你的指纹，独一无二，**绝对不能给别人**
- **公钥（id_rsa.pub）**：配套的锁，可以安装在任何你想进入的门上

**第二步：把你的锁安装到小区门上**

```bash
# 把公钥复制到服务器
ssh-copy-id username@server_ip

# 或者手动添加
cat ~/.ssh/id_rsa.pub | ssh username@server_ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

**发生了什么：**
```
你（客户端）          门卫（服务器）
    |                      |
    |----"把我的锁装在门上"->|
    |                      |
    |                      |（把你的锁安装在 ~/.ssh/authorized_keys 文件中）
    |<---"锁已安装好了"-----|
```

**现在服务器的 `~/.ssh/authorized_keys` 文件里就有了你的"锁"（公钥）。**

#### 🚪 免密登录的完整过程

现在你再次想进门时：

```
你（客户端）          门卫（服务器）
    |                      |
    |----"我想进门"-------->|
    |                      |
    |<---"用你的钥匙试试"---|（发送一个随机挑战）
    |                      |
    |（用私钥签名挑战）      |
    |----"签名结果"-------->|
    |                      |
    |                      |（用你的公钥验证签名）
    |                      |（匹配！这确实是钥匙的主人）
    |<---"验证通过，进来吧"|
    |                      |
   进门（无需密码！）
```

#### 🔬 技术细节：非对称加密原理

**核心概念：**
- **公钥加密，私钥解密**
- **私钥签名，公钥验证**

**实际验证过程：**

1. **服务器生成随机挑战**：
   ```
   服务器：我给你一个随机数：12345678
   ```

2. **客户端用私钥签名**：
   ```
   客户端：用我的私钥对 12345678 进行签名
   结果：一串加密后的数据
   ```

3. **服务器用公钥验证**：
   ```
   服务器：用存储的公钥验证这个签名
   如果验证通过 → 说明对方确实持有配对的私钥 → 放行
   ```

#### 🎬 完整配置演示

**场景：你想从本地电脑免密登录到云服务器**

**在本地电脑上操作：**

```bash
# 1. 生成密钥对（如果还没有）
ssh-keygen -t rsa -b 4096 -C "myname@email.com"

# 提示：
# Enter file in which to save the key (/Users/you/.ssh/id_rsa): 
# [直接回车，使用默认路径]

# Enter passphrase (empty for no passphrase): 
# [可以设置密码短语，也可以直接回车不设置]

# 2. 查看生成的密钥
ls -la ~/.ssh/

# 输出：
# id_rsa       <- 私钥（钥匙）千万别泄露！
# id_rsa.pub   <- 公钥（锁）可以分享

# 3. 查看公钥内容
cat ~/.ssh/id_rsa.pub

# 输出类似：
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC... myname@email.com
```

**把公钥上传到服务器：**

**方法一：使用 ssh-copy-id（最简单）**

```bash
ssh-copy-id username@192.168.1.100

# 提示输入一次密码
# Password: ******

# 成功后会显示：
# Number of key(s) added: 1
```

**方法二：手动复制（如果 ssh-copy-id 不可用）**

```bash
# 在本地查看公钥
cat ~/.ssh/id_rsa.pub

# 登录到服务器
ssh username@192.168.1.100

# 在服务器上操作
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# 编辑 authorized_keys 文件
vim ~/.ssh/authorized_keys
# 粘贴刚才复制的公钥内容

# 设置正确的权限（很重要！）
chmod 600 ~/.ssh/authorized_keys

# 退出服务器
exit
```

**测试免密登录：**

```bash
# 现在再次连接，不需要密码了！
ssh username@192.168.1.100

# 如果成功，你会直接进入服务器，无需输入密码
```

#### 🔒 安全性分析

**为什么免密登录比密码更安全？**

| 对比项 | 密码登录 | 密钥登录 |
|--------|---------|---------|
| **暴力破解** | ❌ 容易被暴力破解 | ✅ 几乎不可能（4096位密钥） |
| **网络传输** | ❌ 密码可能被中间人截获 | ✅ 私钥从不传输 |
| **唯一性** | ❌ 可能使用弱密码 | ✅ 每个密钥对独一无二 |
| **可撤销性** | ❌ 改密码麻烦 | ✅ 删除公钥即可撤销 |

**私钥的保护：**

```bash
# 私钥文件的权限必须是 600（只有所有者可读写）
chmod 600 ~/.ssh/id_rsa

# 如果权限不对，SSH 会拒绝使用
# 错误提示：
# @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
# @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
# @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
# Permissions 0644 for '/Users/you/.ssh/id_rsa' are too open.
```

#### 🎯 实用场景

**场景 1：管理多台服务器**

```bash
# 一个公钥可以部署到多台服务器
ssh-copy-id user@server1.com
ssh-copy-id user@server2.com
ssh-copy-id user@server3.com

# 之后访问任何一台都不需要密码
```

**场景 2：配置 Git 仓库**

```bash
# GitHub/GitLab 也是用同样的原理
# 把公钥添加到 GitHub Settings -> SSH Keys

# 之后 git clone 就可以免密
git clone git@github.com:username/repo.git
```

**场景 3：使用不同的密钥对**

```bash
# 为不同用途生成不同的密钥对
ssh-keygen -t rsa -f ~/.ssh/id_rsa_work      # 工作用
ssh-keygen -t rsa -f ~/.ssh/id_rsa_personal  # 个人用

# 使用时指定密钥
ssh -i ~/.ssh/id_rsa_work user@work-server.com
```

**配置 SSH config 文件：**

```bash
# 编辑 ~/.ssh/config
vim ~/.ssh/config

# 添加配置：
Host work-server
    HostName 192.168.1.100
    User admin
    IdentityFile ~/.ssh/id_rsa_work

Host personal-server
    HostName 192.168.1.200
    User john
    IdentityFile ~/.ssh/id_rsa_personal

# 之后可以简化命令
ssh work-server      # 自动使用对应的密钥
ssh personal-server
```

#### ⚠️ 常见问题排查

**问题 1：仍然提示输入密码**

```bash
# 检查服务器上公钥是否正确安装
cat ~/.ssh/authorized_keys

# 检查文件权限
ls -la ~/.ssh/
# 应该是：
# drwx------  2 user user 4096 ... .ssh/
# -rw-------  1 user user  xxx ... authorized_keys

# 修正权限
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

**问题 2：权限被拒绝**

```bash
# 查看详细的 SSH 连接日志
ssh -v username@server_ip

# 或更详细的日志
ssh -vvv username@server_ip

# 在服务器上查看 SSH 日志
sudo tail -f /var/log/auth.log     # Ubuntu/Debian
sudo tail -f /var/log/secure       # CentOS/RHEL
```

**问题 3：服务器禁用了密钥认证**

```bash
# 在服务器上检查 SSH 配置
sudo vim /etc/ssh/sshd_config

# 确保以下配置正确：
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# 修改后重启 SSH 服务
sudo systemctl restart sshd
```

#### 📝 总结

**SSH 免密登录的核心原理：**

1. **生成密钥对**：创建一把钥匙（私钥）和一把锁（公钥）
2. **安装公钥**：把锁安装到服务器上
3. **验证身份**：服务器用锁验证你的钥匙
4. **放行通过**：验证通过后无需密码

**记忆口诀：**
- 私钥是钥匙，藏家里不给人
- 公钥是锁，装哪里都可以
- 钥匙开锁，配对才能进
- 一把钥匙，开遍天下锁

**最佳实践：**
1. ✅ 生成强度足够的密钥（至少 2048 位，推荐 4096 位）
2. ✅ 为私钥设置密码短语（额外保护层）
3. ✅ 正确设置文件权限（700 for .ssh, 600 for keys）
4. ✅ 定期轮换密钥
5. ✅ 离职或不再使用时，从服务器删除公钥

这就是 SSH 免密登录的完整原理！简单来说就是：**用数学证明你是你，而不是用密码。**

#### 💼 补充

**命令 `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"` 的参数解释**

1. `-t rsa`
- 作用：指定生成密钥所使用的算法类型。rsa 是一种历史悠久、应用广泛且兼容性最好的非对称加密算法。
- 其他选项：除了RSA，你还可以选择：
  - ed25519：更现代、更安全、更快速，且密钥更短。是目前的首选推荐，用法：-t ed25519（注意：使用此算法时通常不需要 -b 参数）。
  - ecdsa：基于椭圆曲线的算法，比RSA更高效。

2. `-b 4096`
- 作用：指定生成密钥的位数（长度）。对于RSA算法，这个数字决定了密钥的强度。
- 4096 的含义：
  - 密钥长度是 4096 比特。
  - 这是当前安全的标准。更长的位数意味着暴力破解的难度呈指数级增长。
  - 早期的默认长度是 2048 比特，虽然目前仍然安全，但 4096 提供了更强的面向未来的安全性。
- 注意：如果使用 `-t ed25519`，密钥长度是固定的（256位），所以不需要也不应再使用 `-b` 参数。

3. `-C "your_email@example.com"`
- 作用：为生成的公钥添加一个注释。这是一个标识信息，主要用于帮助用户识别这个密钥的用途或所有者。
- 注释的内容：
  - 它可以是任何字符串，但惯例是使用你的邮箱地址。
  - 这个注释会被附加在公钥文件的末尾。
- 实际用途：当你将公钥上传到 GitHub、GitLab 或服务器后，在这些服务的界面上看到密钥列表时，注释会显示出来，方便你区分和管理多个不同的密钥。
- 例如，你可能会看到：ssh-rsa AAAAB3NzaC1yc2E... your_email@example.com

---

## 9. 服务器防火墙详解

### 问题
什么是防火墙？服务器上开启和关闭防火墙的目的和作用是什么？

### 解答

防火墙是服务器安全的第一道防线。让我们通过生动的比喻来理解它。

#### 🏰 什么是防火墙？

**生动比喻：城堡的守卫**

想象你的服务器是一座城堡，防火墙就是城墙上的守卫。

```
互联网（外面的世界）
       |
       |  各种请求试图进入
       ↓
   ┌─────────────────┐
   │   防火墙守卫     │  ← 检查每个进出的人/数据
   │  (Firewall)     │
   └─────────────────┘
       |
       | 只放行允许的请求
       ↓
   ┌─────────────────┐
   │   你的服务器     │
   │   (城堡内部)     │
   └─────────────────┘
```

**守卫的职责：**
1. **检查来访者**：谁在敲门？从哪里来？想做什么？
2. **核对通行证**：这个人有权限进入吗？
3. **只放行合法访客**：符合规则的才能进，其他一律拒绝
4. **记录访问日志**：谁什么时候来过

#### 📝 防火墙的本质

**技术定义：**

防火墙（Firewall）是一个**网络安全系统**，它根据预定义的安全规则来监控和控制进出网络的流量。

**工作原理：**

```
外部请求 → 防火墙检查 → 符合规则？ → YES → 放行
                         ↓
                        NO
                         ↓
                      拦截/丢弃
```

**检查的维度：**
- **端口**：访问的是哪个端口？（如 SSH 使用 22 端口，HTTP 使用 80 端口）
- **协议**：使用什么协议？（TCP、UDP、ICMP）
- **IP 地址**：来自哪个 IP？去往哪个 IP？
- **方向**：是进来的流量（入站）还是出去的流量（出站）？

#### 🎯 防火墙的作用

**1. 保护服务器安全**

```
❌ 没有防火墙：
所有端口都开放 → 黑客可以尝试任意服务 → 高风险

✅ 有防火墙：
只开放必要端口 → 黑客攻击面大幅减少 → 安全
```

**2. 防止未授权访问**

```bash
# 示例：只允许 SSH 访问
防火墙规则：
- 允许 22 端口（SSH）
- 拒绝其他所有端口

结果：
✅ 你可以通过 SSH 管理服务器
❌ 黑客无法通过其他端口攻击
```

**3. 减少攻击面**

```
开放所有端口的服务器：
65535 个端口都可能被攻击 😱

只开放必要端口的服务器：
例如只开 22(SSH)、80(HTTP)、443(HTTPS) → 只有 3 个入口 ✅
```

**4. 防御常见攻击**

- **端口扫描**：黑客扫描开放的端口，防火墙可以阻止
- **DDoS 攻击**：配合规则限制连接数
- **暴力破解**：配合 Fail2Ban 阻止多次失败的登录尝试

#### 🔓 开启防火墙的目的

**为什么要开启防火墙？**

| 场景 | 没有防火墙 | 有防火墙 |
|------|-----------|---------|
| **SSH 服务** | 任何人都能尝试连接 | 只允许指定 IP 或全部拒绝 |
| **数据库** | 可能被外部直接访问 | 只允许本地访问 |
| **未知漏洞** | 服务直接暴露 | 端口被拦截，无法利用 |
| **恶意扫描** | 全部响应 | 大部分拒绝 |

**开启防火墙的核心目的：**

1. **最小权限原则**：默认拒绝所有，只开放必要的服务
2. **深度防御**：即使某个服务有漏洞，也不会暴露给外部
3. **合规要求**：很多安全标准要求必须启用防火墙
4. **审计追踪**：记录所有被拒绝的访问尝试

#### 🔒 关闭防火墙的场景

**什么情况下可能需要临时关闭？**

⚠️ **注意：生产环境不建议关闭防火墙！**

**可能的场景：**

1. **测试环境调试**
   ```bash
   # 调试网络问题时，临时关闭排查
   sudo systemctl stop firewalld
   
   # 问题解决后，立即重新开启
   sudo systemctl start firewalld
   ```

2. **内网测试环境**
   ```
   如果服务器在完全隔离的内网，且有其他安全措施
   ```

3. **特定云平台**
   ```
   某些云平台提供自己的安全组（Security Group）
   这种情况下，操作系统防火墙可能是多余的
   但仍建议保持开启，双重保护
   ```

**关闭防火墙的风险：**

| 风险 | 说明 |
|------|------|
| **端口全开** | 所有服务直接暴露给互联网 |
| **易被扫描** | 黑客可以轻易发现所有运行的服务 |
| **无保护层** | 服务漏洞直接可被利用 |
| **合规问题** | 不符合安全标准 |

#### 🛠️ Linux 防火墙工具

**主流工具对比：**

| 工具 | 系统 | 特点 |
|------|------|------|
| **firewalld** | CentOS/RHEL 7+ | 动态防火墙，支持区域概念 |
| **ufw** | Ubuntu/Debian | 简单易用，适合新手 |
| **iptables** | 所有 Linux | 底层工具，功能强大但复杂 |

#### 🎬 实战示例

**场景 1：配置 Web 服务器防火墙**

```bash
# 使用 firewalld (CentOS/RHEL)
sudo systemctl start firewalld
sudo systemctl enable firewalld

# 允许 SSH（管理用）
sudo firewall-cmd --permanent --add-service=ssh

# 允许 HTTP 和 HTTPS（网站服务）
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

# 重载规则
sudo firewall-cmd --reload

# 查看开放的端口
sudo firewall-cmd --list-all
```

**结果：**
```
public (active)
  target: default
  services: ssh http https
  ports: 
  
✅ 只有 22、80、443 端口可以访问
❌ 其他所有端口被拒绝
```

**场景 2：只允许特定 IP 访问 SSH**

```bash
# 移除默认的 SSH 服务规则
sudo firewall-cmd --permanent --remove-service=ssh

# 创建 rich rule，只允许特定 IP
sudo firewall-cmd --permanent --add-rich-rule='
  rule family="ipv4"
  source address="192.168.1.100"
  port protocol="tcp" port="22"
  accept'

# 重载
sudo firewall-cmd --reload
```

**效果：**
```
✅ 192.168.1.100 可以 SSH 连接
❌ 其他所有 IP 无法访问 SSH
```

**场景 3：使用 ufw (Ubuntu)**

```bash
# 启用 ufw
sudo ufw enable

# 默认策略：拒绝所有入站，允许所有出站
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 允许 SSH
sudo ufw allow 22/tcp

# 允许 HTTP 和 HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 查看状态
sudo ufw status verbose
```

**输出示例：**
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
```

#### 🧪 防火墙状态检查

**检查防火墙是否运行：**

```bash
# firewalld
sudo systemctl status firewalld
sudo firewall-cmd --state

# ufw
sudo ufw status

# iptables
sudo iptables -L -n -v
```

**查看所有规则：**

```bash
# firewalld
sudo firewall-cmd --list-all
sudo firewall-cmd --list-all-zones

# ufw
sudo ufw status numbered

# iptables
sudo iptables -L -n --line-numbers
```

#### 📊 防火墙工作流程图

```
                        互联网请求
                            |
                            ↓
                    ┌───────────────┐
                    │  防火墙检查    │
                    └───────────────┘
                            |
            ┌───────────────┼───────────────┐
            |                               |
    ┌───────▼────────┐              ┌──────▼──────┐
    │  允许的服务     │              │  拒绝的服务  │
    │  (白名单规则)   │              │  (默认策略)  │
    └───────┬────────┘              └──────┬──────┘
            |                               |
            ↓                               ↓
    ┌───────────────┐              ┌──────────────┐
    │  放行到服务器  │              │  丢弃/拒绝   │
    └───────────────┘              └──────────────┘
            |                               |
            ↓                               ↓
    服务正常响应                    记录到日志
```

#### ⚠️ 常见误区

**误区 1："我的服务器没人知道，不需要防火墙"**

❌ 错误！互联网上有大量自动化扫描工具，每天扫描所有 IP

✅ 正确做法：无论服务器大小，都应该启用防火墙

**误区 2："防火墙太麻烦，影响性能"**

❌ 现代防火墙性能影响极小，安全收益远大于性能损耗

✅ 正确做法：合理配置防火墙，既安全又不影响正常使用

**误区 3："云平台有安全组，不需要系统防火墙"**

❌ 安全组和系统防火墙是两层防护，都应该启用

✅ 正确做法：安全组+系统防火墙 = 双重保护

**误区 4："开了防火墙就绝对安全"**

❌ 防火墙只是安全措施之一，不是万能的

✅ 正确做法：防火墙 + SSH 密钥 + Fail2Ban + 及时更新 = 综合防御

#### 💡 最佳实践

**1. 默认拒绝策略**

```bash
# 默认拒绝所有入站连接
sudo ufw default deny incoming

# 只开放必要的端口
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
```

**2. 最小权限原则**

```
只开放你需要的服务，不要"以防万一"开一堆端口
```

**3. 定期审查规则**

```bash
# 每月检查一次防火墙规则
sudo firewall-cmd --list-all

# 删除不再需要的规则
sudo firewall-cmd --permanent --remove-service=mysql
sudo firewall-cmd --reload
```

**4. 结合其他安全措施**

```
防火墙（端口控制）
    +
SSH 密钥（身份认证）
    +
Fail2Ban（防暴力破解）
    +
定期更新（修补漏洞）
    =
完整的安全防护体系
```

#### 🎓 实用命令速查

**firewalld 常用命令：**

```bash
# 启动/停止/重启
sudo systemctl start firewalld
sudo systemctl stop firewalld
sudo systemctl restart firewalld

# 开机自启
sudo systemctl enable firewalld

# 查看状态
sudo firewall-cmd --state
sudo firewall-cmd --list-all

# 允许服务/端口
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-port=8080/tcp

# 删除规则
sudo firewall-cmd --permanent --remove-service=http
sudo firewall-cmd --permanent --remove-port=8080/tcp

# 重载规则（使修改生效）
sudo firewall-cmd --reload
```

**ufw 常用命令：**

```bash
# 启用/禁用
sudo ufw enable
sudo ufw disable

# 查看状态
sudo ufw status verbose
sudo ufw status numbered

# 允许/拒绝
sudo ufw allow 80/tcp
sudo ufw deny 3306/tcp

# 允许特定 IP
sudo ufw allow from 192.168.1.100

# 删除规则（通过编号）
sudo ufw delete 3

# 重置所有规则
sudo ufw reset
```

#### 📝 总结

**防火墙是什么？**
- 网络安全的守门员
- 根据规则过滤进出流量
- 只放行允许的连接，拦截其他一切

**开启防火墙的目的：**
1. ✅ 保护服务器免受未授权访问
2. ✅ 减少攻击面，最小化风险
3. ✅ 实现最小权限原则
4. ✅ 符合安全合规要求

**关闭防火墙的风险：**
1. ❌ 所有端口暴露
2. ❌ 易被扫描和攻击
3. ❌ 服务漏洞直接可利用
4. ❌ 不符合安全标准

**记忆口诀：**
- 防火墙是守卫，站在服务器门口
- 默认拒绝全部，只放必要的进来
- 生产环境必开，测试环境也别关
- 定期检查规则，过时的要删掉

**核心理念：**
> 防火墙不是万能的，但没有防火墙是万万不能的。它是服务器安全的基础，也是第一道防线。

---

## 10. 什么是代理和反向代理

#### 🎭 正向代理（Forward Proxy）：你的"代购"

**生活场景：**
想象你在国内想买一双只在美国发售的限量版球鞋，但商家不支持国际配送。这时你找了一个在美国的朋友帮你代购：
1. 你把钱给朋友（发送请求）
2. 朋友去美国商店买鞋（代理访问目标服务器）
3. 商店只知道是你朋友买的，不知道最终是给你的（隐藏真实客户端）
4. 朋友把鞋寄回给你（返回响应）

**技术场景：**
```
[你的电脑] → [代理服务器] → [目标网站]
  客户端        代理           服务器
                ↓
        服务器只看到代理的 IP
```

**核心特点：**
- **服务对象**：客户端（你）
- **隐藏对象**：客户端的真实 IP
- **目标服务器**：不知道真实客户端是谁

**实际应用：**
```bash
# 1. 科学上网工具（如 VPN）
你 → VPN 服务器 → Google
     ↑
   隐藏你的真实 IP

# 2. 公司内网访问外网
员工电脑 → 公司代理服务器 → 互联网
          ↑
        统一出口管理

# 3. 爬虫防封禁
爬虫 → 代理池（多个 IP）→ 目标网站
      ↑
    轮换 IP 避免被封
```

---

#### 🏪 反向代理（Reverse Proxy）：商场的"总服务台"

**生活场景：**
你去一个大型商场购物，商场有很多楼层和店铺：
1. 你走进商场大门（访问域名 www.mall.com）
2. 总服务台接待你，问你要买什么（反向代理接收请求）
3. 服务台根据需求把你引导到具体楼层：
   - 买衣服 → 3 楼服装区
   - 买电器 → 5 楼电器区
   - 吃饭 → 6 楼美食城
4. 你不需要知道后面有多少个仓库、多少个供应商（隐藏真实服务器）

**技术场景：**
```
[用户] → [反向代理 Nginx] → [服务器 A/B/C]
           www.example.com
                ↓
          /api      → 后端服务器 A (192.168.1.10)
          /static   → 静态资源服务器 B (192.168.1.20)
          /video    → 视频服务器 C (192.168.1.30)
```

**核心特点：**
- **服务对象**：服务器（后端）
- **隐藏对象**：真实服务器的地址和数量
- **客户端**：只知道反向代理的地址

**实际应用：**

**1. 负载均衡**
```nginx
# Nginx 配置示例
upstream backend {
    server 192.168.1.10;  # 服务器 1
    server 192.168.1.11;  # 服务器 2
    server 192.168.1.12;  # 服务器 3
}

server {
    listen 80;
    server_name www.example.com;
    
    location / {
        proxy_pass http://backend;
    }
}

# 用户访问 www.example.com
# Nginx 自动分配请求到 3 台服务器
# 就像餐厅有 3 个厨师，服务员按顺序分配订单
```

**2. 静态资源加速**
```nginx
location /static/ {
    proxy_cache my_cache;
    proxy_pass http://static_server;
}

# 反向代理缓存图片、CSS、JS
# 就像商场在门口设置热销商品展示区
# 不用每次都去仓库取货
```

**3. 安全隔离**
```
互联网 → [Nginx (公网)] → [应用服务器 (内网)]
           80.1.2.3          192.168.1.10
                              ↑
                          不暴露给公网
```

---

#### 🔄 两者对比

| 维度 | 正向代理 | 反向代理 |
|------|---------|---------|
| **服务对象** | 客户端 | 服务器 |
| **隐藏对象** | 客户端身份 | 服务器地址 |
| **位置** | 靠近客户端 | 靠近服务器 |
| **生活比喻** | 代购（替你买东西） | 总服务台（替商家接客） |
| **典型工具** | VPN、Shadowsocks | Nginx、HAProxy |
| **配置方** | 客户端配置 | 服务端配置 |

#### 🎬 经典案例

**正向代理案例：公司上网**
```
公司员工 → 公司代理 → 互联网
- 所有员工共用一个公网 IP
- IT 部门可以监控和过滤网站
- 外网看到的都是公司 IP
```

**反向代理案例：淘宝网**
```
你访问 www.taobao.com
         ↓
    [CDN/负载均衡]
         ↓
   ┌─────┼─────┐
   ↓     ↓     ↓
 服务器1 服务器2 服务器3
 
- 你不知道后面有多少台服务器
- 访问压力被智能分散
- 某台服务器挂了你也感觉不到
```

#### 💡 记忆口诀

- **正向代理**："我"请代理帮"我"办事（客户端视角）
- **反向代理**："服务器"请代理帮"服务器"接客（服务器视角）

---

## 11. Context 传递登录态的技术方案

#### 🎫 生活场景类比：游乐园的通行手环

想象你去迪士尼乐园：
1. **入园登录**：在门口刷身份证,工作人员给你戴上一个智能手环（登录获取 Token）
2. **园区游玩**：每次玩项目，刷手环就行，不用再次验证身份证（携带 Token 请求）
3. **信息传递**：手环信息通过闸机传给后台，后台知道你是谁、有什么权限（Context 传递）
4. **服务调用**：后台服务之间互相传递你的信息，确保全程服务一致（微服务间传递）

---

#### 🔐 完整技术流程

##### **第一步：用户登录获取凭证**

```javascript
// 前端：用户登录
async function login() {
  const response = await fetch('https://api.example.com/auth/login', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      username: 'zhangsan',
      password: 'abc123'
    })
  });
  
  const data = await response.json();
  // 返回：{ "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...", "userId": "10086" }
  
  // 存储 token（就像把手环戴在手腕上）
  localStorage.setItem('auth_token', data.token);
}
```

**服务端处理：**
```go
// 后端：验证用户名密码，生成 JWT Token
func Login(c *gin.Context) {
    var req LoginRequest
    c.BindJSON(&req)
    
    // 1. 验证用户名密码
    user := db.FindUser(req.Username, req.Password)
    
    // 2. 生成 JWT Token（打造专属手环）
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
        "user_id": user.ID,           // 10086
        "username": user.Username,     // "zhangsan"
        "role": "VIP",                 // 权限级别
        "exp": time.Now().Add(24*time.Hour).Unix(), // 过期时间
    })
    
    tokenString, _ := token.SignedString([]byte("secret_key"))
    
    c.JSON(200, gin.H{
        "token": tokenString,
        "userId": user.ID,
    })
}
```

**此时的 Token 内容解码后：**
```json
{
  "user_id": 10086,
  "username": "zhangsan",
  "role": "VIP",
  "exp": 1700000000
}
```

---

##### **第二步：用户发起业务请求（携带登录态）**

```javascript
// 前端：请求获取订单列表
async function getOrders() {
  const token = localStorage.getItem('auth_token');
  
  const response = await fetch('https://api.example.com/orders', {
    method: 'GET',
    headers: {
      // 🔑 关键：把 token 放在 Authorization 请求头中
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  });
  
  return await response.json();
}
```

**网络请求实际样子：**
```http
GET /orders HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMDA4Niwi...
Content-Type: application/json
```

---

##### **第三步：API Gateway 解析并创建 Context**

```go
// API 网关：接收请求，解析 Token，创建 Context
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 1. 从 Header 中提取 Token（检查手环）
        authHeader := c.GetHeader("Authorization")
        // "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
        
        tokenString := strings.TrimPrefix(authHeader, "Bearer ")
        
        // 2. 解析 Token（读取手环信息）
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            return []byte("secret_key"), nil
        })
        
        if err != nil || !token.Valid {
            c.JSON(401, gin.H{"error": "未授权"})
            c.Abort()
            return
        }
        
        // 3. 提取用户信息
        claims := token.Claims.(jwt.MapClaims)
        userID := int(claims["user_id"].(float64))      // 10086
        username := claims["username"].(string)          // "zhangsan"
        role := claims["role"].(string)                  // "VIP"
        
        // 4. 🎯 创建 Context，存入用户信息（核心步骤！）
        ctx := context.WithValue(c.Request.Context(), "user_id", userID)
        ctx = context.WithValue(ctx, "username", username)
        ctx = context.WithValue(ctx, "role", role)
        
        // 5. 替换请求的 Context
        c.Request = c.Request.WithContext(ctx)
        
        // 6. 继续处理后续逻辑
        c.Next()
    }
}
```

**此时 Context 中的数据：**
```
Context {
  "user_id": 10086,
  "username": "zhangsan",
  "role": "VIP"
}
```

---

##### **第四步：订单服务使用 Context**

```go
// 订单服务：从 Context 获取用户信息
func GetOrders(c *gin.Context) {
    // 1. 从 Context 中取出用户信息（读取手环信息）
    ctx := c.Request.Context()
    userID := ctx.Value("user_id").(int)        // 10086
    username := ctx.Value("username").(string)  // "zhangsan"
    
    // 2. 根据用户 ID 查询订单
    orders := db.Query("SELECT * FROM orders WHERE user_id = ?", userID)
    
    // 3. 记录日志
    log.Printf("用户 %s (ID: %d) 查询了订单", username, userID)
    
    c.JSON(200, orders)
}
```

---

##### **第五步：微服务间调用（Context 传递）**

假设订单服务需要调用库存服务：

```go
// 订单服务：调用库存服务
func CreateOrder(c *gin.Context) {
    ctx := c.Request.Context()
    userID := ctx.Value("user_id").(int)
    
    // 🔄 关键：调用其他微服务时传递 Context
    // 方式一：通过 HTTP Header 传递
    req, _ := http.NewRequestWithContext(ctx, "POST", 
        "http://inventory-service/check", body)
    
    // 手动添加用户信息到 Header（就像把手环信息写在纸条上）
    req.Header.Set("X-User-ID", strconv.Itoa(userID))
    req.Header.Set("X-Username", ctx.Value("username").(string))
    req.Header.Set("X-User-Role", ctx.Value("role").(string))
    
    client := &http.Client{}
    resp, _ := client.Do(req)
}
```

**实际的网络请求：**
```http
POST /check HTTP/1.1
Host: inventory-service
X-User-ID: 10086
X-Username: zhangsan
X-User-Role: VIP
Content-Type: application/json

{"product_id": 123, "quantity": 1}
```

---

##### **第六步：库存服务接收 Context**

```go
// 库存服务：接收并重建 Context
func CheckInventory(c *gin.Context) {
    // 1. 从 Header 中提取用户信息
    userID, _ := strconv.Atoi(c.GetHeader("X-User-ID"))
    username := c.GetHeader("X-Username")
    role := c.GetHeader("X-User-Role")
    
    // 2. 重建 Context（还原手环信息）
    ctx := context.WithValue(c.Request.Context(), "user_id", userID)
    ctx = context.WithValue(ctx, "username", username)
    ctx = context.WithValue(ctx, "role", role)
    
    // 3. 使用用户信息进行业务逻辑
    if role == "VIP" {
        // VIP 用户可以预定缺货商品
        log.Printf("VIP 用户 %s 正在检查库存", username)
    }
    
    // 4. 检查库存
    available := db.CheckStock(productID)
    c.JSON(200, gin.H{"available": available})
}
```

---

#### 📊 完整数据流向图

```
┌─────────────┐
│  用户浏览器  │
│ (zhangsan)  │
└──────┬──────┘
       │ 1. POST /login
       │    username: zhangsan
       │    password: abc123
       ↓
┌─────────────────────┐
│   认证服务          │
│  - 验证密码         │
│  - 生成 JWT Token   │
└──────┬──────────────┘
       │ 2. 返回 Token
       │    eyJhbGciOiJIUzI1NiIsInR...
       ↓
┌─────────────┐
│  用户浏览器  │
│ localStorage│ ← 存储 Token
└──────┬──────┘
       │ 3. GET /orders
       │    Header: Authorization: Bearer eyJhbGci...
       ↓
┌─────────────────────────────┐
│   API Gateway (中间件)       │
│  1. 解析 Token               │
│  2. 验证签名                 │
│  3. 提取用户信息             │
│     user_id: 10086          │
│     username: zhangsan      │
│     role: VIP               │
│  4. 创建 Context             │
│     ctx.Value("user_id")    │
└──────┬──────────────────────┘
       │ 4. Request + Context
       │    Context {
       │      user_id: 10086,
       │      username: zhangsan,
       │      role: VIP
       │    }
       ↓
┌─────────────────────┐
│   订单服务          │
│  - 从 Context 取值  │
│  - userID = 10086   │
│  - 查询订单         │
└──────┬──────────────┘
       │ 5. 调用库存服务
       │    POST /inventory/check
       │    Header: X-User-ID: 10086
       │            X-Username: zhangsan
       │            X-User-Role: VIP
       ↓
┌─────────────────────┐
│   库存服务          │
│  - 从 Header 重建   │
│    Context          │
│  - 检查库存         │
│  - VIP 用户优先     │
└─────────────────────┘
```

---

#### 🎯 核心技术要点

##### **1. Token 携带方式（3 种常见方案）**

**方案 A：Authorization Header（推荐）**
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**方案 B：Cookie**
```http
Cookie: session_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**方案 C：URL 参数（不推荐，不安全）**
```
GET /orders?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

##### **2. Context 传递的两种模式**

**模式一：单体应用内传递（同一进程）**
```go
// 直接通过 Go 的 context.Context 传递
ctx := context.WithValue(r.Context(), "user_id", 10086)
req = req.WithContext(ctx)

// 在函数调用链中传递
func A(ctx context.Context) {
    B(ctx)  // 直接传递
}

func B(ctx context.Context) {
    userID := ctx.Value("user_id")
}
```

**模式二：微服务间传递（跨进程/网络）**
```go
// 需要序列化到 HTTP Header
req.Header.Set("X-User-ID", "10086")
req.Header.Set("X-Username", "zhangsan")
req.Header.Set("X-Request-ID", "uuid-123")  // 链路追踪

// 或使用专业的链路追踪协议（如 OpenTelemetry）
req.Header.Set("traceparent", "00-trace-id-span-id-01")
```

##### **3. 安全性保障**

```go
// 1. Token 签名验证
token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
    // 检查签名算法
    if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
        return nil, fmt.Errorf("非法签名算法")
    }
    return []byte(secretKey), nil
})

// 2. 过期时间检查
claims := token.Claims.(jwt.MapClaims)
if exp, ok := claims["exp"].(float64); ok {
    if time.Now().Unix() > int64(exp) {
        return errors.New("Token 已过期")
    }
}

// 3. HTTPS 传输加密
// 所有请求必须通过 HTTPS，防止 Token 被窃听
```

---

#### 🔧 实战完整示例

```go
package main

import (
    "context"
    "github.com/gin-gonic/gin"
    "github.com/golang-jwt/jwt/v4"
    "net/http"
    "time"
)

// 中间件：解析 Token 并创建 Context
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        tokenString := c.GetHeader("Authorization")
        tokenString = strings.TrimPrefix(tokenString, "Bearer ")
        
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            return []byte("my-secret-key"), nil
        })
        
        if err != nil {
            c.JSON(401, gin.H{"error": "未授权"})
            c.Abort()
            return
        }
        
        claims := token.Claims.(jwt.MapClaims)
        
        // 创建 Context
        ctx := c.Request.Context()
        ctx = context.WithValue(ctx, "user_id", int(claims["user_id"].(float64)))
        ctx = context.WithValue(ctx, "username", claims["username"].(string))
        
        c.Request = c.Request.WithContext(ctx)
        c.Next()
    }
}

// 业务处理：使用 Context
func GetOrders(c *gin.Context) {
    ctx := c.Request.Context()
    userID := ctx.Value("user_id").(int)
    username := ctx.Value("username").(string)
    
    // 调用其他服务，传递 Context
    inventoryResp := callInventoryService(ctx, 123)
    
    c.JSON(200, gin.H{
        "user_id": userID,
        "username": username,
        "orders": []string{"订单1", "订单2"},
        "inventory": inventoryResp,
    })
}

// 调用其他微服务
func callInventoryService(ctx context.Context, productID int) map[string]interface{} {
    req, _ := http.NewRequestWithContext(ctx, "GET", 
        "http://inventory-service/product/123", nil)
    
    // 传递用户信息
    req.Header.Set("X-User-ID", fmt.Sprintf("%v", ctx.Value("user_id")))
    req.Header.Set("X-Username", fmt.Sprintf("%v", ctx.Value("username")))
    
    client := &http.Client{Timeout: 5 * time.Second}
    resp, _ := client.Do(req)
    
    var result map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&result)
    return result
}

func main() {
    r := gin.Default()
    
    // 需要认证的路由
    authorized := r.Group("/")
    authorized.Use(AuthMiddleware())
    {
        authorized.GET("/orders", GetOrders)
        authorized.GET("/profile", GetProfile)
    }
    
    r.Run(":8080")
}
```

---

#### 💡 关键记忆点

1. **Token 生成**：登录时，服务端生成包含用户信息的 JWT Token
2. **Token 携带**：客户端每次请求都在 `Authorization` Header 中带上 Token
3. **Token 解析**：API Gateway 中间件解析 Token，提取用户信息
4. **Context 创建**：将用户信息存入 `context.Context` 对象
5. **Context 传递**：在同一进程内直接传递，跨服务通过 HTTP Header 传递
6. **Context 使用**：业务代码从 Context 中读取用户信息，实现权限控制

**比喻总结：**
- Token = 游乐园手环（一次办理，全天使用）
- Context = 手环里的芯片数据（记录你的身份信息）
- 中间件 = 每个项目的闸机（读取手环，传递信息）
- 微服务调用 = 不同区域的工作人员传递你的信息

---

## 12. JWT 签名（signToken）与验签（VerifyToken）原理

#### 一、JWT 是什么

JWT（JSON Web Token）是一个**自包含的字符串**，格式为三段 Base64 用 `.` 拼接：

```
Header.Payload.Signature
```

示例：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9          ← Header（算法信息）
.eyJ1c2VyaWQiOiI1NTBlODQwMC4uLiIsInVzZXJuYW1lIjoiYWxpY2UiLCJleHAiOjE3NDc5OTk5OTl9  ← Payload（载荷数据）
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature（签名）
```

> **注意**：Payload 内容任何人都能 Base64 解码看到，但**无法伪造签名**（不知道密钥就算不出正确的 Signature）。因此不要把敏感信息（如密码）放入 Payload。

---

#### 二、signToken 签发原理

```go
func (s *Service) signToken(user *types.User) (string, error) {
    now := time.Now()
    claims := &Claims{
        UserID:   user.ID,       // 自定义载荷：用户 ID
        Username: user.Username, // 自定义载荷：用户名
        RegisteredClaims: jwt.RegisteredClaims{
            IssuedAt:  jwt.NewNumericDate(now),                                              // 签发时间
            ExpiresAt: jwt.NewNumericDate(now.Add(time.Duration(s.expireHour) * time.Hour)), // 过期时间
            Subject:   user.ID,                                                              // 主题（标准字段）
        },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(s.jwtSecret) // 用密钥签名，生成最终字符串
}
```

**三步过程：**

```
1. 组装 Claims（载荷）
   ┌─────────────────────────────────────────┐
   │ user_id:   "550e8400-..."               │
   │ username:  "alice"                      │
   │ iat:       1747900000  (签发时间戳)      │
   │ exp:       1748159200  (72小时后过期)    │
   │ sub:       "550e8400-..."               │
   └─────────────────────────────────────────┘

2. 用 HS256 算法 + 密钥 对 Header+Payload 做 HMAC 签名
   Signature = HMAC_SHA256(Base64(Header) + "." + Base64(Payload), jwtSecret)

3. 拼接输出
   Token = Base64(Header) + "." + Base64(Payload) + "." + Signature
```

**关键点**：`jwtSecret` 是只有服务器知道的密钥，存在配置文件的 `jwt.secret` 中。

---

#### 三、VerifyToken 验签原理

```go
func (s *Service) VerifyToken(tokenStr string) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenStr, &Claims{}, func(t *jwt.Token) (interface{}, error) {
        // 第一步：检查签名算法是否是预期的 HMAC（防止算法替换攻击）
        if _, ok := t.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("非预期的签名算法: %v", t.Header["alg"])
        }
        // 第二步：返回密钥，库用它来验签
        return s.jwtSecret, nil
    })
    // 第三步：验签通过后，库还会自动检查 exp 是否过期
    ...
    return claims, nil
}
```

**验证过程：**

```
收到 Token: "eyJ....eyJ....SflK..."
         ↓ 拆分
Header:    {"alg": "HS256", "typ": "JWT"}
Payload:   {"user_id": "550e...", "exp": 1748159200, ...}
Signature: "SflKxwRJ..."

         ↓ 重新计算签名
Expected = HMAC_SHA256(Header + "." + Payload, jwtSecret)

         ↓ 对比
Expected == Signature ?  → 签名合法
now < exp ?              → 未过期
         ↓
返回 Claims（包含 user_id、username）
```

> `jwt.ParseWithClaims` 库内部自动完成：签名验证 + 过期检查 + 格式校验，无需手动处理。

---

#### 四、实际使用完整流程

```
客户端                          HTTP 服务                    数据库

POST /api/auth/login
{"username":"alice","password":"xx"}
─────────────────────────────────────>
                                      ── 查用户、验 bcrypt 密码 ──────>
                                      <── 返回用户信息 ────────────────

                                      signToken(user)
                                      生成 JWT Token

{"token": "eyJ...", "user": {...}}
<─────────────────────────────────────

（客户端把 Token 存起来）

POST /api/sessions/xxx/chat
Authorization: Bearer eyJ...
─────────────────────────────────────>
                                      middleware.AuthMiddleware
                                      VerifyToken("eyJ...")
                                      ✅ 签名合法 + 未过期
                                      → 从 Claims 取出 user_id
                                      → 注入 ctx

                                      handler 里调用
                                      middleware.GetUserID(r)
                                      → "550e8400-..."

SSE 流式响应...
<─────────────────────────────────────
```

对应代码调用链：

```go
// 1. 登录时签发
user, token, err := authSvc.Login(req.Username, req.Password)
// token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyaWQiOi..."

// 2. 每次请求时，中间件自动验证（middleware/auth.go）
claims, err := authSvc.VerifyToken(tokenStr)
ctx = context.WithValue(ctx, ContextKeyUserID, claims.UserID)

// 3. Handler 里直接取用，不需要再查数据库
userID := middleware.GetUserID(r)  // "550e8400-..."
```

---

#### 五、JWT 的核心优势：无状态

| 方案 | 服务器存储 | 每次请求 | 横向扩容 |
|------|-----------|---------|---------|
| 传统 Session | 需要存 Redis/数据库 | 必须查一次存储 | 需要共享 Session 存储 |
| JWT | 不需要存储 | 只做数学运算（HMAC） | 天然支持，无状态 |

`VerifyToken` **只做数学运算**，不查数据库，服务器重启、横向扩容都不影响已有 Token 的验证。

---

#### 六、注意事项

1. **`jwtSecret` 必须足够复杂**，生产环境不能使用默认值，建议 32 位以上随机字符串。
2. **Token 无法主动吊销**：JWT 一旦签发，在过期前始终有效。如需强制下线，需配合黑名单（Redis 存储已吊销的 Token）。
3. **不要在 Payload 中存放敏感信息**：Payload 只是 Base64 编码，不是加密，任何人都能解码读取。
4. **算法检查不能省略**：`VerifyToken` 中必须检查 `t.Method.(*jwt.SigningMethodHMAC)`，防止攻击者将算法改为 `none` 绕过验签。

---

## 13. MCP 协议的两种传输方式（SSE Transport / Streamable HTTP）

#### 1. SSE Transport（/sse）

```
客户端                              服务端
  │                                   │
  │── GET /sse ──────────────────────>│  ① 建立 SSE 长连接（服务端推送通道）
  │<── SSE: endpoint=/message?sid=xx──│  ② 服务端返回一个 POST 端点 URL
  │                                   │
  │── POST /message?sid=xx ─────────>│  ③ 客户端发送 JSON-RPC 请求
  │<── SSE event: result ────────────│  ④ 服务端通过 SSE 长连接推送响应
  │                                   │
  │── POST /message?sid=xx ─────────>│  ⑤ 后续请求复用同一个 session
  │<── SSE event: result ────────────│
```

特点：
- 需要先 GET /sse 建立 SSE 长连接（有状态的 session）
- 请求和响应走不同的通道（POST 发请求，SSE 收响应）
- 服务端需要维护 session 状态（sid）
- 对负载均衡不友好（SSE 连接必须粘到同一个实例）

#### 2. Streamable HTTP（/mcp）

```
客户端                              服务端
  │                                   │
  │── POST /mcp ────────────────────>│  ① 直接发送 JSON-RPC 请求
  │<── 200 OK (JSON 或 SSE stream) ──│  ② 响应直接在同一个 HTTP 响应中返回
  │                                   │
  │── POST /mcp ────────────────────>│  ③ 每次请求都是独立的
  │<── 200 OK ───────────────────────│
```

特点：

- 无需建立长连接，每次 POST /mcp 就是一个完整的请求-响应
- 请求和响应走同一个 HTTP 连接
- 完全无状态，对负载均衡友好
- 响应可以是普通 JSON，也可以是 SSE 流（用于流式输出）

---

## 14. 服务器上编译基于 go 实现的后端代码并部署的流程

### 问题
如何在服务器上编译基于 go 实现的后端代码并部署？

### 解答

核心步骤概览：编译 -> 传输 -> 运行
部署的核心流程可以总结为三个步骤，这是所有方案的基础。

| 步骤 | 关键命令 / 操作 | 核心目的 |
| :--- | :--- | :--- |
| **1. 编译** | `GOOS=linux GOARCH=amd64 go build -o myapp` | 在你的 **本地电脑** 上，为 Linux 服务器编译出一个独立的、可直接运行的二进制文件 `myapp`。 |
| **2. 传输** | ```bash<br>scp myapp user@your_server_ip:/path/to/deploy<br>``` | 将生成的二进制文件上传到云服务器上你指定的目录（如 `/home/ubuntu/myapp`）。 |
| **3. 运行** | 通过 `systemd` 或 `Supervisor` 进行管理 | 让二进制文件在服务器后台持续、稳定地运行，并能在意外退出后自动重启。 |

#### 一、本地交叉编译

在电脑上（可以是 Windows、macOS、云服务器），打开终端，进入项目根目录，执行以下命令：

```bash
# 注意：将 'myapp' 替换为你期望的二进制文件名
# 如果你的程序入口文件不是 main.go，请相应替换
GOOS=linux GOARCH=amd64 go build -o myapp main.go
```

这会在当前目录生成一个名为 myapp 的Linux可执行文件。GOOS=linux 和 GOARCH=amd64 是Go交叉编译的关键参数，它们指定了目标操作系统和CPU架构。

注：如果是在云服务器上，需要你先将代码 pull 下来。

#### 二、传输二进制文件（如果是在云服务器上执行的步骤一，则此步骤跳过）

使用 scp 命令将本地编译好的二进制文件 myapp 上传到云服务器。请将 user、your_server_ip 和 /path/to/deploy 替换为实际值。

```bash
scp myapp user@your_server_ip:/path/to/deploy/
```

注：通常将打包后的文件放在 /opt 目录下，并遵循 "一个应用一个目录" 的"自包含"原则。即：所有相关文件（如可执行文件、配置、日志等）都放在一个独立子目录下，如 /opt/myapp。这使得应用的安装、升级和彻底删除都非常方便（适用于第三方商业软件、自己开发的应用服务）。

典型路径：/opt/myapp/bin/（可执行文件）、/opt/myapp/config/（配置文件）、/opt/myapp/logs/（日志文件）。

#### 三、配置 systemd 服务

登录云服务器，为你的应用创建一个 systemd 服务配置文件。

```bash
# 创建一个新的服务单元文件
sudo vim /etc/systemd/system/myapp.service
```

#### 四、编写服务单元文件

将以下内容粘贴到 myapp.service 文件中，并根据你的实际情况修改 WorkingDirectory、ExecStart、User 等字段。

```bash
[Unit]
Description=My Go Application
After=network.target   # 确保在网络服务启动后再启动本服务

[Service]
Type=simple
# 运行服务的系统用户，强烈不建议使用 root，建议单独创建如 'myapp' 的用户
User=ubuntu
WorkingDirectory=/path/to/deploy    # 你上传二进制文件的目录
ExecStart=/path/to/deploy/myapp     # 二进制文件的完整路径
Restart=always                      # 总是重启，保证服务稳定性
RestartSec=5                        # 5秒后尝试重启

# 可选：设置环境变量
Environment="APP_ENV=production"

# 可选：重定向标准输出和错误输出到文件
StandardOutput=file:/path/to/deploy/app.log
StandardError=file:/path/to/deploy/error.log

[Install]
WantedBy=multi-user.target
```

#### 五、启动并启用服务

```bash
# 1. 重新加载 systemd 的配置文件
sudo systemctl daemon-reload

# 2. 立即启动你的服务
sudo systemctl start myapp

# 3. 将服务设置为开机自启
sudo systemctl enable myapp

# 4. 查看服务运行状态和日志
sudo systemctl status myapp
journalctl -u myapp -f  # -f 参数可以实时跟踪日志输出
```

#### 进阶、用 Nginx 做反向代理

如果你的应用是一个 Web 服务，通常建议在前面加一层 Nginx 作为反向代理。这样做可以更方便地管理 SSL 证书（实现 HTTPS）、处理静态文件或做负载均衡。

1、安装 Nginx：sudo apt install nginx (Ubuntu/Debian)。

2、配置反向代理：创建一个新的 Nginx 配置文件 /etc/nginx/sites-available/myapp，内容如下：

```bash
server {
    listen 80;
    server_name your-domain.com;  # 替换为你的域名

    location / {
        # 将所有请求转发到本地的 Go 应用，假设它监听 8080 端口
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

3、启用配置并重启：

```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

---


## 15. 服务器上编译基于 vue 实现的前端代码并部署的流程

### 问题
如何在服务器上编译基于 vue 实现的前端代码并部署？

### 解答

### 一、环境准备

#### 1. 安装 Node.js（>= 18）

```bash
# 使用 nvm 安装（推荐）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 18
nvm use 18

# 验证版本
node -v   # 应输出 v18.x.x 或更高
npm -v
```

#### 2. 安装 Nginx（用于静态文件托管）

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install -y nginx

# CentOS/RHEL
sudo yum install -y nginx
```

---

### 二、获取源码

```bash
# 克隆项目到服务器
git clone <你的仓库地址> /home/user/code
cd /home/user/code

# 如果已有代码，拉取最新
cd /home/user/code
git pull origin main
```

---

### 三、安装依赖

```bash
cd /opt/agently-vue
npm install
```

> 如果服务器网络较慢，可使用国内镜像：
> ```bash
> npm install --registry=https://registry.npmmirror.com
> ```

---

### 四、修改生产环境配置（重要）

#### 4.1 修改 API 基础地址

编辑 `src/api.js`，将 `BASE_URL` 改为生产环境的后端地址：

```javascript
// 开发环境（本地）
// const BASE_URL = 'http://localhost:8081'

// 生产环境（根据实际部署情况修改）
const BASE_URL = ''  // 留空，通过 Nginx 反向代理转发 /api 请求
```

> **推荐方案**：将 `BASE_URL` 设为空字符串 `''`，让前端请求走相对路径 `/api/...`，由 Nginx 统一反向代理到后端服务。这样无需暴露后端端口，也避免跨域问题。

#### 4.2 （可选）使用环境变量

也可以通过 Vite 环境变量实现：

```bash
# 创建 .env.production 文件
echo 'VITE_API_BASE_URL=' > .env.production
```

然后修改 `src/api.js`：
```javascript
const BASE_URL = import.meta.env.VITE_API_BASE_URL || ''
```

---

### 五、编译构建

```bash
cd /opt/agently-vue
npm run build
```

构建成功后，产物输出到 `dist/` 目录：

```
dist/
├── index.html
├── favicon.svg
└── assets/
    ├── index-xxxxx.js      # 打包后的 JS
    └── index-xxxxx.css     # 打包后的 CSS
```

> 构建过程中如果报错，常见原因：
> - Node.js 版本过低 → 升级到 18+
> - 依赖未安装 → 重新执行 `npm install`
> - 内存不足 → 增加 Node 内存：`NODE_OPTIONS=--max-old-space-size=4096 npm run build`

---

### 六、部署到 Nginx

#### 6.1 复制构建产物

```bash
# 创建部署目录
sudo mkdir -p /var/www/agently-vue

# 复制构建产物
sudo cp -r /opt/agently-vue/dist/* /var/www/agently-vue/

# 设置权限
sudo chown -R www-data:www-data /var/www/agently-vue
```

#### 6.2 配置 Nginx

创建 Nginx 配置文件：

```bash
sudo vim /etc/nginx/sites-available/agently-vue
```

写入以下内容：

```nginx
server {
    listen 80;
    server_name your-domain.com;  # 替换为你的域名或服务器 IP

    root /var/www/agently-vue;
    index index.html;

    # 前端路由 - SPA 单页应用支持
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API 反向代理 - 转发到后端服务
    location /api/ {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # SSE 流式响应支持（重要）
        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 300s;
        proxy_set_header Connection '';
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
    }

    # 静态资源缓存
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Gzip 压缩
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml;
    gzip_min_length 1024;
}
```

#### 6.3 启用站点并重启 Nginx

```bash
# 创建软链接启用站点
sudo ln -sf /etc/nginx/sites-available/agently-vue /etc/nginx/sites-enabled/

# 测试配置是否正确
sudo nginx -t

# 重启 Nginx
sudo systemctl restart nginx

# 设置开机自启
sudo systemctl enable nginx
```

---

### 七、验证部署

```bash
# 检查 Nginx 是否正常运行
sudo systemctl status nginx

# 测试前端页面
curl -I http://your-domain.com

# 测试 API 代理是否正常
curl http://your-domain.com/api/agents
```

浏览器访问 `http://your-domain.com` 即可看到前端界面。

---

### 八、HTTPS 配置（推荐）

使用 Let's Encrypt 免费证书：

```bash
# 安装 certbot
sudo apt install -y certbot python3-certbot-nginx

# 自动申请并配置证书
sudo certbot --nginx -d your-domain.com

# 证书自动续期（certbot 会自动添加定时任务）
sudo certbot renew --dry-run
```

---

### 九、更新部署（后续迭代）

每次代码更新后，执行以下步骤：

```bash
cd /opt/agently-vue

# 1. 拉取最新代码
git pull origin main

# 2. 安装依赖（如有新增）
npm install

# 3. 重新构建
npm run build

# 4. 替换部署文件
sudo rm -rf /var/www/agently-vue/*
sudo cp -r dist/* /var/www/agently-vue/

# Nginx 无需重启（静态文件直接生效）
```

> 可将以上步骤写成脚本 `deploy.sh` 实现一键部署。




---

### 十、一键部署脚本（可选）

创建 `/opt/agently-vue/deploy.sh`：

```bash
#!/bin/bash
set -e

echo "=== Agently Vue 部署脚本 ==="

cd /opt/agently-vue

echo "[1/4] 拉取最新代码..."
git pull origin main

echo "[2/4] 安装依赖..."
npm install --registry=https://registry.npmmirror.com

echo "[3/4] 构建生产版本..."
npm run build

echo "[4/4] 部署到 Nginx..."
sudo rm -rf /var/www/agently-vue/*
sudo cp -r dist/* /var/www/agently-vue/

echo "=== 部署完成！==="
echo "访问地址: http://your-domain.com"
```

```bash
chmod +x deploy.sh
./deploy.sh
```

---

### 常见问题

| 问题 | 解决方案 |
|------|----------|
| 页面空白 | 检查 Nginx root 路径是否正确，`dist/` 中是否有 `index.html` |
| API 请求 404 | 检查 Nginx 的 `/api/` 代理配置，确认后端服务（端口 8081）正在运行 |
| SSE 流式不工作 | 确认 Nginx 配置了 `proxy_buffering off` |
| 刷新页面 404 | 确认 `try_files $uri $uri/ /index.html` 配置正确 |
| 构建内存不足 | 使用 `NODE_OPTIONS=--max-old-space-size=4096 npm run build` |
| 权限问题 | 确认 `/var/www/agently-vue` 目录属主为 `www-data` |

---

## 16. PostgreSQL 安装与配置（CentOS 8）

### 问题
如何在 CentOS 8 上安装、初始化并配置 PostgreSQL，使其支持远程连接？

### 解答

#### 一、安装 PostgreSQL

**1. 添加官方 YUM 仓库**

```bash
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

**2. 禁用系统自带的 PostgreSQL 模块**

```bash
sudo dnf -qy module disable postgresql
```

> **注意**：输入 sudo dnf -qy module disable postgresql 如果输出下面的内容：
```
Importing GPG key 0x08B40D20:
 Userid     : "PostgreSQL RPM Repository <pgsql-pkg-yum@lists.postgresql.org>"
 Fingerprint: D4BF 08AE 67A0 B4C7 A1DB CCD2 40BC A2B4 08B4 0D20
 From       : /etc/pki/rpm-gpg/PGDG-RPM-GPG-KEY-RHEL
```
这是正常现象！系统在导入 PostgreSQL 官方 GPG 签名密钥，这是验证软件包真实性的安全步骤。
如果命令一直卡住没有提示输入，按 Ctrl + C 中断，然后手动先导入 GPG 密钥再执行：
```
# 手动导入 GPG 密钥
sudo rpm --import /etc/pki/rpm-gpg/PGDG-RPM-GPG-KEY-RHEL

# 再执行禁用模块命令
sudo dnf -qy module disable postgresql
```

> 如果执行时出现的提示上让你确认导入 GPG Key（“Is this ok [y/N]: y”），输入 `y` 确认即可

**3. 安装服务端（以 16 版本为例）**

```bash
sudo dnf install -y postgresql16-server
```

**4. 初始化数据库**

```bash
sudo /usr/pgsql-16/bin/postgresql-16-setup initdb
```

**5. 启动并设置开机自启**

```bash
sudo systemctl enable postgresql-16
sudo systemctl start postgresql-16

# 验证
sudo systemctl status postgresql-16
```

---

#### 二、基本配置

**1. 设置 postgres 超级用户密码**

```bash
sudo -i -u postgres
psql

-- 设置密码
ALTER USER postgres WITH PASSWORD '你的强密码';

\q
exit
```

**2. 配置监听地址（postgresql.conf）**

```bash
sudo vim /var/lib/pgsql/16/data/postgresql.conf
```

修改以下内容：

```
listen_addresses = '*'   # 监听所有网卡（默认仅本地）
port = 5432              # 默认端口，可按需修改
```

**3. 配置客户端认证（pg_hba.conf）**

```bash
sudo vim /var/lib/pgsql/16/data/pg_hba.conf
```

文件中已默认包含本地连接规则，**无需重复添加**，只在末尾追加远程访问规则：

```
# 允许指定公网 IP 段远程连接（按实际网段修改）
host    all    all    219.152.236.0/24    scram-sha-256
```

> **CIDR 写法说明：**
>
> | 写法 | 含义 |
> |------|------|
> | `219.152.236.0/24` | 允许 `219.152.236.0` ~ `219.152.236.255` 整个网段 |
> | `219.152.236.100/32` | 只允许单个 IP `219.152.236.100` |
> | `0.0.0.0/0` | 允许所有远程 IP（不推荐用于生产环境） |

**4. 开放防火墙端口**

```bash
sudo firewall-cmd --permanent --add-port=5432/tcp
sudo firewall-cmd --reload

# 验证
sudo firewall-cmd --list-ports
```

**5. 重新加载配置使其生效**

```bash
# 使用 reload 而非 restart，不中断现有连接
sudo systemctl reload postgresql-16
```

---

#### 三、创建业务数据库和用户（推荐）

不建议直接使用 `postgres` 超级用户，应为每个应用创建独立用户：

```bash
sudo -i -u postgres
psql
```

```sql
-- 创建用户
CREATE USER appuser WITH PASSWORD 'apppassword';

-- 创建数据库
CREATE DATABASE appdb OWNER appuser;

-- 授权
GRANT ALL PRIVILEGES ON DATABASE appdb TO appuser;

\q
```

---

#### 四、验证连接

```bash
# 本地连接测试
psql -U appuser -d appdb -h 127.0.0.1 -W

# 查看所有数据库
\l

# 查看所有用户
\du
```

---


---

## 17. pgvector 扩展安装（CentOS 8 + PostgreSQL 16）

### 背景

pgvector 是 PostgreSQL 的向量相似度搜索扩展，用于支持 RAG（检索增强生成）等 AI 场景。在 CentOS 8 + PostgreSQL 16 环境下安装需要从源码编译，过程中会遇到若干依赖问题，以下为完整流程及问题处理记录。

---

### 一、安装编译工具

```bash
sudo dnf install -y gcc make git
```

---

### 二、安装 postgresql16-devel（遇到依赖问题的处理）

#### 问题一：直接安装报依赖缺失

```bash
sudo dnf install -y postgresql16-devel
# 报错：nothing provides clang-devel >= 19.0 / llvm-devel >= 19.0 / perl(IPC::Run)
```

#### 问题二：`--nobest` 和 `--skip-broken` 均无效

所有历史版本都依赖 `perl(IPC::Run)`，而该包在 CentOS 8 默认源中不存在：

```bash
sudo dnf install -y perl-IPC-Run
# 报错：No match for argument: perl-IPC-Run
```

#### 解决方案：跳过 devel 包，直接验证头文件是否已存在

PostgreSQL 16 server 包安装时已附带了头文件，无需单独安装 devel 包：

```bash
# 验证头文件是否存在
ls /usr/pgsql-16/include/server/
```

如果输出了大量 `.h` 文件（如 `postgres.h`、`fmgr.h` 等），说明头文件已就绪，**直接跳到编译步骤**。

> **结论：** `postgresql16-devel` 包在 CentOS 8 上因依赖问题无法安装，但头文件已随 `postgresql16-server` 一并安装，编译 pgvector 不受影响。

---

### 三、编译安装 pgvector

```bash
# 克隆源码
git clone https://github.com/pgvector/pgvector.git
cd pgvector

# 编译
make PG_CONFIG=/usr/pgsql-16/bin/pg_config

# 安装
sudo make install PG_CONFIG=/usr/pgsql-16/bin/pg_config
```

#### 问题三：编译时报 `redhat-hardened-cc1: No such file or directory`

```
gcc: error: /usr/lib/rpm/redhat/redhat-hardened-cc1: No such file or directory
```

**原因：** 缺少 `redhat-rpm-config` 包，该包提供 `/usr/lib/rpm/redhat/` 下的编译规格文件。

**解决：**

```bash
sudo dnf install -y redhat-rpm-config
# 然后重新执行 make
make PG_CONFIG=/usr/pgsql-16/bin/pg_config
sudo make install PG_CONFIG=/usr/pgsql-16/bin/pg_config
```

#### 问题四：`make install` 最后报 `llvm-lto: No such file or directory`

```
/bin/sh: /usr/lib64/llvm20/bin/llvm-lto: No such file or directory
make: *** [...] Error 127
```

**原因：** 这是 LLVM JIT 优化的链接步骤，与 pgvector 核心功能无关。此时 `vector.so` 已经安装成功。

**结论：** 此错误**可以忽略**，pgvector 功能完整可用。如需消除该错误可安装 `llvm-toolset`，但非必须。

---

### 四、在数据库中启用扩展

#### 问题五：普通用户执行报权限不足

```sql
-- 以普通用户连接时执行
CREATE EXTENSION IF NOT EXISTS vector;
-- 报错：ERROR: permission denied to create extension "vector"
-- HINT: Must be superuser to create this extension.
```

**解决：** 必须使用 `postgres` 超级用户执行：

```bash
sudo -u postgres psql -d agentlydb -c "CREATE EXTENSION IF NOT EXISTS vector;"
```

或切换用户后执行：

```bash
sudo -u postgres psql
```

```sql
\c agentlydb
CREATE EXTENSION IF NOT EXISTS vector;
-- 输出：CREATE EXTENSION
```

> **说明：** 扩展只需超级用户**创建一次**，之后所有普通用户均可正常使用 `vector` 类型，无需额外授权。

---

### 五、验证安装成功

```sql
SELECT extname, extversion FROM pg_extension WHERE extname = 'vector';
-- 返回：vector | 0.8.2
```

---

### 六、完整流程总结

```bash
# 1. 安装编译工具
sudo dnf install -y gcc make git redhat-rpm-config

# 2. 验证头文件存在（无需安装 devel 包）
ls /usr/pgsql-16/include/server/

# 3. 编译安装 pgvector
git clone https://github.com/pgvector/pgvector.git
cd pgvector
make PG_CONFIG=/usr/pgsql-16/bin/pg_config
sudo make install PG_CONFIG=/usr/pgsql-16/bin/pg_config
# 最后的 llvm-lto 报错可忽略

# 4. 以超级用户启用扩展
sudo -u postgres psql -d 你的数据库名 -c "CREATE EXTENSION IF NOT EXISTS vector;"

# 5. 验证
sudo -u postgres psql -d 你的数据库名 -c "SELECT extname, extversion FROM pg_extension WHERE extname = 'vector';"
```

---

## 18. Nginx proxy_pass 末尾斜杠的差别（反代路径改写规则）

### 问题

在 Nginx 配置中，下面两种写法的区别是什么？为什么仅仅末尾差一个斜杠，就会让后端返回 `404 page not found`？

```nginx
# 写法 A（不带 URI）
location /api/agently/ {
    proxy_pass http://127.0.0.1:8081;
}

# 写法 B（带 URI，哪怕只有一个 /）
location /api/agently/ {
    proxy_pass http://127.0.0.1:8081/;
}
```

### 真实踩坑现场

某次生产环境部署后，前端调用登录接口失败：

```
POST https://domain/api/agently/auth/login  →  404 Not Found
响应体：404 page not found
```

排查链路：

1. **响应体是 `404 page not found`** —— 这是 Go `net/http.ServeMux` 的默认 404 文案，Nginx 默认 404 页是 HTML，不可能输出这串纯文本。说明请求**已经到了后端**，只是路径没命中。
2. 直接打后端 `curl http://127.0.0.1:8081/api/agently/auth/login` —— 返回正常的业务 401（参数缺失），证明后端路由完全正常。
3. 翻看 Nginx 配置，发现 `proxy_pass` 末尾**多了一个 `/`**。
4. 去掉斜杠后，问题立即修复。

根因就是本文要讲的：**`proxy_pass` 末尾的 `/` 不是格式问题，是语义开关**。

### 一、核心规则（一句话记住）

> `proxy_pass` 后面的 URL **是否带 URI 部分**（哪怕只有一个 `/`），决定了 Nginx 用**两种完全不同的算法**构造转发给后端的路径。

判断"带不带 URI"看的是 host:port 后面有没有任何东西，**`/` 也算 URI**。

| 模式 | `proxy_pass` 写法 | 行为 |
|---|---|---|
| **不带 URI** | `proxy_pass http://127.0.0.1:8081;` | **原样透传** 完整请求 URI |
| **带 URI** | `proxy_pass http://127.0.0.1:8081/;` | **替换** location 匹配到的前缀，再拼接剩余部分 |

### 二、用真实场景对比

假设请求是 `POST /api/agently/auth/login`，匹配的 location 是 `/api/agently/`。

#### 写法 A：不带 URI（推荐 ✅）

```nginx
location /api/agently/ {
    proxy_pass http://127.0.0.1:8081;
}
```

后端收到：

```
POST /api/agently/auth/login        ← 完整路径原样透传
```

后端 Go `ServeMux` 注册的就是 `/api/agently/auth/login`，**完美命中**。

#### 写法 B：带 URI（哪怕只有 `/`）❌

```nginx
location /api/agently/ {
    proxy_pass http://127.0.0.1:8081/;
}
```

Nginx 的处理算法：

1. 拿到原始 URI：`/api/agently/auth/login`
2. **剥掉** location 匹配的前缀 `/api/agently/`，剩余部分：`auth/login`
3. 把 `proxy_pass` 中的 URI 部分（`/`）和剩余部分拼起来：`/` + `auth/login` = `/auth/login`

后端收到：

```
POST /auth/login                    ← 前缀被剥掉了！
```

后端没注册这个路径 → `ServeMux` 返回 `404 page not found`。

### 三、更多组合举例（彻底搞懂）

假设请求都是 `GET /api/agently/auth/login`：

| location | proxy_pass | 后端实际收到 | 说明 |
|---|---|---|---|
| `/api/agently/` | `http://backend` | `/api/agently/auth/login` | 原样透传 |
| `/api/agently/` | `http://backend/` | `/auth/login` | 剥前缀，拼 `/` |
| `/api/agently/` | `http://backend/v1` | `/v1auth/login` ⚠️ | 拼接时不补斜杠，**经典 bug** |
| `/api/agently/` | `http://backend/v1/` | `/v1/auth/login` | 把前缀替换成 `/v1/` |
| `/api/agently/` | `http://backend/api/` | `/api/auth/login` | 把 `/api/agently/` 替换成 `/api/` |
| `/api/agently`（无尾斜杠） | `http://backend/` | `/auth/login` | 同样剥前缀 |

> 📌 **重点**：带 URI 模式 = "**字符串替换**"，把 location 的前缀当作"被替换的部分"，把 `proxy_pass` 的 URI 当作"替换为的部分"。

### 四、为什么 Nginx 要设计这两种模式？

各有用途：

#### 用途 1：纯反向代理（写法 A）

后端就想接收完整路径，前后端路径一致。**99% 的微服务场景都用这个**。

#### 用途 2：路径改写（写法 B）

把对外路径和内部路径解耦：

```nginx
# 对外 /api/v2/users/123，内部其实是 /users/123
location /api/v2/ {
    proxy_pass http://user-service/;
}
```

或反过来给内部服务加上版本前缀：

```nginx
location /api/ {
    proxy_pass http://backend/v1/api/;
}
```

### 五、特殊情况：正则 location **必须不带 URI**

```nginx
# ❌ Nginx 会启动失败
# nginx: [emerg] "proxy_pass" cannot have URI part in location given by regular expression
location ~ ^/api/ {
    proxy_pass http://backend/;
}

# ✅ 正确写法
location ~ ^/api/ {
    proxy_pass http://backend;
}
```

正则 location 由于无法静态确定"前缀有多长"，Nginx 直接禁止带 URI 的写法。

### 六、如何快速判断当前线上 404 是不是这个问题

排查步骤（按顺序执行）：

```bash
# ① 看 404 来源（response body 形态）
curl -i -X POST https://domain/api/xxx/auth/login \
  -H 'Content-Type: application/json' -d '{}'
# - body 是 HTML 报错页 → Nginx 自己返回的 404
# - body 是 "404 page not found" 纯文本 → Go net/http 返回的 → 路径已穿透到后端，是路径改写问题

# ② 直接打后端，绕过 Nginx
curl -i -X POST http://127.0.0.1:8081/api/xxx/auth/login \
  -H 'Content-Type: application/json' -d '{}'
# - 返回正常业务响应 → 100% 是 Nginx proxy_pass 路径改写问题
# - 后端也 404 → 路由没注册 / 二进制版本不对

# ③ 查 Nginx 实际生效配置
sudo nginx -T 2>/dev/null | grep -B2 -A10 "your-prefix"
# 看 proxy_pass 末尾是否多了个 /
```

### 七、口诀（建议背下来）

> **要原样透传 → `proxy_pass` 不带 URI（包括末尾的 `/`）**
> **要改写路径 → `proxy_pass` 末尾补全 URI 前缀**
>
> 末尾 `/` 不是"美观问题"，是"语义开关"。

### 八、和路由风格的配套关系

两种风格都行，但**必须配套**，错配就会 404：

| 后端路由风格 | 推荐 location | 推荐 proxy_pass |
|---|---|---|
| 后端代码里写**全路径**（`/api/agently/auth/login`） | `location /api/agently/` | `proxy_pass http://backend;`（不带 `/`） |
| 后端代码里只写**短路径**（`/auth/login`），前缀让网关剥掉 | `location /api/agently/` | `proxy_pass http://backend/;`（带 `/`） |

> 当前 Agently 项目的后端注册的是全路径（`mux.HandleFunc("/api/agently/auth/login", ...)`），所以**必须用不带 `/` 的写法**。

### 九、结论速查表

| 现象 | 大概率原因 |
|---|---|
| 浏览器 404，body 是 HTML | Nginx 自己 404，看 location 是否匹配 |
| 浏览器 404，body 是 `404 page not found` 纯文本 | 请求到了 Go 后端但路径错，**99% 是 `proxy_pass` 末尾 `/` 问题** |
| `nginx -t` 报 `cannot have URI part` | 正则 location 配了带 URI 的 `proxy_pass` |
| 拼接出 `/v1auth/login` 这种粘连路径 | `proxy_pass` 的 URI 没以 `/` 结尾 |

---

## 19. 多后端服务共存时的 URL 路径分层与网关前缀最佳实践

### 背景

同一个域名下挂着多个后端服务（例如 `news` 服务监听 `:8083`，`gold` 服务监听 `:8082`），前端通过 nginx 反向代理统一接入时，会发现浏览器实际请求的 URL 形如：

```
https://domain/hwan/api/news/v1/by_date
https://domain/hwan/api/gold/v1/volume_oi
```

但 proto 中定义的 HTTP 路由只是 `/v1/by_date`、`/v1/volume_oi`。这中间多出来的 `/api/news`、`/api/gold` 是哪一层加上去的？要不要去掉？这是一个非常常见的疑问，也是微服务网关分层设计的核心知识点。

---

### 一、URL 前缀到底是谁加的？

**结论：是前端 axios 的 `baseURL` 加的，不是 nginx**。

完整链路：

```
前端业务代码：newsHttp.post('/v1/by_date', ...)
     ↓ axios 实例 baseURL = '/api/news'
浏览器实际请求：https://domain/hwan/api/news/v1/by_date
     ↓ nginx 反向代理（按前缀分流到不同上游服务）
     ↓ 转发前 strip 掉 /api/news 前缀
后端实际收到：/v1/by_date    ← 这才是 proto 定义的路径
```

所以三段路径各司其职：

| 路径段 | 出现位置 | 作用 |
|---|---|---|
| `/hwan/` | vite `base` 配置 | 前端 SPA 部署在 nginx 子路径下，用于静态资源前缀 |
| `/api/news/`、`/api/gold/` | 前端 axios `baseURL` | **服务路由标签**，让 nginx 知道该转发给哪个上游服务 |
| `/v1/by_date` | proto `option (google.api.http)` | **真正的业务路径**，后端服务自己定义 |

---

### 二、为什么要这么分层？

**核心原因：同一个域名下有多个后端服务，nginx 必须有一种办法把请求分流到不同上游。**

两种分流策略：

**策略 A：路径前缀分流（当前方案，推荐）**
- 同一域名 `domain.com`
- 用 `/api/news/`、`/api/gold/` 这类前缀区分
- 优点：单域名、单证书、单 nginx server 块、无跨域问题
- 缺点：URL 看起来比"裸" `/v1/by_date` 多一层

**策略 B：子域名分流**
- `news.domain.com/v1/by_date`、`gold.domain.com/v1/volume_oi`
- 优点：URL 干净
- 缺点：要多搞 DNS、证书；需处理跨域（CORS 或 cookie domain）

绝大多数场景应优先选 **策略 A**。

---

### 三、能不能让浏览器直接请求 `https://domain/v1/by_date`？

可以，但要付出代价。下面是用户在排查时提出过的一个错误方案：

> 1. proto 把路径改成 `/api/news/by_date`
> 2. 前端 axios 不再加前缀
> 3. nginx 保持原有 `rewrite ^/api/news/(.*)$ /$1 break` 不变

**这个方案错在第 3 步**：

```
浏览器:   /api/news/by_date
nginx:    rewrite → /by_date           ← 前缀被吃掉
后端收到: /by_date
proto 注册的: /api/news/by_date         ← 不匹配 → 404
```

**正确做法是把 nginx 的 `rewrite` 删掉，让请求原样透传**：

```nginx
location ^~ /api/news/ {
    proxy_pass http://127.0.0.1:8083;     # ← 不带尾斜杠 = 原样透传
    # ...
}
```

⚠️ nginx 的 `proxy_pass` 行为陷阱（**极易踩坑**）：

| 写法 | 行为 |
|---|---|
| `proxy_pass http://127.0.0.1:8083;` | **不带尾斜杠** → 原样转发整个 URI |
| `proxy_pass http://127.0.0.1:8083/;` | **带尾斜杠** → 隐式 strip 掉 location 前缀（等价于 rewrite） |

但即使技术上可行，**也不推荐让 proto 感知 `/api/news` 这种网关前缀**，因为：

- proto 不该知道自己被挂在什么前缀下，会让"业务路径"和"网关分流前缀"耦合。
- 将来运维想把前缀从 `/api/news` 改成 `/svc/news`，得跟着改 proto 重新发版。
- 丢掉 `/v1` 也不利于将来 API 版本演进。

---

### 四、最佳实践（保持 proto 干净，三层各司其职）

#### 4.1 proto 只关心业务路径

```proto
service NewsSummaryService {
  rpc GetNewsByDate(GetNewsByDateRequest) returns (GetNewsByDateResponse) {
    option (google.api.http) = {
      post: "/v1/by_date"
      body: "*"
    };
  }
}
```

**禁忌**：路径里**不要**出现 `/api/news/`、`/news/` 这种网关前缀或服务名。

#### 4.2 nginx：用 `proxy_pass /` 自动 strip 前缀（不要用 rewrite）

```nginx
upstream news_backend { server 127.0.0.1:8083; keepalive 32; }
upstream gold_backend { server 127.0.0.1:8082; keepalive 32; }

server {
    listen 443 ssl http2;
    server_name domain.com;

    # 前端 SPA：部署在 /hwan/ 子路径
    location /hwan/ {
        alias /var/www/hwan-vue/dist/;
        try_files $uri $uri/ /hwan/index.html;
    }

    # news 服务反代（proxy_pass 末尾带 / 自动 strip /api/news 前缀）
    location ^~ /api/news/ {
        proxy_pass http://news_backend/;

        proxy_http_version 1.1;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 5s;
        proxy_send_timeout    60s;
        proxy_read_timeout    60s;

        # 如果有 SSE / 长轮询接口再加：
        # proxy_buffering off;
        # proxy_cache     off;
    }

    # gold 服务反代
    location ^~ /api/gold/ {
        proxy_pass http://gold_backend/;
        # ... 同上的 proxy_set_header 与超时
    }
}
```

要点：

| 点 | 做法 | 原因 |
|---|---|---|
| strip 前缀 | 用 `proxy_pass http://upstream/;`（**带尾斜杠**） | nginx 惯用法，比 `rewrite ... break;` 更简洁 |
| 不要混用 | **不要同时**用 `rewrite` + 带尾斜杠 `proxy_pass` | 行为容易出意外 |
| `location` 用 `^~` | 抢占式精确匹配前缀 | 性能更好、避免被正则 location 抢走 |
| 抽 `upstream` | 多实例时方便加负载均衡和健康检查 | 易维护 |
| 共用配置抽 snippet | 把 `proxy_set_header` 抽到 `/etc/nginx/snippets/proxy-common.conf` | 新增服务一行 `location` 就能搞定 |

##### 4.2.1 共用配置抽 snippet 详解

上面表格里"共用配置抽 snippet"是 **nginx 工程化技巧**，本质是用 `include` 指令把多个 `location` 里**重复的指令**抽到独立文件，再在每个 `location` 里一行 `include` 引入。类似编程里的"函数提取"。

**问题**：原配置里每个 location 都要复制粘贴这一坨：

```nginx
proxy_http_version 1.1;
proxy_set_header Host              $host;
proxy_set_header X-Real-IP         $remote_addr;
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_connect_timeout 5s;
proxy_send_timeout    60s;
proxy_read_timeout    60s;
```

新增服务要复制 8 行；将来想给所有服务加一个新 header（如 `X-Request-Id`），得改 N 处，**漏改一处就配置漂移**。

**做法**：

**步骤 1**：建公共配置文件（**只放公共指令，不放 `proxy_pass`**，因为每个服务上游不同）：

```bash
sudo mkdir -p /etc/nginx/snippets
sudo vim   /etc/nginx/snippets/proxy-common.conf
```

```nginx
# /etc/nginx/snippets/proxy-common.conf
proxy_http_version 1.1;

proxy_set_header Host              $host;
proxy_set_header X-Real-IP         $remote_addr;
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;

proxy_connect_timeout 5s;
proxy_send_timeout    60s;
proxy_read_timeout    60s;
```

**步骤 2**：主配置里 `include` 引用：

```nginx
location ^~ /api/news/ {
    proxy_pass http://news_backend/;
    include /etc/nginx/snippets/proxy-common.conf;
}

location ^~ /api/gold/ {
    proxy_pass http://gold_backend/;
    include /etc/nginx/snippets/proxy-common.conf;
}
```

**步骤 3**：将来加新服务真就两行核心配置：

```nginx
location ^~ /api/report/ {
    proxy_pass http://report_backend/;
    include /etc/nginx/snippets/proxy-common.conf;
}
```

**`include` 的工作原理**：nginx 在 reload 时**原地把 include 那一行替换为目标文件的内容**再解析，**性能上没有任何损耗**（不是运行时 include），可以放心使用。

**约定俗成的存放路径**：

| 路径 | 说明 |
|---|---|
| `/etc/nginx/snippets/` | Debian/Ubuntu 默认推荐路径，nginx 自带的 `snakeoil.conf` 等也在这里；CentOS 自己 `mkdir` 即可 |
| `/etc/nginx/conf.d/_partials/` | 也有团队这么放，下划线开头是为了避免被 `conf.d/*.conf` 通配匹配 |

**什么该抽 / 什么不该抽**：

| ✅ 应该抽 | ❌ 不该抽 |
|---|---|
| `proxy_set_header *`、`proxy_*_timeout`、`proxy_http_version` 等所有上游共用的 | `proxy_pass`（每个服务上游不同） |
| SSL 配置（`ssl_protocols`、`ssl_ciphers` 等） | `location` 块本身（不同服务前缀不同） |
| 安全头（`add_header X-Frame-Options ...` 等） | 任何只有一个 location 用的配置 |

**可以建多个 snippet 按用途分类**：

```
/etc/nginx/snippets/
├── proxy-common.conf      # 通用反代头与超时
├── proxy-sse.conf         # SSE/长轮询专用：proxy_buffering off; ...
├── ssl-strong.conf        # TLS 1.2+/强加密套件
└── security-headers.conf  # X-Frame-Options / CSP / HSTS 等
```

需要 SSE 的服务就 `include` 两个：

```nginx
location ^~ /api/chat/ {
    proxy_pass http://chat_backend/;
    include /etc/nginx/snippets/proxy-common.conf;
    include /etc/nginx/snippets/proxy-sse.conf;
}
```

**`include` 可以放在任意层级**（`http`、`server`、`location`），只要被引入的指令在那个层级合法。如果某段配置真的所有 location 都要，可以提到 `server` 层级 include，连 location 内部都不用写：

```nginx
server {
    include /etc/nginx/snippets/proxy-common.conf;   # 整个 server 内继承

    location ^~ /api/news/ { proxy_pass http://news_backend/; }
    location ^~ /api/gold/ { proxy_pass http://gold_backend/; }
}
```

**经验法则**：

- 配置稳定、所有 location 都需要 → 抬到 `server` 层级 include
- 配置可能因 location 不同而调整 → 在 location 内显式 include，**显式优于隐式**

**改完一定要 `nginx -t` 再 reload**：

```bash
sudo nginx -t              # 测试语法（会展开所有 include 一起检查）
sudo systemctl reload nginx
```

`include` 目标文件如果路径写错或语法有误，`nginx -t` 会**精确报告是哪个 snippet 文件的哪一行**，调试很友好。

#### 4.3 前端 axios：工厂模式 + 业务模块解耦

**`src/api/http.js`** —— 网关前缀**唯一**出现的地方：

```js
import axios from 'axios'

function createHttp(prefix) {
  const instance = axios.create({
    baseURL: prefix,         // '/api/news' 或 '/api/gold'
    timeout: 30_000,
    headers: { 'Content-Type': 'application/json' }
  })

  instance.interceptors.response.use(
    (resp) => resp.data,
    (err) => Promise.reject(err)   // 全局错误处理
  )

  return instance
}

export const newsHttp = createHttp('/api/news')
export const goldHttp = createHttp('/api/gold')
```

**业务接口模块** —— 只写干净的业务路径，与 proto 完全一致：

```js
// src/api/news.js
import { newsHttp } from './http'

export const getNewsByDate     = (data) => newsHttp.post('/v1/by_date', data)
export const listNewsCategories = ()    => newsHttp.get('/v1/categories')

// src/api/gold.js
import { goldHttp } from './http'

export const getGoldVolumeOI = (data) => goldHttp.post('/v1/volume_oi', data)
```

**页面/组件** —— 只 import 业务函数，不再碰 axios 实例：

```js
import { getNewsByDate } from '@/api/news'
const data = await getNewsByDate({ date: '2026-06-26' })
```

#### 4.4 vite dev 代理：与 nginx 行为同构

```js
// vite.config.js
server: {
  proxy: {
    '/api/news': {
      target: 'http://localhost:8083',
      changeOrigin: true,
      rewrite: (p) => p.replace(/^\/api\/news/, '')   // 与 nginx strip 一致
    },
    '/api/gold': {
      target: 'http://localhost:8082',
      changeOrigin: true,
      rewrite: (p) => p.replace(/^\/api\/gold/, '')
    }
  }
}
```

**关键**：dev 与 prod 的代理规则要**保持一致**，避免出现"本地能跑、线上 404"。

---

### 五、各层职责对照表

| 层 | 写什么 | 不该写什么 |
|---|---|---|
| proto | `/v1/by_date` | 不写 `/api/news/...`、`/news/...` |
| nginx | `location ^~ /api/news/ { proxy_pass http://upstream/; }` | 不写 `rewrite`（除非有特殊需求） |
| vite dev | `proxy: { '/api/news': { rewrite: strip前缀 } }` | 行为必须和 nginx 一致 |
| `http.js` | `createHttp('/api/news')` | 网关前缀**唯一**出现的地方 |
| 业务 api 文件 | `newsHttp.post('/v1/by_date')` | 不写 `/api/news/...`、不裸用 axios |
| 页面/组件 | `await getNewsByDate(...)` | 不直接拼 URL、不 import `newsHttp` |

---

### 六、设计哲学与收益

按这套分层走，proto、网关、前端三方各管自己的事，互不"知道"对方的实现细节：

- **proto 不感知前缀** → 后端服务可独立部署、独立换路径，不影响契约
- **前缀只在 `http.js` 一处出现** → 哪天网关前缀从 `/api/news` 改成 `/svc/news`，**只改一行**
- **业务代码读起来等于读 proto** → 排查问题 `grep '/v1/by_date'` 前后端都能命中
- **nginx 用 `proxy_pass /` 而非 `rewrite`** → 配置简洁，少一类常见 bug
- **dev / prod 代理同构** → 本地能跑 ≈ 线上能跑
- **多服务可无限扩展** → 新增 `report` 服务，只是再加一对 `location` + `createHttp('/api/report')`

---

### 七、记忆口诀

> proto 只写业务路径，前缀别往里塞。
> nginx 按前缀分流，转发前 strip 干净。
> 前端 baseURL 收口，业务代码读 proto。
> dev 与 prod 同构，本地线上一致。

---

### 八、常见错误与排查

| 现象 | 原因 | 解决 |
|---|---|---|
| 后端 404 | proto 路径含 `/api/news/...` 但 nginx 仍 strip 了前缀 | 要么去掉 proto 中的前缀，要么去掉 nginx 的 `rewrite`/尾斜杠 |
| 后端收到的路径里多一截 `/api/news` | nginx `proxy_pass` 没带尾斜杠且没写 rewrite | 给 `proxy_pass` 末尾加 `/`，或加 `rewrite ^/api/news/(.*)$ /$1 break;` |
| 后端 URL 多一个 `/` 或拼错 | `proxy_pass` 写法和 location 末尾斜杠不匹配 | 统一规范：`location ^~ /api/news/`（带尾斜杠）+ `proxy_pass http://upstream/;`（带尾斜杠） |
| 本地能跑线上 404 | vite proxy rewrite 与 nginx 行为不一致 | 二者保持同构 |
| 前端业务代码里到处出现 `/api/news/...` | 没用 axios 实例的 baseURL，裸拼 URL | 统一收口到 `http.js`，业务代码只写 `/v1/...` |

---

## 20. 反代到后端为什么不用 HTTP/2？

### 一、问题起源

第 14 点的 nginx 反代配置里有一行 `proxy_http_version 1.1;`。直觉上 HTTP/2 比 HTTP/1.1 更新更好，是不是把它改成 `proxy_http_version 2;` 性能会更高？

**简短答案：不能改，nginx 没有 `2` 这个值；而且即使能改，反代到后端用 HTTP/2 反而比 HTTP/1.1 更差。**

### 二、`proxy_http_version` 的合法取值

nginx 的 `ngx_http_proxy_module` 里这个指令**只支持两个值**：

| 值 | 含义 |
|---|---|
| `1.0` | 默认值，老旧 |
| `1.1` | 现代用法（支持 keepalive、chunked、Upgrade 等） |

写 `proxy_http_version 2;` 会被 `nginx -t` 直接拒绝：

```
nginx: [emerg] invalid value "2" in proxy_http_version
```

原因是 `ngx_http_proxy_module` **从设计之初就只是 HTTP/1.x 的反代客户端，从未实现 HTTP/2 客户端**。

### 三、nginx 反代到后端能不能用 HTTP/2？

截至 nginx 1.27.x（开源版）也**仍然不能**：

| 协议 | 浏览器 → nginx | nginx → 后端 |
|---|---|---|
| HTTP/2 | ✅ 完全支持（`listen 443 ssl http2;`） | ❌ `ngx_http_proxy_module` 不支持 |
| HTTP/1.1 | ✅ | ✅（`proxy_http_version 1.1;`） |
| gRPC（基于 HTTP/2） | 通过 `grpc_pass` 模块 | ✅ 但要走 `grpc_pass` 不是 `proxy_pass` |

**唯一让 nginx 用 HTTP/2 反代的情况：后端是 gRPC 服务，且用 `grpc_pass` 而非 `proxy_pass`。** 普通 HTTP/JSON 反代不行。

### 四、为什么 nginx 一直不做 HTTP/2 反代？

nginx 作者明确表态过：**反代到后端用 HTTP/2 没有实际收益，反而更糟**。

#### 4.1 HTTP/2 的优势在浏览器场景，反代段用不上

HTTP/2 相对 HTTP/1.1 的主要好处是：
- **多路复用（multiplexing）**：单 TCP 连接并发多请求
- **头部压缩（HPACK）**：减少重复 header 体积
- **Server Push**

这些是为了解决**浏览器 ↔ 服务器**的痛点：
- 浏览器同域名只能开 6 个 TCP 连接 → 多路复用解决
- 浏览器请求里 cookie / UA 重复巨大 → HPACK 压缩有效
- 网络是高延迟、高丢包的公网

而 **nginx ↔ 后端**这一段的环境完全不同：
- nginx 和后端通常**同机房甚至同一台机**，TCP 延迟极低（亚毫秒级）
- nginx 维护 **upstream 连接池（keepalive）**，本来就是长连接复用
- 后端服务都是程序自己生成的请求，header 不臃肿

所以 HTTP/2 多路复用在这一段**几乎为零收益**。

#### 4.2 HTTP/2 在反代场景反而是劣化

- **单连接队头阻塞（HOL blocking）**：HTTP/2 多路复用基于单条 TCP，TCP 一丢包重传，所有逻辑流都被阻塞。HTTP/1.1 用多条 TCP 反而**没有**这个问题。
- **CPU 开销**：HTTP/2 的二进制帧解析、HPACK 编解码比 HTTP/1.1 重，nginx 作为高性能反代追求极致 CPU 效率。
- **复杂性**：流控、优先级、Settings 帧带来的代码复杂度对运维和调试都不友好。

nginx 作者 Maxim Dounin 在邮件列表里有过原话（大意）：

> HTTP/2 在 backend 连接上的好处微乎其微，劣化却很明显。我们没有计划支持。

### 五、那怎么保住"前端用 HTTP/2"的好处？

**只让前面那一段走 HTTP/2 就够了**。完整链路：

```
浏览器 ──HTTP/2──► nginx ──HTTP/1.1（keepalive）──► 后端
       (公网，受益最大)         (内网/同机，HTTP/1.1 反而更优)
```

让前端走 HTTP/2 的最小配置：

```nginx
server {
    listen 443 ssl http2;        # ← 浏览器到 nginx 用 HTTP/2
    server_name domain.com;

    location ^~ /api/news/ {
        proxy_pass http://news_backend/;
        proxy_http_version 1.1;  # ← nginx 到后端用 HTTP/1.1（这是对的）
        # ...
    }
}
```

### 六、加分项：在 upstream 上启用 keepalive 真长连接

把 nginx → 后端的连接复用起来，能等价拿到 HTTP/2 多路复用绝大部分的收益：

```nginx
upstream news_backend {
    server 127.0.0.1:8083;
    keepalive 32;                # 维护 32 个空闲长连接备用
    keepalive_requests 1000;     # 单连接最多复用 1000 次再换新连接
    keepalive_timeout 60s;       # 空闲连接保活 60s
}

location ^~ /api/news/ {
    proxy_pass http://news_backend/;
    proxy_http_version 1.1;
    proxy_set_header Connection "";   # ← 必须清空，不然 keepalive 失效
    # ...
}
```

⚠️ **关键陷阱**：`proxy_set_header Connection "";` 这一行**必须有**。原因：

- HTTP/1.0 默认 `Connection: close`，HTTP/1.1 默认 `keep-alive`
- 客户端请求里如果带了 `Connection: close`，nginx 会原样透传给后端，后端就不会复用连接
- 显式置空，让 nginx 自己决定

加上之后，nginx → 后端就是真正的"长连接 + 多请求复用"，性能与 HTTP/2 在这一段几乎无差。

> 这两行（`Connection ""` 和 upstream 的 `keepalive`）也很适合抽进 [`/etc/nginx/snippets/proxy-common.conf`](#421-共用配置抽-snippet-详解) 里复用。

### 七、什么时候真的要 HTTP/2 反代？

**唯一的合理场景：后端是 gRPC 服务**。gRPC 协议本身强制基于 HTTP/2，这时候要用 `grpc_pass`：

```nginx
location /grpc.YourService/ {
    grpc_pass grpc://backend:50051;          # 注意是 grpc_pass，不是 proxy_pass
    grpc_set_header X-Real-IP $remote_addr;
}
```

但当前项目里，后端 `:8082`、`:8083` 是 **gRPC-Gateway 暴露的 HTTP/JSON** 接口（proto 里的 `option (google.api.http)` 走 RESTful 网关），所以**走 `proxy_pass` + `proxy_http_version 1.1` 就是最优解**。

### 八、结论速查表

| 问题 | 答案 |
|---|---|
| `proxy_http_version` 能改成 `2` 吗？ | ❌ 不能，nginx 不支持这个值 |
| 那能不能让 nginx 反代到后端走 HTTP/2？ | ❌ `ngx_http_proxy_module` 从未实现 |
| HTTP/2 不是更好吗？ | 在浏览器↔nginx 段是的；在 nginx↔后端段反而更差 |
| 现在的 `proxy_http_version 1.1;` 配置对吗？ | ✅ 完全正确，且是最佳实践 |
| 还能优化吗？ | 加 `upstream { keepalive 32; }` + `proxy_set_header Connection "";` 启用反代长连接 |

---

## 21. 仅 loopback 的 gRPC 服务与网关反代的意义及实现

### 问题

一个 Go 后端服务（如 nucur 资源评分系统）在同一个进程里同时启动了 gRPC 服务和 HTTP/JSON 网关，`main.go` 里有这么一句注释：

```go
// gRPC 服务（仅 loopback，由 gateway 反代）
grpcServer := grpc.NewServer(...)
```

其中「**仅 loopback，由 gateway 反代**」是什么意思？为什么要这么设计？具体是怎么实现的？

### 一、一句话结论

**同一个进程里跑了两个 server：gRPC 只监听本机回环地址（`127.0.0.1`，即 loopback），外网无法直连，纯作内部通道；真正对外接客的是 HTTP 网关，它把浏览器的 HTTP/JSON 请求「翻译」成 gRPC 调用转发给本机的 gRPC 服务，再把结果翻译回 JSON——这就是「反向代理（反代）」。**

### 二、逐词拆解那句注释

| 词 | 含义 | 对应配置 |
|---|---|---|
| **gRPC 服务** | 真正实现业务逻辑的服务器，注册了 Auth/Category/Resource/Comment 等服务 | `grpc.NewServer(...)` |
| **仅 loopback** | 只监听 `127.0.0.1:9090`，回环地址只有本机能访问，**外网连不上** | `GRPC_ADDR=127.0.0.1:9090` |
| **由 gateway 反代** | 对外的 HTTP 网关（监听 `:8080`）把请求反向代理到那个 loopback 的 gRPC | `HTTP_ADDR=:8080` |

关键：`127.0.0.1` 是回环地址，把真正的服务藏在内网，只暴露一个受控的对外入口，这是一种**安全设计**。

### 三、一次对外请求的完整流转

```
浏览器/前端
   │  HTTP/JSON:  POST /api/v1/auth/login {...}
   ▼
[HTTP 网关 :8080]                      ← 对外唯一入口（gateway/http.go）
   │  1) CORS / 静态文件 / 上传等特殊路由先行拦截
   │  2) 其余交给 grpc-gateway 的 ServeMux
   │  3) 按 .proto 的 HTTP 注解把 JSON 转成 gRPC 请求
   ▼  gRPC over 127.0.0.1（loopback，外网不可达）
[gRPC 服务 127.0.0.1:9090]
   │  JWT 拦截器解析令牌 → service 薄适配层
   ▼
service → usecase → repository → PostgreSQL
   ▲
   └── 结果沿原路返回，网关再把 gRPC 响应转回 JSON
```

### 四、实现方式（Go 代码拆解）

#### 4.1 启动仅监听回环的 gRPC 服务

```go
// gRPC 服务（仅 loopback，由 gateway 反代）
grpcServer := grpc.NewServer(grpc.ChainUnaryInterceptor(jwtMgr.UnaryInterceptor()))
nucurv1.RegisterAuthServiceServer(grpcServer, svc)
nucurv1.RegisterCategoryServiceServer(grpcServer, svc)
nucurv1.RegisterResourceServiceServer(grpcServer, svc)
nucurv1.RegisterCommentServiceServer(grpcServer, svc)

lis, _ := net.Listen("tcp", cfg.GRPCAddr)  // cfg.GRPCAddr = "127.0.0.1:9090"
go func() { grpcServer.Serve(lis) }()
```

`GRPCAddr` 配成 `127.0.0.1:9090` 而不是 `:9090`（后者会监听所有网卡），这一个字之差就是「仅 loopback」的关键——绑定回环网卡，外部网络的数据包根本到不了这个端口。

#### 4.2 网关作为 gRPC 客户端反代到本机

```go
// gateway/http.go
gwMux := runtime.NewServeMux(runtime.WithErrorHandler(runtime.DefaultHTTPErrorHandler))

// 因为目标是本机回环，明文通信无需 TLS
dialOpts := []grpc.DialOption{grpc.WithTransportCredentials(insecure.NewCredentials())}

// 把四组服务的 HTTP 路由都挂到 gwMux，拨号到 grpcAddr（127.0.0.1:9090）
nucurv1.RegisterAuthServiceHandlerFromEndpoint(ctx, gwMux, grpcAddr, dialOpts)
nucurv1.RegisterCategoryServiceHandlerFromEndpoint(ctx, gwMux, grpcAddr, dialOpts)
nucurv1.RegisterResourceServiceHandlerFromEndpoint(ctx, gwMux, grpcAddr, dialOpts)
nucurv1.RegisterCommentServiceHandlerFromEndpoint(ctx, gwMux, grpcAddr, dialOpts)
```

注意角色反转：`main.go` 里的 gRPC 是**服务端**，而网关这里是作为 gRPC **客户端**去连它的。`insecure` 凭证之所以安全，正是因为通信只发生在本机回环内。

#### 4.3 网关侧路由分流（不走 gRPC 的对外路由）

```go
mux := http.NewServeMux()
mux.Handle("/uploads/", fs)                       // 静态文件
mux.HandleFunc("/api/v1/upload/cover", uploadHandler) // 文件上传（multipart 不适合走网关自动转换）
mux.HandleFunc("/healthz", healthzHandler)        // 健康检查
mux.Handle("/", gwMux)                             // 兜底：其余全部反代到 gRPC

h.handler = h.withCORS(mux)                        // 最外层套 CORS 中间件
```

`http.ServeMux` 按最长前缀匹配分流：三条更具体的路径优先处理，其余 `/api/v1/...` 业务请求兜底给 grpc-gateway。

### 五、为什么这样设计（收益）

1. **安全**：真正的服务只监听回环，攻击面收敛为一个受控的 HTTP 网关；数据库、内部逻辑都不直接暴露公网。
2. **一份定义两套接口**：同一 `.proto` 同时产出 gRPC（供内部/未来服务间调用）与 REST/JSON（供浏览器前端），无需重复编写转换代码（详见第 22、23 节）。
3. **职责清晰**：网关专管对外 HTTP 关注点（CORS / 上传 / 静态 / 鉴权头），业务服务专注 gRPC 逻辑，互不干扰。

### 六、与「双端口对外」架构的区别

第 22、23 节讲的金价/新闻等服务，是 gRPC 与 HTTP **两个端口都对外**（gRPC 给后端微服务、HTTP 给浏览器）。而本节的 nucur 模式更进一步：**gRPC 端口仅绑定 loopback、完全不对外**，对外只留 HTTP 一个口子。两者取舍：

| 模式 | gRPC 端口 | 适用场景 |
|---|---|---|
| 双端口对外（第 22/23 节） | 监听 `:50052` 等，对外可达 | 确实有其他后端微服务要用 gRPC 直连 |
| 仅 loopback + 网关反代（本节） | 仅 `127.0.0.1`，外网不可达 | 只有浏览器前端消费，追求最小攻击面 |

### 七、记忆口诀

> **gRPC 藏在回环里，网关站在大门口。**
>
> 127.0.0.1 只认本机，外网想连连不上。
> 浏览器只跟 HTTP 打交道，网关翻译成 gRPC 再转发。
> 一份 proto 两套皮，安全与复用两不误。

---

## 22. 前端调用 gRPC 服务还是 HTTP 网关？两者有何区别？

### 问题

一个 Go 后端服务里同时启动了 gRPC 服务（监听 `GRPCPort`）和 HTTP 网关（grpc-gateway，监听 `HTTPPort`）。前端在调用的时候，是不是也可以走 gRPC？这两条调用路径到底有什么区别？该怎么选？

### 一、简短结论

**前端（浏览器）几乎无法直接调用原生 gRPC，绝大多数场景下只能走 HTTP 网关。** 这就是为什么 Go 后端服务在 `main.go` 里要同时启动  `rpc` 和  `http` 两套服务（如 news 项目下起了 `startGRPCServer` 和 `startHTTPGateway` 两套服务）。根本原因在于：前者服务于其他后端微服务，后者服务于浏览器/移动端等外部客户端。

### 二、为什么浏览器不能直接调 gRPC？

gRPC 基于 **HTTP/2 + Protobuf 二进制帧**，它依赖一些**浏览器无法直接操作的底层能力**：

| 能力 | gRPC 的要求 | 浏览器的限制 |
|------|------------|------------|
| HTTP/2 帧控制 | 需要直接读写 HTTP/2 frame（trailers、流控等） | 浏览器的 `fetch`/`XHR` API 是高度封装的，**拿不到 trailer**，也不让你操作底层帧 |
| Trailer 头 | gRPC 状态码 `grpc-status` 是放在 **HTTP trailer** 里返回的 | 浏览器无法读取 trailer |
| 二进制传输 | Protobuf 是二进制格式 | 能传二进制，但解码需要额外的 protobuf-js 库 |
| 双向流 | 支持客户端/服务端/双向流 | 浏览器不支持发送任意 HTTP/2 流 |

所以**前端写 `fetch('/v1/gold/price')` 走的永远是 HTTP/JSON**，这正好是 grpc-gateway 提供的能力。

### 三、典型项目中的两条调用路径（以金价市场分析服务 Gold 为例）

```mermaid
flowchart LR
    Browser["前端浏览器<br/>fetch / axios"] -->|HTTP/JSON| Gateway["HTTP 网关<br/>:HTTPPort"]
    Gateway -->|进程内直连| GoldSvc["GoldService<br/>业务实现单例"]

    GoClient["Go 后端服务<br/>其他微服务"] -->|gRPC/HTTP2/Protobuf| GRPCServer["gRPC 服务<br/>:GRPCPort"]
    GRPCServer --> GoldSvc

    GoldSvc --> UseCase["usecase + repository"]
```

关键点：`service.NewGoldService(repo)` 创建出来的是**同一个实例**，被两边共享，并通过两个不同的 Register 函数同时挂上去：

- `pb.RegisterGoldServiceServer(s, goldSvc)` —— 暴露给 gRPC 客户端
- `pb.RegisterGoldServiceHandlerServer(ctx, mux, goldSvc)` —— 暴露给 HTTP 客户端（**进程内直连**，不再绕一次 TCP，性能比传统 grpc-gateway 反向回连更好）

### 四、两种方式的核心区别

| 维度 | gRPC（直连） | HTTP 网关（grpc-gateway） |
|------|------------|----------------------|
| **协议** | HTTP/2 | HTTP/1.1 或 HTTP/2 |
| **数据格式** | Protobuf（二进制） | JSON（文本） |
| **传输体积** | 小（约 JSON 的 30%-50%） | 大 |
| **序列化性能** | 快（编译期生成代码） | 慢（反射 + 字符串解析） |
| **客户端生态** | Go/Java/C++/Python 等后端语言完善 | 全平台通吃，浏览器原生支持 |
| **可读性/调试** | 二进制，需要工具（grpcurl、BloomRPC） | curl、Postman、浏览器 DevTools 直接看 |
| **流式调用** | 支持四种模式（含双向流） | 仅 unary（单次请求-响应）友好 |
| **跨语言契约** | `.proto` 文件强约束 | 同 `.proto`，但走 JSON 自动转换 |
| **典型使用场景** | 后端微服务之间互相调用 | 浏览器、移动端、第三方对接 |

### 五、前端到底该怎么调？

#### ✅ 推荐：直接走 HTTP 网关

假设 `.proto` 里某个 RPC 映射到 `GET /v1/gold/price`，前端只要：

```javascript
// 前端代码（浏览器）
const res = await fetch('http://your-server:HTTPPort/v1/gold/price?type=AU9999');
const data = await res.json();
console.log(data); // 直接拿到 JSON 对象
```

如果在 mux 里配置了下面这两个选项：

```go
UseProtoNames:   true, // 字段名是 user_name 而非 userName
EmitUnpopulated: true, // 零值也输出
```

那前端拿到的字段命名风格**与 `.proto` 完全一致**，调试非常友好。

#### ❌ 不推荐：浏览器直连 gRPC

技术上可以用 **gRPC-Web**（需要在服务端额外加一层 `grpcweb` 代理，比如 envoy 或 `improbable-eng/grpc-web`），但：

- 仍然只支持 unary 和 server-streaming（双向流不行）
- 要引入 protobuf-js + 生成代码的工程复杂度
- 调试不直观，浏览器 DevTools 看不到结构化内容

对于**内部数据查询型小流量服务**，没必要上 gRPC-Web，**HTTP 网关就是最优解**。

### 六、什么时候用 gRPC 直连？

当一个**Go 服务（或其他后端服务）**要调另一个 Go 服务时，应该用 gRPC 客户端直连 `GRPCPort`：

```go
conn, _ := grpc.Dial("gold-service:GRPCPort", grpc.WithInsecure())
client := pb.NewGoldServiceClient(conn)
resp, _ := client.GetPrice(ctx, &pb.GetPriceRequest{...})
```

这样可以享受 Protobuf 的**性能红利**和**编译期类型安全**——参数错了直接编译不通过，比 HTTP/JSON 的运行时报错友好得多。

### 七、记忆口诀

> **前端用 HTTP 网关，后端服务间用 gRPC。**
>
> 浏览器拿不到 HTTP/2 trailer，所以读不了 gRPC 状态码 → 只能走 JSON。
> 后端服务之间是同语言、同机房、可控环境 → 走 Protobuf 又快又安全。

### 八、常见误区

| 误区 | 实际情况 |
|---|---|
| "既然有 gRPC 服务，前端也走 gRPC 性能更好" | 浏览器根本调不通原生 gRPC，硬要上要引入 gRPC-Web，得不偿失 |
| "HTTP 网关性能差，应该砍掉" | 网关是进程内直连同一个 service 实例，开销只是多一层 JSON 编解码，可接受 |
| "两个端口暴露的接口不一样" | 同一个 `service.NewGoldService(repo)` 实例同时挂在两边，**业务逻辑完全一致**，只是协议外壳不同 |
| "前端非要用 protobuf 才能保证字段一致" | 配 `UseProtoNames: true` 后，JSON 字段名与 `.proto` 完全一致，前后端共用同一份字段约定即可 |

---

## 23. grpc-gateway 路由注册机制与双端口架构

### 问题

一个 Go 后端服务的 `main.go` 大致长这样：

```go
go startGRPCServer(cfg, goldSvc)    // 监听 :50050
go startHTTPGateway(cfg, goldSvc)   // 监听 :8080
```

两个 goroutine 启动完就阻塞等信号了。**通篇看不到任何 `mux.HandleFunc("/v1/kline", ...)` 之类的路由注册代码**，但浏览器请求 `POST /v1/kline` 却能正常命中后端。

两个核心疑问：

1. 这些 HTTP 路由到底是在哪一步被注册到 `mux` 上的？
2. 既然有了 gRPC 服务，为什么还要再开一个 HTTP 端口？两个端口职责到底怎么分？

### 一、简短结论

- **路由不是手写的**，是 `protoc-gen-grpc-gateway` 这个 protoc 插件**在编译期**根据 `.proto` 文件里的 `google.api.http` 注解**自动生成**到一份叫 `xxx.pb.gw.go` 的代码里，用一行 `pb.RegisterXxxServiceHandlerXxx(...)` 就能把所有路由批量塞进 `mux`。
- **双端口（gRPC + HTTP）**是为了**用一份业务实现同时服务两类客户端**：gRPC 端口（`:50052`）服务后端微服务之间高效通信，HTTP 端口（`:8082`）服务浏览器/移动端等只能走 HTTP/JSON 的客户端。两端共享同一个 `*service.GoldService` 单例，**业务逻辑只写一份**。

### 二、路由注册的完整链路（编译期 → 运行期）

```mermaid
flowchart TB
    subgraph Compile["编译期（protoc）"]
        Proto[".proto 文件<br/>option (google.api.http) {<br/>  post: '/v1/kline'<br/>  body: '*'<br/>}"] -->|protoc-gen-grpc-gateway 插件| GW["自动生成 xxx.pb.gw.go<br/>内部含 mux.Handle(POST, '/v1/kline', ...)"]
    end

    subgraph Runtime["运行期（main.go）"]
        Main["runtime.NewServeMux()<br/>创建空 mux"] --> Reg["pb.RegisterXxxHandlerServer(ctx, mux, svc)<br/>一行调用 = 7 条 mux.Handle 全部注入"]
        Reg --> Listen["http.ListenAndServe(:8082, mux)"]
        Listen --> HTTP["浏览器 POST /v1/kline → mux 路由命中 → 函数调用 svc.GetKlineData"]
    end

    GW -.->|go module 拉到本地缓存<br/>~/go/pkg/mod/.../xxx.pb.gw.go| Reg

    style Compile fill:#fff3e0,stroke:#f57c00
    style Runtime fill:#e8f5e9,stroke:#2e7d32
```

#### 第 0 层：源头是 `.proto` 文件里的 HTTP 注解

```proto
rpc GetKlineData (GetKlineRequest) returns (GetKlineResponse) {
  option (google.api.http) = {
    post: "/v1/kline"
    body: "*"
  };
}
```

这是 Google 标准的 HTTP 转译注解，告诉插件：「这个 RPC 要暴露成 HTTP `POST /v1/kline`」。

#### 第 1 层：编译期插件生成 `xxx.pb.gw.go`

执行 `protoc --grpc-gateway_out=. xxx.proto` 时，`protoc-gen-grpc-gateway` 插件读取 `option (google.api.http)`，生成大致这样的代码：

```go
mux.Handle(http.MethodPost, pattern_GoldService_GetKlineData_0, func(w, req, pathParams) {
    resp, _, err := request_GoldService_GetKlineData_0(...)
    ...
})

var pattern_GoldService_GetKlineData_0 = runtime.MustPattern(
    runtime.NewPattern(1, []int{2, 0, 2, 1}, []string{"v1", "kline"}, ""))
```

**「找不到 `mux.HandleFunc` 是因为它根本不在你的工程里**——它在 proto 仓库的产物里，被 go module 拉到本机缓存路径 `~/go/pkg/mod/.../xxx.pb.gw.go`。

#### 第 2 层：包装函数 `RegisterXxxHandlerServer / FromEndpoint`

生成的网关文件里还有 4 个对外的注册入口，最常用的两个：

| 注册函数 | 工作机制 | 何时使用 |
|---|---|---|
| `RegisterXxxHandlerServer(ctx, mux, svc)` | 进程内直接调用 svc 的 Go 方法 | HTTP 网关与 gRPC 服务在**同一进程**（推荐） |
| `RegisterXxxHandlerFromEndpoint(ctx, mux, "localhost:50052", opts)` | 拨号 gRPC 端口、当 gRPC 客户端转发 | HTTP 网关与 gRPC 服务**可分离部署**或需要 streaming |

它们内部都会把 7 条 `mux.Handle(...)` 一次性灌进 `mux`。

#### 第 3 层：`main.go` 一行调用触发注册

```go
mux := runtime.NewServeMux(...)                                    // 创建空 mux
pb.RegisterGoldServiceHandlerServer(ctx, mux, goldSvc)             // ⭐ 一行 = 7 条 mux.Handle
http.ListenAndServe(cfg.Server.HTTPPort, mux)                      // 启动 HTTP，全部路由就绪
```

**真相**：你「看不到」`mux.HandleFunc` 的根本原因——它被插件生成、藏在外部 module 里、被一行 `Register...` 函数批量执行掉。这正是 grpc-gateway 的精髓：**单一契约（`.proto`）驱动 gRPC server + gRPC client + HTTP 路由 + Swagger 四份代码同时产出**。

### 三、双端口架构：gRPC + HTTP 各司其职

```mermaid
flowchart LR
    Browser["浏览器 / curl<br/>HTTP/JSON"] --> HTTPPort["HTTP 网关<br/>:8082"]
    HTTPPort -->|进程内函数调用| Svc["*service.GoldService<br/>共享单例"]

    OtherSvc["其他后端服务<br/>(如 mcp)"] -->|gRPC/HTTP2/Protobuf| GRPCPort["gRPC 服务<br/>:50052"]
    GRPCPort --> Svc

    Svc --> Logic["usecase + repository"]
    Logic --> DB[("数据库")]

    style Svc fill:#e8f5e9,stroke:#2e7d32,stroke-width:3px
```

#### 3.1 为什么要开两个端口？

两类客户端**协议偏好截然不同**：

| 客户端类型 | 偏好协议 | 原因 |
|---|---|---|
| 浏览器 / 移动端 / 第三方 | HTTP/JSON | 浏览器 `fetch`/`XHR` 拿不到 HTTP/2 trailer，**根本读不出 gRPC 状态码**；JSON 调试友好 |
| 后端 Go / Java / Python 服务 | gRPC（HTTP/2 + Protobuf） | 二进制体积小、编译期类型安全、性能强 |

**单一服务无法两边都最优**——所以同时开两个端口，让客户端各取所需。详见第 17 节。

#### 3.2 共享单例：业务逻辑只写一份

```go
// main.go
goldSvc := service.NewGoldService(repo)   // ← 整个进程只构造一次

go startGRPCServer(cfg, goldSvc)          // gRPC 端注册
go startHTTPGateway(cfg, goldSvc)         // HTTP 端注册（进程内直连）
```

两个端口注册的是**同一个对象指针**：

- `pb.RegisterGoldServiceServer(grpcSrv, goldSvc)` —— 给 gRPC 客户端用
- `pb.RegisterGoldServiceHandlerServer(ctx, mux, goldSvc)` —— 给 HTTP 客户端用

**好处**：参数校验、业务逻辑、依赖注入只写一份；不会出现「HTTP 接口和 gRPC 接口行为不一致」的低级 bug。

#### 3.3 HTTP 网关的两种实现模式（关键设计抉择）

```mermaid
flowchart TB
    subgraph A["模式 A：HandlerServer（进程内直连，推荐）"]
        direction LR
        H1["HTTP 请求"] --> M1["mux"] -->|JSON→pb.Request| F1["goldSvc.GetKlineData(ctx, req)<br/>普通 Go 函数调用"]
    end

    subgraph B["模式 B：FromEndpoint（回拨 TCP）"]
        direction LR
        H2["HTTP 请求"] --> M2["mux"] -->|JSON→pb.Request| GC["内置 gRPC Client"]
        GC -.->|localhost TCP+HTTP/2| GS["gRPC Server :50052"]
        GS --> F2["goldSvc.GetKlineData"]
    end

    style A fill:#e8f5e9,stroke:#2e7d32,stroke-width:3px
    style B fill:#fff3e0,stroke:#f57c00
```

两种模式对比：

| 维度 | HandlerServer（A） | FromEndpoint（B） |
|---|---|---|
| HTTP 路径上是否经过 TCP | ❌ 不经过 | ✅ 回拨本机 gRPC 端口 |
| HTTP 路径上 Protobuf 编解码次数 | 0 次 | 2 次（client 编 + server 解） |
| 单请求延迟 | 仅 JSON 编解码 + 函数调用 | 含一次本机 TCP 往返 + 两次 protobuf 编解码 |
| 是否支持 streaming RPC | ⚠️ 仅 unary | ✅ unary / server-stream / client-stream / bidi |
| 是否可独立部署 HTTP 网关与 gRPC | ❌ 必须同进程 | ✅ 可分离 |
| 是否经过 gRPC 拦截器（认证/限流/tracing） | ❌ 不经过 | ✅ 经过 |

**选型口诀**：

> 全是 unary RPC、内部小流量 → 用 **HandlerServer**（更快、更简洁）
> 含 streaming RPC、需要独立扩缩容、需要 gRPC 拦截器统一管控 → 用 **FromEndpoint**

### 四、谁该走哪个端口？

| 调用方 | 应该走 | 原因 |
|---|---|---|
| 浏览器（Vue/React） | HTTP `:8082` | 浏览器读不出 HTTP/2 trailer，原生 gRPC 不可用 |
| 移动端（iOS/Android） | HTTP `:8082`（或 gRPC-Web） | gRPC 客户端库不普及；HTTP/JSON 通吃 |
| Go/Java/Python 后端微服务 | gRPC `:50052` | Protobuf 体积小、编译期类型安全、性能强 |
| Postman / curl 调试 | HTTP `:8082` | 看得见结构化 JSON，所见即所得 |
| `grpcurl` / BloomRPC | gRPC `:50052` | 调试 gRPC 专用工具 |

### 五、常见误区

| 误区 | 实际情况 |
|---|---|
| 「找不到 `mux.HandleFunc`，路由是不是没注册？」 | 注册代码在 `xxx.pb.gw.go` 里，由 protoc 插件生成，被 `RegisterXxxHandlerXxx` 一行批量执行 |
| 「HTTP 网关性能差，应该砍掉只留 gRPC」 | 浏览器调不通原生 gRPC；HandlerServer 模式下 HTTP 只比 gRPC 多一次 JSON 编解码，开销极小 |
| 「两个端口应该跑两份不同的业务代码」 | 应该共享同一个 `*service.GoldService` 单例，避免双份维护漂移 |
| 「HTTP 网关只能走 FromEndpoint 回拨 gRPC 端口」 | grpc-gateway 同时支持 `HandlerServer`（进程内直连）和 `FromEndpoint`（回拨）两种模式，前者更快 |
| 「`google.api.http` 注解只是给文档看的」 | 它是 protoc 插件生成路由表的**唯一依据**，少写一个注解，对应 RPC 就没有 HTTP 入口 |

### 六、记忆口诀

> **HTTP 路由不是写出来的，是 proto 注解长出来的。**
>
> 一份 `.proto` → 编译期插件生成网关代码 → `main.go` 一行 `Register` 把所有 `mux.Handle` 全部塞进 mux。
>
> **gRPC 端口给后端服务用，HTTP 端口给浏览器用，业务实现只写一份。**
>
> 两个端口对应两层壳，里面共享同一个 service 单例。

---

## 24. 契约与部署拓扑解耦与资源导向的 API 路径设计

### 问题

在用 gRPC + grpc-gateway 定义接口时，`.proto` 里的 `option (google.api.http)` 路径到底该怎么写？要不要把产品名（如 `/api/agently`）写进去？动词式的 `/auth/register`、`/switch`、`/chat` 是不是不够规范？企业级做法是怎样的？

### 一、两个核心原则

#### 原则 1：契约与部署拓扑解耦（路径里不写部署前缀）

**`.proto` 契约只描述"业务是什么"，不描述"它被部署在哪个前缀下"。**

- 契约里只写**版本 + 资源**的业务路径，如 `/v1/sessions`、`/v1/sessions/{id}/messages`。
- **不要**把产品名 / 网关分流前缀（`/api`、`/agently`、`/svc/xxx`）写进注解——这些是**部署拓扑**，应由 nginx / ingress / API 网关在外层统一添加或剥离（strip）。
- 前端 axios 的 `baseURL` 收口前缀，业务代码只写 `/v1/...`（详见第 19 点的分层实践）。

**为什么要解耦：**

| 若把 `/api/agently` 写进 proto | 解耦后（proto 只写 `/v1/...`） |
|---|---|
| 运维想把前缀改成 `/svc/agently` 得改 proto 重新发版 | 改 nginx 一行即可，契约不动 |
| 契约与某一次部署强绑定，无法复用 | 同一契约可挂在任意前缀 / 任意域名下 |
| 多环境（内网/公网/多租户）前缀不同就冲突 | 一份契约通吃所有环境 |

> 一句话：**proto 关心"业务身份"，网关关心"挂在哪"，两者互不侵入。**

#### 原则 2：资源导向（Google AIP 风格）

RPC 风格的 `/auth/register`、`/switch` 是"动词满天飞"；Google API 设计指南（AIP）推崇 **资源（名词）+ 标准方法** 的 REST 化风格：

- **名词用复数集合**：`/v1/sessions`、`/v1/agents`、`/v1/tools`。
- **层级表达从属关系**：`/v1/sessions/{session_id}/messages`。
- **标准 CRUD 用 HTTP 动词**：`GET`（查）、`POST`（建）、`PATCH/PUT`（改）、`DELETE`（删）。
- **非 CRUD 的"动作型"接口用自定义方法**：路径里用冒号 `:动词`，如 `:switch`、`:chat`、`:archive`、`:login`。
- **版本前缀**：路径带 `/v1`，与 proto 的 `package xxx.v1` 对齐，便于将来 `v2` 并存。

### 二、实战改造对照表（以 Agently 为例）

把 RPC 风格重构为"资源导向 + `/v1` + 去产品名前缀"：

| 旧（RPC 风格，含产品名） | 新（资源导向，契约干净） |
|---|---|
| `POST /api/agently/auth/send-code` | `POST /v1/auth:sendVerifyCode` |
| `POST /api/agently/auth/register` | `POST /v1/auth:register` |
| `POST /api/agently/auth/login` | `POST /v1/auth:login` |
| `GET /api/agently/agents` | `GET /v1/agents` |
| `GET /api/agently/tools` | `GET /v1/tools` |
| `POST /api/agently/switch` | `POST /v1/agents:switch` |
| `GET/POST /api/agently/sessions` | `GET/POST /v1/sessions` |
| `GET .../sessions/{id}/messages` | `GET /v1/sessions/{session_id}/messages` |
| `POST .../sessions/{id}/chat` | `POST /v1/sessions/{session_id}:chat` |
| `DELETE .../sessions/{id}`（归档） | `POST /v1/sessions/{session_id}:archive` |

> 归档是"软删除/状态变更"而非物理删除，用自定义方法 `:archive`（POST）比 `DELETE` 语义更贴切。

产品名前缀 `/api/agently` 交给 nginx 去前缀反代：外部 `/api/agently/v1/...` → 网关内部 `/v1/...`。

### 三、AIP 命名规则速记

| 场景 | 写法 | 示例 |
|---|---|---|
| 查询集合 | `GET /v1/{集合复数}` | `GET /v1/sessions` |
| 创建资源 | `POST /v1/{集合复数}` | `POST /v1/sessions` |
| 查询单个 | `GET /v1/{集合}/{id}` | `GET /v1/sessions/{id}` |
| 子资源 | `GET /v1/{集合}/{id}/{子集合}` | `GET /v1/sessions/{id}/messages` |
| 删除 | `DELETE /v1/{集合}/{id}` | `DELETE /v1/sessions/{id}` |
| 自定义动作 | `POST /v1/{集合}/{id}:动词` 或 `POST /v1/{集合}:动词` | `:chat`、`:switch`、`:archive` |

### 四、自定义方法 `:动词` 在 proto / grpc-gateway 中的写法

```proto
// 带资源 id 的动作
rpc Chat(ChatRequest) returns (stream ChatEvent) {
  option (google.api.http) = {
    post: "/v1/sessions/{session_id}:chat"
    body: "*"
  };
}

// 集合级动作（无 id）
rpc SwitchAgent(SwitchAgentRequest) returns (SwitchAgentResponse) {
  option (google.api.http) = {
    post: "/v1/agents:switch"
    body: "*"
  };
}
```

- protoc-gen-grpc-gateway 会正确把 `:chat` 解析为"自定义动词"（最后一段 `{session_id}` 之后的 `:动词`）。
- 若手动 `mux.HandlePath` 注册（如自定义 SSE），pattern 同样写 `"/v1/sessions/{session_id}:chat"`。

### 五、与前端 / nginx 的配合

- **proto 路径不含产品名** → 前端只需把产品前缀收口在 axios `baseURL`（如 `/api/agently`），业务代码只写 `/v1/...`。
- **nginx 去前缀反代**：`location ^~ /api/agently/ { proxy_pass http://127.0.0.1:8080/; }`（末尾带 `/` 自动 strip 前缀，详见第 18、19 点）。
- **package 名不影响 REST 路径**：`package xxx.v1` 只决定原生 gRPC 方法路径 `/xxx.v1.Service/Method`（通常仅回环可达，见第 21 点），浏览器走的 REST 路径完全由 `google.api.http` 注解决定，两者互不影响。

### 六、收益

- **契约稳定**：换前缀、换域名、多环境部署都不用改 proto。
- **语义清晰**：URL 读起来就是"对什么资源做什么"，符合业界共识，第三方对接零学习成本。
- **版本可演进**：`/v1` 与 `package .v1` 对齐，`v2` 可平滑并存。
- **前后端一致**：`grep '/v1/sessions'` 前后端都能命中，排查高效。

### 七、记忆口诀

> **契约只写业务路径，版本打头（/v1），前缀别塞。**
> **名词复数做资源，从属靠层级；CRUD 用 HTTP 动词，动作用 `:动词`。**
> **产品名交给网关加，package 名只管 gRPC 内部路由。**

---

## 25. Go 编译期接口实现断言（`var _ Iface = (*T)(nil)`）

### 问题

在 Go 代码里经常看到这样一行：

```go
// 编译期断言：*StoreService 实现 AnClawStore（接口漂移时启动期失败）
var _ AnClawStore = (*StoreService)(nil)
```

它到底是什么意思？为什么要这么写？

### 一、逐字拆解语法

| 片段 | 含义 |
|---|---|
| `var _` | 声明一个变量，名字是 `_`（空标识符），表示**不打算使用**这个变量，只想让编译器帮忙做检查 |
| `AnClawStore` | 变量的类型，是一个**接口** |
| `(*StoreService)(nil)` | 把 `nil` 类型转换成 `*StoreService`，得到一个**类型是 `*StoreService`、值是 nil 的指针** |
| `=` | 把右边 `*StoreService` 类型的 nil 值，**赋值**给左边 `AnClawStore` 接口类型的变量 |

**关键点**：Go 是静态类型语言，把 `*StoreService` 赋值给 `AnClawStore` 接口时，编译器**必须验证** `*StoreService` 实现了 `AnClawStore` 接口的**所有方法**。少实现任何一个方法，**编译直接报错**。

### 二、为什么写 `(*StoreService)(nil)` 而不是 `&StoreService{}`

两种写法都能触发编译期检查，但 `(*StoreService)(nil)` 有两个优势：

1. **零运行时开销**
   - `&StoreService{}` → 真的会在堆/栈上分配一个 `StoreService` 结构体。
   - `(*StoreService)(nil)` → 只是一个类型化的 nil 指针，**不分配任何内存**。
   - 因为用的是 `_`（丢弃变量），编译器会优化掉，但语义上更"干净"—— 明确表达"只关心类型检查，不关心值"。

2. **不依赖结构体字段**
   - 若 `StoreService` 有必填字段（如 `db *sql.DB`、`logger *zap.Logger`），`&StoreService{}` 会创建一个字段全 nil 的"半成品"实例。
   - `(*StoreService)(nil)` 就是 nil，反正也用不到，语义更明确。

### 三、这行代码到底在防什么 —— "接口漂移"

**"接口漂移"**：接口和实现悄悄跑偏了，直到运行时才爆炸。

#### 场景：接口演进的真实案例

一开始接口是这样：

```go
type AnClawStore interface {
    GetTicketStatus(ctx context.Context, ticketID int64) (*Status, error)
    UpdateLastEventID(ctx context.Context, ticketID int64, eventID int64) error
}

type StoreService struct { /* ... */ }

func (s *StoreService) GetTicketStatus(...) (*Status, error) { ... }
func (s *StoreService) UpdateLastEventID(...) error { ... }

// 编译期断言
var _ AnClawStore = (*StoreService)(nil)  // ✅ 编译通过
```

半年后，同事 A 给接口加了个方法：

```go
type AnClawStore interface {
    GetTicketStatus(...) (*Status, error)
    UpdateLastEventID(...) error
    IncFailureCount(ctx context.Context, ticketID int64) error  // ← 新增
}
```

但同事 A **忘了**给 `StoreService` 补上 `IncFailureCount` 方法。

##### 若**没有**这行断言：

- `StoreService` 本身**编译得过**（Go 是结构化类型系统，实现方不主动声明要实现哪个接口）。
- 只有当**别的代码**真的写了 `var x AnClawStore = &StoreService{...}` 的地方，才会报错。
- 更糟糕的是：若 `StoreService` 只在**依赖注入框架**里通过 `interface{}` 传递（wire、fx、反射注入），那么：
  - **编译过了** ✅
  - **单测可能也没覆盖到那条注入路径** ✅
  - **生产环境启动/运行时**才 panic ❌ 💥

##### 若**有**这行断言：

同事 A 一 `go build`，**当场编译失败**：

```
./store.go:42:5: cannot use (*StoreService)(nil) (type *StoreService) as type AnClawStore in assignment:
        *StoreService does not implement AnClawStore (missing IncFailureCount method)
```

**秒级发现问题**，进不了代码库，更别说到生产。

### 四、为什么注释说"启动期失败"

严格讲，这行断言是**编译期**就失败，比"启动期"还早。作者写"启动期失败"是想强调对比：

| 检查时机 | 发现问题的成本 |
|---|---|
| **编译期**（这行断言） | 秒级，开发本地 `go build` 就爆 |
| **启动期**（依赖注入初始化） | 分钟级，服务起不来，但至少没影响线上 |
| **运行期**（真正调那个方法） | 小时级/天级，可能发生在半夜、大促、客户投诉时 💀 |

作者的意思是：**"宁愿让它在启动/编译时挂，也不要让它偷偷带病上线"** —— 防御性编程的经典体现。

### 五、和 `ExtFlowWriter` 接口的呼应

在生产代码里通常会成对出现"接口 + 编译期断言"：

```go
// 接口定义
type ExtFlowWriter interface {
    WriteReply(ctx context.Context, ticketID int64, content string) (int64, error)
}

// 生产实现
type TicketFlowWriter struct{ client *ticket.Client }
func (w *TicketFlowWriter) WriteReply(...) (int64, error) { ... }

// 🔒 编译期断言：一旦接口新增方法，这里立刻炸
var _ ExtFlowWriter = (*TicketFlowWriter)(nil)
```

**接口 + 编译期断言**是 Go 里事实上的"契约验证"套路：

- 接口定义了**契约**；
- 断言保证**实现始终符合契约**；
- 一旦契约变了（接口漂移），实现必须**同步跟进**，否则连编译都过不去。

### 六、一句话记住

> **`var _ Iface = (*T)(nil)` 是 Go 里"我承诺 `*T` 实现了 `Iface`"的书面契约，让编译器帮你 24 小时监督这个承诺不被打破。**

这是几乎所有 Go 生产级项目都会用的模式，尤其在**接口和实现分文件/分包**、或**大量依赖注入**的场景下，属于**低成本、高收益**的防御性技巧。

---

## 26. 查看云服务器运行状态的常用命令速查

### 问题
查看云服务器运行状态有哪些命令？比如查看内存使用情况、网络带宽等数据。

### 解答

Linux 云服务器的运行状态排查可以按 **CPU / 内存 / 磁盘 / 网络 / 进程 / 系统整体** 六大维度来看，每类都有「一句话看全景」和「深入排查」两种命令。下面标注 ⭐ 的是日常最常用的。

---

### 一、系统整体概览

| 命令 | 作用 | 备注 |
|---|---|---|
| ⭐ `top` | 实时进程 + CPU + 内存动态视图 | 按 `q` 退出，按 `M` 按内存排序，`P` 按 CPU 排序 |
| ⭐ `htop` | `top` 的彩色增强版，可鼠标操作 | 需要 `yum/apt install htop`，强烈推荐 |
| ⭐ `uptime` | 一行看负载：运行时长 + 1/5/15 分钟 load average | load 超过 CPU 核数就要警惕 |
| `w` | 当前登录用户 + load + 每个用户在干啥 | 类似 uptime，多了用户信息 |
| `dstat -tcmndl 2` | CPU/内存/网络/磁盘/负载 **一屏看全** | 需装 `dstat`，运维排查神器 |
| `glances` | Python 写的全能监控面板 | 一屏看 CPU/内存/磁盘/网络/进程 |

**Load average 怎么看？**

```
load average: 0.85, 1.20, 1.55
```

三个数分别是 **过去 1 / 5 / 15 分钟的平均运行/等待进程数**。经验法则：数值 ≤ CPU 核数视为健康；持续高于核数 × 1.5 说明系统在排队。

---

### 二、CPU 使用情况

| 命令 | 作用 |
|---|---|
| ⭐ `top` 后按 `1` | 展开显示每个 CPU 核的使用率 |
| ⭐ `mpstat -P ALL 2` | 每 2 秒刷新每个 CPU 核的详细占用（`%usr`/`%sys`/`%iowait`/`%idle`） |
| `vmstat 2` | 2 秒刷新一次：进程队列、内存、swap、IO、CPU 综合 |
| `pidstat -u 2` | 按进程维度看 CPU 占用 |
| `nproc` | 一行显示 CPU 逻辑核数 |
| `lscpu` | CPU 型号、核数、主频、缓存、虚拟化信息 |

**关键指标解读**（`top` 里的 CPU 那行）：

- `us`：用户态占用高 → 应用自身在算
- `sy`：内核态占用高 → 系统调用/上下文切换多
- `wa`：**iowait 高 → 磁盘 IO 瓶颈**，不是 CPU 忙
- `st`：**steal time 高 → 云主机被邻居抢 CPU**（超卖了！云服务器特有指标）
- `id`：空闲

---

### 三、内存使用情况

| 命令 | 作用 |
|---|---|
| ⭐ `free -h` | 一行看总量/已用/可用/swap，`-h` 是人类可读单位 |
| ⭐ `free -h -s 2` | 每 2 秒刷新一次 |
| `vmstat 2` | 观察 swap 换入换出（`si`/`so`），非 0 就说明内存紧张 |
| `cat /proc/meminfo` | 内存细节全景（含 Buffers/Cached/Slab/HugePages） |
| `ps aux --sort=-%mem \| head -20` | 内存占用 Top 20 进程 |
| `pmap -x <PID>` | 单个进程的内存分布明细 |
| `smem -tk` | 更准确的 PSS/USS 内存统计（需安装） |

**`free -h` 输出别看错列** ⚠️：

```
              total   used   free   shared  buff/cache   available
Mem:           16G    8.2G   1.1G     0.5G       6.7G        7.2G
```

- **`available` 才是应用可申请的真实可用内存**（≠ free）
- `buff/cache` 是文件系统缓存，紧张时会自动释放，**不算真的被占用**
- 判断内存是否紧张：看 `available` 占比，以及 `swap used` 是否在增长

---

### 四、磁盘使用情况

#### 4.1 空间维度

| 命令 | 作用 |
|---|---|
| ⭐ `df -h` | 各文件系统的容量/已用/可用/挂载点 |
| ⭐ `df -i` | 查看 **inode 使用率**（inode 满了也会写不进去，容易忽视！） |
| ⭐ `du -sh *` | 当前目录下每项占用大小 |
| `du -h --max-depth=1 / \| sort -h` | 逐层找出大目录（找磁盘吃谁了） |
| `ncdu /` | 交互式磁盘占用分析（强烈推荐，需 `apt install ncdu`） |
| `lsblk` | 块设备树（哪些盘、分区、挂载点） |

#### 4.2 IO 性能维度

| 命令 | 作用 |
|---|---|
| ⭐ `iostat -x 2` | 每 2 秒刷新磁盘 IO：吞吐、IOPS、await、%util |
| ⭐ `iotop` | 按进程看谁在读写磁盘（需 root） |
| `pidstat -d 2` | 按进程维度的 IO 统计 |

**iostat 关键列**：

- `%util`：**接近 100% → 磁盘打满**
- `await`：单次 IO 平均耗时（ms），机械盘 > 20ms、SSD > 5ms 都算慢
- `r/s`、`w/s`：每秒读写次数（IOPS）

---

### 五、网络使用情况

#### 5.1 连通性 & 配置

| 命令 | 作用 |
|---|---|
| ⭐ `ip a` / `ifconfig` | 网卡列表和 IP 地址 |
| ⭐ `ip r` / `route -n` | 路由表 |
| `ping <host>` | 连通性 + 延迟 |
| `traceroute <host>` / `mtr <host>` | 逐跳看链路，`mtr` 是持续统计版，排查网络故障首选 |
| `curl -w "@format" -o /dev/null -s <url>` | 精细化测量 DNS/TCP/TLS/首字节各阶段耗时 |

#### 5.2 带宽 & 流量 ⭐

| 命令 | 作用 |
|---|---|
| ⭐ `iftop -i eth0` | 实时看**每个连接**的带宽占用（需 root，最常用） |
| ⭐ `nload eth0` | 简洁的入/出带宽实时图 |
| ⭐ `sar -n DEV 2` | 按网卡统计收发速率（`rxkB/s`、`txkB/s`） |
| `vnstat -l` | 实时流量；`vnstat -d` 看每日累计（做流量账单必备） |
| `bmon` | 多网卡带宽仪表盘 |
| `iptraf-ng` | 按协议/端口/连接维度的老牌工具 |

#### 5.3 连接 & 端口

| 命令 | 作用 |
|---|---|
| ⭐ `ss -tnlp` | 看**监听中**的 TCP 端口和对应进程（比 netstat 快很多） |
| ⭐ `ss -tan` | 看所有 TCP 连接状态 |
| ⭐ `ss -s` | 连接总数摘要（各种状态数量） |
| `ss -tan \| awk '{print $1}' \| sort \| uniq -c` | 快速统计各状态连接数（`ESTAB`/`TIME-WAIT`/`CLOSE-WAIT` 等） |
| `netstat -tnlp` | 传统写法，新系统建议用 `ss` |
| `lsof -i:8080` | 谁在监听 8080 端口 |

**排查连接异常时重点看**：

- `TIME_WAIT` 过多 → 短连接太多，考虑复用
- `CLOSE_WAIT` 过多 → **应用层没正确关闭连接**（代码 bug 信号）
- `SYN_RECV` 过多 → 可能被 SYN Flood 攻击

#### 5.4 抓包

| 命令 | 作用 |
|---|---|
| `tcpdump -i eth0 -nn port 80` | 抓 80 端口流量 |
| `tcpdump -i eth0 -w cap.pcap` | 抓包存文件，回本地用 Wireshark 分析 |

---

### 六、进程排查

| 命令 | 作用 |
|---|---|
| ⭐ `ps aux` / `ps -ef` | 所有进程快照 |
| ⭐ `ps aux --sort=-%cpu \| head` | CPU Top N |
| ⭐ `ps aux --sort=-%mem \| head` | 内存 Top N |
| `pgrep -af nginx` | 按名字找 PID |
| `pstree -p` | 进程树 |
| `lsof -p <PID>` | 进程打开的所有文件/端口/连接 |
| `strace -p <PID>` | 系统调用跟踪（在线诊断卡死） |

---

### 七、日志 & 内核事件

| 命令 | 作用 |
|---|---|
| `dmesg -T \| tail -50` | 内核日志（OOM、硬盘错误、网卡 down 都在这） |
| `journalctl -xe` | systemd 全局日志 |
| `journalctl -u nginx -f` | 跟踪某个服务的日志 |
| `tail -f /var/log/messages` | 老系统的系统日志 |
| `grep -i "out of memory" /var/log/messages` | 排查 OOM Killer 记录 |

---

### 八、日常巡检「一键三连」推荐流程 🚀

排查一台可疑的云主机，建议按这个顺序敲：

```bash
# 1. 30 秒看全景
uptime && free -h && df -h

# 2. 找 CPU 和内存大户
top       # 按 P 看 CPU，按 M 看内存，按 q 退出

# 3. 磁盘 IO 是否是瓶颈
iostat -x 2 3

# 4. 网络带宽 & 连接
sar -n DEV 2 3    # 或 iftop -i eth0
ss -s

# 5. 有没有系统级异常
dmesg -T | tail -30
```

---

### 九、云厂商专有工具补充 ☁️

各家云厂商都提供了 **云监控（Cloud Monitor）** 面板，比命令行更适合看**历史趋势**和**告警**：

| 云厂商 | 监控平台 | Agent 命令 |
|---|---|---|
| 腾讯云 | 云监控 CM | `barad-agent`（自动安装） |
| 阿里云 | 云监控 CloudMonitor | `cloudmonitor` |
| 华为云 | 云监控 CES | `telescope` |
| AWS | CloudWatch | `amazon-cloudwatch-agent` |

**命令行 vs 云监控 怎么选？**

- **命令行**：适合**实时**深入排查、看瞬时抖动、抓现场
- **云监控**：适合看**趋势**、设置阈值告警、跨机聚合；**公网带宽 / 云盘带宽**这些**外部指标**云监控更准，因为它是在宿主机层面统计的

---

### 十、进阶方向

更系统的性能分析方法论推荐 Brendan Gregg 的两张经典图（可搜「Linux Performance Observability Tools」）：

- **USE 方法**：对每个资源看 **U**tilization（使用率）、**S**aturation（饱和度）、**E**rrors（错误）
- **60 秒诊断清单**：`uptime` → `dmesg` → `vmstat` → `mpstat` → `pidstat` → `iostat` → `free` → `sar -n DEV` → `sar -n TCP,ETCP` → `top`

---

## 27. journalctl 单个服务日志过多的清理与重置方法

### 问题
`journalctl -u [servicename]` 的日志太多了，怎么重置？

### 解答

`journalctl` 本身**没有直接清空单个服务日志的命令**（journald 是集中式日志系统，日志按二进制格式统一存储在 `/var/log/journal/`，无法只删除某个 unit 的记录）。可以通过以下几种方式达到「重置」或「只看最新」的效果。

---

### 方式一：只想「重新开始看」（推荐，无需清空）✅

这是最常见也最安全的做法 —— **不删日志，只是让视图聚焦到最新**。

| 命令 | 作用 |
|---|---|
| `journalctl -u 服务名 -f` | 实时追踪，仅显示新日志，不加载历史 |
| `journalctl -u 服务名 -n 50` | 只看最近 50 行 |
| `journalctl -u 服务名 --since "1 hour ago"` | 从 1 小时前开始 |
| `journalctl -u 服务名 --since today` | 从今天 00:00 开始 |
| `journalctl -u 服务名 --since "2026-07-10 09:00"` | 从指定时间开始 |
| `journalctl -u 服务名 -p err` | 只看 error 及以上级别 |

**优点**：不会丢失任何历史日志，随时可以回溯排查。

---

### 方式二：确实想「清空」旧日志（全局操作）⚠️

**注意**：`journalctl` 的清理是**系统级**的，会影响**所有服务**的日志，而不是单个服务。执行前请确认不需要保留历史。

```bash
# 1. 轮转当前日志（把活跃日志文件归档，开启新文件）
sudo journalctl --rotate

# 2. 按时间清理：只保留最近 1 天
sudo journalctl --vacuum-time=1d

# 或按大小清理：只保留 100M
sudo journalctl --vacuum-size=100M

# 或按文件数清理：只保留最新的 5 个归档文件
sudo journalctl --vacuum-files=5
```

**执行流程**：先 `--rotate` 关闭当前活跃日志文件（否则活跃文件不会被 vacuum 清理），再 `--vacuum-*` 按条件删除旧的归档文件。

**长期自动化**：可以在 `/etc/systemd/journald.conf` 里配置保留策略，避免手动清理：

```ini
[Journal]
SystemMaxUse=500M       # 全部日志最多占 500M
SystemMaxFileSize=50M   # 单个日志文件最大 50M
MaxRetentionSec=1week   # 最长保留 1 周
```

修改后重启 journald 生效：`sudo systemctl restart systemd-journald`。

---

### 方式三：把服务日志独立到文件（长期方案）🔧

如果**只想单独管理某个服务**（真正实现「清空某个服务的日志」），可以修改该服务的 systemd unit 文件，把标准输出重定向到独立文件。

#### 步骤

1. 编辑服务文件，例如 `/etc/systemd/system/yourapp.service`：

```ini
[Service]
ExecStart=/usr/local/bin/yourapp
StandardOutput=append:/var/log/yourapp.log
StandardError=append:/var/log/yourapp-error.log
```

> 说明：`file:` 每次启动会**截断**旧文件；`append:` 会**追加**。生产建议用 `append:`，配合 logrotate 做切割。

2. 重载并重启服务：

```bash
sudo systemctl daemon-reload
sudo systemctl restart yourapp
```

3. 之后就可以像普通日志文件一样处理：

```bash
# 清空日志（保留文件描述符，不影响正在写入的进程）
sudo truncate -s 0 /var/log/yourapp.log

# 或直接清空
sudo sh -c '> /var/log/yourapp.log'

# 配合 logrotate 自动切割
sudo vim /etc/logrotate.d/yourapp
```

---

### 三种方式对比

| 方式 | 影响范围 | 是否删除历史 | 适用场景 |
|---|---|---|---|
| **一、只看最新** | 无 | ❌ 不删 | 日常排查，只想聚焦最新事件 |
| **二、vacuum 清理** | **全部服务** | ✅ 全删 | 磁盘紧张、系统日志占用过大 |
| **三、独立到文件** | 单个服务 | ✅ 可单独清空 | 需要长期单独管理某个服务日志 |

---

### 建议 💡

1. **排查问题** → 用 `-f` / `-n` / `--since` 即可，**无需清空**。
2. **磁盘告急** → 用 `journalctl --vacuum-*` 做全局清理，并在 `journald.conf` 里配置长期保留策略。
3. **频繁需要独立管理某服务日志** → 采用方式三，把日志重定向到独立文件，再配合 `logrotate` 自动切割。

---

### 补充：`truncate` vs `rm` 清空日志文件的区别

如果采用方式三，**千万别用 `rm` 删日志文件**！因为很多进程启动时就打开了这个文件描述符，`rm` 只是删除目录项，**文件本身仍被进程持有**，磁盘空间不会释放，且新日志还会继续写入这个「已被删除的 inode」，直到进程重启才会真正回收。

正确做法：

```bash
# ✅ 推荐：truncate 保留文件、清零内容
sudo truncate -s 0 /var/log/yourapp.log

# ✅ 也可以：用 shell 重定向清空
sudo sh -c '> /var/log/yourapp.log'

# ❌ 错误：rm 会导致磁盘空间无法立即回收
sudo rm /var/log/yourapp.log
```

---

## 28. 在云服务器上把应用注册为 systemd 系统级服务

### 问题
如何在云服务器上把某个应用注册为系统级服务（以 `mildlab` 为例）？unit 文件各字段含义是什么？如何设置开机自启、启动、重启、关闭？还有什么需要补充的？

### 解答

把应用注册为 systemd 服务后，可以获得：**开机自启、崩溃自动拉起、统一日志接入 journald、标准化启停命令、优雅退出、资源限制、安全隔离**等一整套系统级能力。下面是完整流程。

---

### 一、创建 unit 文件

在 `/etc/systemd/system/` 目录下新建以 `.service` 结尾的文件，文件名就是服务名：

```bash
sudo vim /etc/systemd/system/mildlab.service
```

**为什么放在 `/etc/systemd/system/`？** systemd 从多个目录加载 unit，优先级从高到低：

- `/etc/systemd/system/` ← **管理员自定义**（自建服务放这里）
- `/run/systemd/system/` ← 运行时临时生成
- `/usr/lib/systemd/system/` ← 软件包安装时自带，**不要手动改**（升级时会被覆盖）

---

### 二、unit 文件各字段详解

典型模板：

```ini
[Unit]
Description=MildLab backend service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/mildlab
ExecStart=/opt/mildlab/bin/mildlab -config /opt/mildlab/config/config.yaml
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

#### 2.1 `[Unit]` 段：元信息与启动依赖

| 字段 | 含义 |
|---|---|
| `Description=` | 人类可读的服务描述，`systemctl status` 和 `journalctl` 都会显示 |
| `After=xxx.target` | 启动**顺序**依赖：本服务在 xxx **之后**启动。**只管顺序，不保证 xxx 一定启动** |
| `Wants=xxx.service` | **弱依赖**：会拉起 xxx，但 xxx 失败不影响本服务 |
| `Requires=xxx.service` | **强依赖**：xxx 必须成功启动，否则本服务也不起 |
| `Before=` | 启动顺序：本服务在 xxx 之前启动 |
| `ConditionPathExists=` | 条件启动，路径不存在则跳过 |
| `StartLimitIntervalSec=` / `StartLimitBurst=` | 启动限流，防止崩溃时死循环拉起 |

**⚠️ `network.target` vs `network-online.target`（生产易踩坑）**：

- `network.target`：网络栈已加载，但**不保证有 IP、能连通外网**
- `network-online.target`：网络真正连通（可拨外网、DNS 可解析）

如果服务启动时要连数据库、拉配置中心、访问远程 API，必须用后者：

```ini
[Unit]
Wants=network-online.target
After=network-online.target
```

#### 2.2 `[Service]` 段：如何启动/停止/重启

**（1）`Type=` 进程类型**（决定 systemd 如何判定「启动成功」）：

| 值 | 含义 | 适用场景 |
|---|---|---|
| ⭐ `simple` | ExecStart 启动的进程本身就是主进程，不 fork | 大多数 Go / Python / Node.js 应用 |
| `exec` | 类似 simple，等 execve() 成功才算就绪 | 更精确的现代应用 |
| `forking` | 主进程 fork 出子进程后自己退出，systemd 跟踪子进程 | 传统守护进程（老式 nginx、mysqld） |
| `oneshot` | 跑完一次就退出 | 一次性初始化脚本、备份任务 |
| `notify` | 进程主动调 `sd_notify()` 通知就绪 | 需要精确就绪信号的服务 |

**（2）`User=` / `Group=` 运行身份**：

- 决定服务以哪个 Linux 用户运行。
- **不推荐用 root**！被利用就是灾难。生产建议创建专用低权限用户：
  ```bash
  sudo useradd -r -s /sbin/nologin -d /opt/mildlab mildlab
  sudo chown -R mildlab:mildlab /opt/mildlab
  ```
- 若必须监听 <1024 端口，可用 `AmbientCapabilities=CAP_NET_BIND_SERVICE` 替代 root。

**（3）`WorkingDirectory=` 工作目录**：

- 相当于启动前 `cd` 到该目录。
- 影响程序里所有**相对路径**（如 `./config/`、`./logs/`）。
- **建议永远显式设置**，避免相对路径踩坑。

**（4）`ExecStart=` 启动命令** ⭐（最核心）：

- **必须写绝对路径**（systemd 不解析 `$PATH`）。
- **不能用 shell 特性**：管道 `|`、重定向 `>`、通配符 `*`、`&&` 都不生效。若必须要用：
  ```ini
  ExecStart=/bin/bash -c "cmd1 && cmd2"
  ```
- 相关字段：
  - `ExecStartPre=` / `ExecStartPost=`：启动前/后钩子
  - `ExecStop=`：自定义停止命令（默认发 SIGTERM）
  - `ExecReload=`：`systemctl reload` 时执行的命令，如 `ExecReload=/bin/kill -HUP $MAINPID`

**（5）`Restart=` + `RestartSec=` 崩溃自动重启** ⭐：

| `Restart=` 值 | 含义 |
|---|---|
| `no`（默认） | 从不重启 |
| ⭐ `always` | 任何情况退出都重启（含正常退出） |
| `on-failure` | **仅在异常退出时重启**（多数场景推荐） |
| `on-abnormal` | 只在被信号杀死/超时时重启 |
| `unless-stopped` | 除非 `systemctl stop` 手动停止，否则一直重启 |

- `RestartSec=5`：崩溃后等 5 秒再重启，避免死循环打爆 CPU。
- 推荐搭配启动限流：
  ```ini
  [Unit]
  StartLimitIntervalSec=60
  StartLimitBurst=5
  # 60 秒内最多重启 5 次，超了就标记为 failed 不再拉起
  ```

**（6）`StandardOutput=` / `StandardError=` 日志去向**：

| 值 | 含义 |
|---|---|
| ⭐ `journal` | 输出到 journald，用 `journalctl -u xxx` 查看（默认推荐） |
| `null` | 丢弃 |
| `file:/path/to/log` | 写到文件（每次启动会**覆盖**） |
| `append:/path/to/log` | 追加写文件（不覆盖，配合 logrotate 用） |
| `truncate:/path/to/log` | 启动时截断再写 |

**（7）其他常用字段**：

| 字段 | 用途 |
|---|---|
| `Environment="KEY=VALUE"` | 设置环境变量 |
| `EnvironmentFile=/etc/mildlab/env` | 从文件加载环境变量（改配置不用改 unit） |
| `LimitNOFILE=65535` | 文件描述符上限（高并发服务必设） |
| `LimitNPROC=8192` | 进程数上限 |
| `TimeoutStartSec=30` | 启动超时 |
| `TimeoutStopSec=30` | 停止超时，超过强杀 |
| `KillSignal=SIGTERM` | 停止时发送的信号 |
| `KillMode=mixed` | 主进程收 SIGTERM，子进程收 SIGKILL |
| `PIDFile=/run/mildlab.pid` | `Type=forking` 时必需 |

**安全加固字段**（生产推荐）：

| 字段 | 作用 |
|---|---|
| `NoNewPrivileges=true` | 禁止提权 |
| `ProtectSystem=strict` | `/usr`、`/boot` 只读 |
| `ProtectHome=true` | 隐藏 `/home` |
| `PrivateTmp=true` | 独立 `/tmp` 命名空间 |
| `ReadWritePaths=/opt/mildlab/data` | 白名单可写目录 |

#### 2.3 `[Install]` 段：启用规则

```ini
[Install]
WantedBy=multi-user.target
```

- **只有存在 `[Install]` 段的 unit 才能被 `systemctl enable`（开机自启）**。
- `WantedBy=multi-user.target`：当系统进入多用户命令行模式时，把本服务拉起来（云服务器几乎都是这种模式）。
- `systemctl enable` 的本质就是根据 `WantedBy` 创建软链接：
  ```
  /etc/systemd/system/multi-user.target.wants/mildlab.service
      → /etc/systemd/system/mildlab.service
  ```
- 其他常见值：
  - `graphical.target`：图形界面模式
  - `default.target`：系统默认目标

---

### 三、开机自启、启动、重启、关闭命令

#### 3.1 修改 unit 后必做的一步 ⭐

```bash
sudo systemctl daemon-reload   # 让 systemd 重读 unit 配置
```

> **每次修改 `.service` 文件后都要执行**，否则改动不生效 —— 这是新手最常踩的坑。

#### 3.2 开机自启

```bash
sudo systemctl enable mildlab            # 只启用自启，不立即启动
sudo systemctl enable --now mildlab      # ⭐ 启用自启 + 立即启动（一步到位）
sudo systemctl disable mildlab           # 取消自启
sudo systemctl disable --now mildlab     # 取消自启 + 立即停止
```

#### 3.3 启动、停止、重启、重载

```bash
sudo systemctl start mildlab             # 启动
sudo systemctl stop mildlab              # 停止
sudo systemctl restart mildlab           # 重启（先 stop 再 start）
sudo systemctl reload mildlab            # 重载配置（需程序支持 SIGHUP + unit 有 ExecReload=）
sudo systemctl reload-or-restart mildlab # 优先 reload，不支持则 restart
```

#### 3.4 状态与日志

```bash
sudo systemctl status mildlab            # 状态（含最近若干行日志）
sudo systemctl is-active mildlab         # active / inactive / failed
sudo systemctl is-enabled mildlab        # enabled / disabled

journalctl -u mildlab                    # 全部日志
journalctl -u mildlab -f                 # 实时跟踪
journalctl -u mildlab -n 100             # 最近 100 行
journalctl -u mildlab --since "10 min ago"
journalctl -u mildlab -p err             # 仅 error 以上
```

#### 3.5 完整上线流程

```bash
# 1. 编辑 unit 文件
sudo vim /etc/systemd/system/mildlab.service

# 2. 校验语法（可选）
sudo systemd-analyze verify /etc/systemd/system/mildlab.service

# 3. 加载配置
sudo systemctl daemon-reload

# 4. 启用并启动
sudo systemctl enable --now mildlab

# 5. 检查状态
sudo systemctl status mildlab
journalctl -u mildlab -f
```

---

### 四、生产补充建议 ✨

#### 4.1 专用低权限用户跑业务进程

```bash
sudo useradd -r -s /sbin/nologin -d /opt/mildlab mildlab
sudo chown -R mildlab:mildlab /opt/mildlab
```

对应 unit 改为 `User=mildlab`，即使被攻破也无法影响系统其他部分。

#### 4.2 环境变量从文件加载

```ini
EnvironmentFile=-/etc/mildlab/mildlab.env
ExecStart=/opt/mildlab/bin/mildlab -config /opt/mildlab/config/config.yaml
```

> `-` 前缀表示文件不存在也不报错。改环境变量后 `systemctl restart mildlab` 即可，不用改 unit。

#### 4.3 优雅退出（Go 程序建议）

程序要监听 `SIGTERM`，收到后释放连接、flush 日志、再退出：

```go
sig := make(chan os.Signal, 1)
signal.Notify(sig, syscall.SIGTERM, syscall.SIGINT)
<-sig
// 优雅关闭逻辑
```

这样 `systemctl stop` 才能干净地关闭服务，不会丢请求。

#### 4.4 高并发服务提升 ulimit

```ini
LimitNOFILE=65535
LimitNPROC=8192
```

#### 4.5 升级发布流程

```bash
# 1. 上传新二进制到 /opt/mildlab/bin/mildlab.new
# 2. 备份旧版本
sudo mv /opt/mildlab/bin/mildlab /opt/mildlab/bin/mildlab.prev
# 3. 原子替换
sudo mv /opt/mildlab/bin/mildlab.new /opt/mildlab/bin/mildlab
# 4. 重启
sudo systemctl restart mildlab
# 5. 验证
sudo systemctl status mildlab
journalctl -u mildlab -f
```

出问题立即回滚：

```bash
sudo mv /opt/mildlab/bin/mildlab.prev /opt/mildlab/bin/mildlab
sudo systemctl restart mildlab
```

#### 4.6 常见排错

| 现象 | 排查方向 |
|---|---|
| `systemctl start` 后立刻 failed | `journalctl -u mildlab -n 50` 看具体错误；多为路径错、权限错、端口占用 |
| `Failed to load environment files` | `EnvironmentFile` 路径写错 |
| 修改后不生效 | 忘了 `daemon-reload` |
| 服务反复重启 | 加 `StartLimitBurst` 限流，先解决崩溃根因 |
| `Permission denied` | 检查 `User` 是否对可执行文件、`WorkingDirectory` 有权限 |

#### 4.7 卸载服务

```bash
sudo systemctl disable --now mildlab
sudo rm /etc/systemd/system/mildlab.service
sudo systemctl daemon-reload
sudo systemctl reset-failed
```

---

### 五、增强版模板（生产推荐）

综合以上建议，一个更完善的模板如下：

```ini
[Unit]
Description=MildLab backend service
Documentation=https://internal.wiki/mildlab
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=60
StartLimitBurst=5

[Service]
Type=simple
User=mildlab
Group=mildlab
WorkingDirectory=/opt/mildlab
EnvironmentFile=-/etc/mildlab/mildlab.env
ExecStart=/opt/mildlab/bin/mildlab -config /opt/mildlab/config/config.yaml
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5
TimeoutStopSec=30
KillSignal=SIGTERM
KillMode=mixed
LimitNOFILE=65535
LimitNPROC=8192
StandardOutput=journal
StandardError=journal
# 安全加固
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/mildlab/data /opt/mildlab/logs

[Install]
WantedBy=multi-user.target
```

---

## 29. 为什么生产环境中要用低权限用户跑业务进程

### 问题
在 systemd unit 中经常看到「不要用 root 跑业务进程，应该创建专用低权限用户」这类建议。为什么必须这么做？背后的原理是什么？

### 解答

这是一条**运维安全的黄金法则**，本质是**最小权限原则（Principle of Least Privilege, PoLP）**：**任何进程只应拥有完成自身工作所需的最小权限，多一分都不给。**

---

### 一、核心威胁模型：进程权限 = 攻击者拿到的权限 ⭐

安全的第一定律是：**任何面向网络的进程，都要假设它总有一天会被攻破**。

一旦业务进程被攻破（RCE、反序列化漏洞、SQL 注入拿到 shell 等），攻击者就会**继承这个进程的所有权限**。于是问题变成了：**你希望攻击者拿到什么级别的权限？**

| 运行身份 | 被攻破后攻击者能做什么 |
|---|---|
| **root** | 🔴 读所有文件（含 `/etc/shadow`）、装后门、加内核模块、改防火墙、擦日志……**整机沦陷** |
| **专用低权限用户 mildlab** | 🟢 只能读写 `/opt/mildlab/`、只能杀自己的进程，系统文件动不了、其他服务动不了 |

**核心差别**：一个 CVE、一个 Log4Shell、一个反序列化漏洞，若用 root 跑就是**整机沦陷**；若用低权限用户跑，损害被**牢牢限制在业务目录内**。

---

### 二、用 root 跑业务进程的六大具体危害

1. **🔴 读取系统敏感文件**  
   `/etc/shadow`（密码哈希）、`/root/.ssh/`、所有用户的 `.ssh/id_rsa` 私钥——拖走 shadow 就能离线暴破 root 密码。

2. **🔴 修改系统关键配置**  
   - 改 `/etc/passwd` 加隐藏账户
   - 改 `authorized_keys` 植入 SSH 后门
   - 注册恶意 systemd 服务（开机自启后门）
   - 改 `iptables`/`firewalld` 开放任意端口
   - 加载内核 rootkit

3. **🔴 横向感染其他服务**  
   服务器上每个服务（数据库、缓存、其他业务）的**数据文件、socket、配置**都能被 root 读写。例：MySQL 的 datadir 通常 `mysql:mysql 700`，但 root 无视权限直接搬走 InnoDB 文件——**即使没有密码，所有数据库表都泄露**。

4. **🔴 擦除日志、消除痕迹**  
   `/var/log/*`、`journalctl --vacuum`、`wtmp`/`btmp`、shell history 都能改，**取证都做不了**。

5. **🔴 挖矿、DDoS、成为跳板**  
   挖矿占满 CPU、加定时任务复活后门、作为跳板攻击内网、参与 DDoS——**可能带来运营商投诉甚至法律责任**。

6. **🔴 突破容器边界**  
   在 Docker/K8s 中，容器内 root 通过某些 CVE（如 CVE-2019-5736、CVE-2022-0492）可以**逃逸到宿主机**；非 root 逃逸门槛陡增。

---

### 三、除了安全，还有这些日常好处

#### 1. 🛡️ 防止业务 bug 造成灾难性误操作

```go
os.RemoveAll(cfg.WorkDir) // 若 WorkDir 因配置错误变成 "/"
```

- **root 跑**：整机 `rm -rf /`，**服务器 GG**
- **mildlab 跑**：大部分系统文件无权限删除，**只删自己家里的东西**，损失可控

**低权限用户 = 给失控代码兜底**。

#### 2. 🛡️ 与其他服务隔离

各业务各用各的专用用户，**每个服务只能碰自己的目录**，一个坏掉不牵连其他。

#### 3. 🛡️ 权限即文档

`ls -l /opt/mildlab/` 一眼看出：这个目录属于 mildlab 用户，**排查问题时权限本身就是最好的文档**。

#### 4. 🛡️ 符合审计合规

金融、政企、等保二/三级、ISO27001、SOC2 等审计**明确要求**业务进程不得以特权账户运行。

#### 5. 🛡️ 防止意外「顺手」修改

手一滑 `vim /etc/hosts` 就改了——用 mildlab 跑，想改都改不了，**给危险操作加了确认闸门**。

---

### 四、真实反面教材

#### 案例 1：Redis 未授权 + root 运行 → 服务器沦陷（教科书级）

- Redis 6379 无密码、绑 `0.0.0.0`
- Redis 以 root 启动
- 攻击者利用 `CONFIG SET dir /root/.ssh/` + `CONFIG SET dbfilename authorized_keys` + `SET x "ssh-rsa xxx"` + `SAVE`
- **成功把公钥写入 root 的 authorized_keys → SSH 免密登录整机**

若 Redis 用 `redis` 用户跑，`/root/.ssh/` 根本没写权限，攻击直接失败。

#### 案例 2：Log4Shell（CVE-2021-44228）

大量 Java 服务被 RCE，**用 root 跑 Tomcat/Java 应用**的公司损失惨重；遵循最佳实践用 `tomcat` 专用用户跑的，攻击者只能碰 webapp 目录，为应急响应争取时间。

#### 案例 3：容器逃逸

CVE-2022-0847（Dirty Pipe）允许容器内任意用户改写只读文件——**容器内 root** 可改写宿主机 SUID 二进制实现提权；**容器内非 root** 影响则小得多。

---

### 五、正确做法（结合 systemd 落地）

#### 1. 创建专用系统用户

```bash
sudo useradd -r -s /sbin/nologin -d /opt/mildlab mildlab
```

- `-r`：**系统账户**（UID < 1000，不占普通用户段）
- `-s /sbin/nologin`：**禁止 shell 登录**——即使密钥泄露也无法交互式登录
- `-d /opt/mildlab`：家目录

#### 2. 严格设置目录权限

```bash
sudo chown -R mildlab:mildlab /opt/mildlab
sudo chmod 750 /opt/mildlab              # 拥有者读写执行、同组读执行、其他无权
sudo chmod 640 /opt/mildlab/config/*.yaml # 配置文件更严格
```

#### 3. systemd unit 配置

```ini
[Service]
User=mildlab
Group=mildlab
WorkingDirectory=/opt/mildlab
ExecStart=/opt/mildlab/bin/mildlab -config /opt/mildlab/config/config.yaml
```

#### 4. 加一层安全加固

```ini
NoNewPrivileges=true       # 禁止 setuid 提权
ProtectSystem=strict       # /usr /boot 只读
ProtectHome=true           # 隐藏 /home
PrivateTmp=true            # 独立 /tmp
ReadWritePaths=/opt/mildlab/data /opt/mildlab/logs  # 白名单可写目录
```

即使被 RCE，攻击者只能在白名单目录内活动。

---

### 六、看似需要 root 的场景 & 优雅替代方案

#### 场景 1：监听 <1024 端口（80/443）

❌ 用 root 跑整个服务  
✅ 授予**能力（capability）**：

```ini
[Service]
User=mildlab
AmbientCapabilities=CAP_NET_BIND_SERVICE
```

或对二进制永久授权：

```bash
sudo setcap 'cap_net_bind_service=+ep' /opt/mildlab/bin/mildlab
```

#### 场景 2：读 `/proc/*/io`、发 raw socket

```ini
AmbientCapabilities=CAP_SYS_PTRACE   # 或 CAP_NET_RAW
```

单独授能力，而非整个 root。

#### 场景 3：需要操作 systemd

写 D-Bus policy 授权特定接口，而不是给 root。

#### 场景 4：真·必须 root（系统工具/内核模块）

应**短暂持有 root，做完立刻降权**（`setuid()`），并在文档中明确标注、纳入更严格的漏洞跟踪。

---

### 七、一句话总结

> **root 是「上帝权限」，一旦泄露就是整机沦陷；专用低权限用户则把爆炸半径牢牢锁死在业务目录里。**  
> **在生产环境里，多花两条命令的成本，换来的是「一旦出事，你还能收拾局面」的保命底牌。**

从**安全**（防被攻破后横向扩散）、**稳定**（防 bug 误删）、**隔离**（防服务互相影响）、**合规**（过审计）、**运维**（权限即文档）五个维度看，**「业务进程绝不用 root 跑」应视为一条无条件的生产铁律**。

---

## 30. 云服务器上查看当前有哪些用户

### 问题
云服务器上查看当前有哪些用户？

### 解答

查看云服务器上的用户主要分两种情况：**当前登录的用户**和**系统里存在的所有用户**。前者看「现在谁在操作机器」，后者看「系统上有哪些账号」（含服务账号）。

---

### 一、👤 查看当前登录到服务器的用户

这些命令告诉你「谁正在操作这台机器」，信息来源主要是 `/var/run/utmp`。

| 命令 | 作用 | 典型输出 |
|---|---|---|
| `who` | 显示当前登录的用户名、终端、登录时间、来源 IP | `root pts/0 2026-07-10 10:32 (1.2.3.4)` |
| `w` | 更详细：用户 + 当前命令 + 系统负载（load average） | 包含 IDLE、JCPU、PCPU、WHAT 列 |
| `users` | 仅列出当前登录用户名（去重） | `root ubuntu` |
| `last` | 历史登录记录（来自 `/var/log/wtmp`） | 含登入登出时间与来源 IP |
| `lastlog` | 每个用户**最后一次**登录时间 | 排查可疑账号时很好用 |

```bash
who          # 当前登录用户
w            # 更详细：包含在执行什么命令
users        # 只看用户名
last -n 20   # 最近 20 条登录历史
lastlog      # 所有用户的最后登录记录
```

> 💡 `who` 和 `w` 看的是 **实时在线**的 session；`last` 看的是历史，可用来审计异常登录。

---

### 二、📋 查看系统里所有用户（账户）

系统账号信息存在 `/etc/passwd`，**包含真人用户和服务/守护进程账号**（如 mysql、nginx、sshd）。

#### 1. 看全部（含字段）

```bash
cat /etc/passwd
```

`/etc/passwd` 行格式：`用户名:密码占位(x):UID:GID:描述:家目录:登录shell`。

#### 2. 只列用户名

```bash
cut -d: -f1 /etc/passwd
```

#### 3. 只看可登录的普通用户（UID ≥ 1000）

```bash
awk -F: '$3>=1000 && $1!="nobody"{print $1}' /etc/passwd
```

或按 shell 过滤：

```bash
grep -E ":/bin/bash|:/bin/sh" /etc/passwd | cut -d: -f1
```

> 💡 UID 约定：`0` = root；`1~999` = 系统/服务账号；`≥1000` = 普通用户（部分发行版为 500）。

---

### 三、🔍 快速查找特定用户是否存在

```bash
id 用户名              # 有就列出 UID、GID、所属组；无则报 "no such user"
getent passwd 用户名   # 兼容 LDAP/NIS 等远程账户库
grep "^用户名:" /etc/passwd
```

---

### 四、🔐 查看拥有 sudo 权限的用户

```bash
sudo cat /etc/sudoers            # 主配置
sudo ls /etc/sudoers.d/          # 分模块配置（推荐放新规则的位置）
getent group sudo                # Ubuntu/Debian：sudo 组成员
getent group wheel               # CentOS/RHEL：wheel 组成员
```

> 💡 修改 sudoers **必须用 `visudo`**，它会语法校验，防止写错导致 sudo 完全不可用。

---

### 五、⚠️ 安全小贴士（异常账号排查思路）

1. **除 root 外 UID=0 的账号 → 高危**（后门典型手法）：
   ```bash
   awk -F: '$3==0 {print $1}' /etc/passwd
   ```
   正常情况下只应有 `root` 一行。

2. **未知账号且能登录 shell**：
   ```bash
   grep -E ":/bin/bash|:/bin/sh" /etc/passwd
   ```

3. **最近新增账号**：
   ```bash
   ls -lt /home/                    # 看家目录创建时间
   sudo tail -50 /var/log/secure    # RHEL/CentOS 认证日志
   sudo tail -50 /var/log/auth.log  # Debian/Ubuntu 认证日志
   ```

4. **异常登录来源 IP**：
   ```bash
   last -a | head -30
   ```

> 💡 看到 `hacker:x:0:0:::/bin/bash` 这种行 → **基本可以确定被入侵**，立即断网保证现场、备份、重装。

---

### 六、一张表快速回忆

| 需求 | 命令 |
|---|---|
| 现在谁在登录 | `who` / `w` / `users` |
| 历史登录 | `last` / `lastlog` |
| 所有账号 | `cat /etc/passwd` / `cut -d: -f1 /etc/passwd` |
| 只看真人用户 | `awk -F: '$3>=1000{print $1}' /etc/passwd` |
| 某个用户存不存在 | `id 名字` / `getent passwd 名字` |
| 权限大的用户 | `awk -F: '$3==0{print $1}' /etc/passwd` 、`getent group sudo` |
| sudo 配置 | `sudo cat /etc/sudoers` 、`ls /etc/sudoers.d/` |

---

### 七、一句话总结

> **`/var/run/utmp` 告诉你「谁在在线」，`/etc/passwd` 告诉你「有哪些账号」，`last` 告诉你「过去谁登过」。**  
> **日常用 `w`、审计用 `last`、排查入侵用 `awk` 扫 `UID=0` 与 `/bin/bash` 行，三招就能看住大部分场景。**

---


