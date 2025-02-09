# Rocky的基础

## 下载系统地址

```bash
https://rockylinux.org/zh-CN/download
```



## 关闭防火墙

```bash
# 所有节点关闭防火墙
systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld

# 清空所有规则iptables
yum install iptables-services -y
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
iptables -t raw -F
iptables -t raw -X
service iptables save
systemctl enable iptables


# 禁用 Selinux
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
grubby --update-kernel ALL --args selinux=0
# 查看是否禁用, grubby --info DEFAULT
# 回滚内核层禁用操作, grubby --update-kernel ALL --remove-args selinux

# 禁用 Swap
swapoff -a
sed -i '/swap/d' /etc/fstab
```



## 修改网卡

```bash
# 我们打开配置文件修改固定ip地址
vim /etc/NetworkManager/system-connections/ens160.nmconnection

[connection]
id=ens160
uuid=4f4e3ee3-37d4-3efa-a6a8-0ab8e2f47ff3
type=ethernet
autoconnect-priority=-999
interface-name=ens160
timestamp=1729933754

[ethernet]

[ipv4]
method=auto # 修改成手动 method=manual

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]


# 修改后的配置
[connection]
id=ens160
uuid=4f4e3ee3-37d4-3efa-a6a8-0ab8e2f47ff3
type=ethernet
autoconnect-priority=-999
interface-name=ens160
timestamp=1729933754

[ethernet]

[ipv4]
method=manual
addresses=192.168.50.25/24
gateway=192.168.50.1
dns=223.5.5.5



[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]

# 执行以下命令重新加载并激活连接
nmcli connection reload
nmcli connection down ens33
nmcli connection up ens33

```



## 优化变量

```bash
echo 'export PS1="[\[\e[34;1m\]\u@\[\e[0m\]\[\e[32;1m\]\H\[\e[0m\] \[\e[31;1m\]\w\[\e[0m\]]\\$"' >> /etc/profile
```



## 安装docker

```bash
# 添加 Docker 的官方仓库安装最新版本
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 安装 Docker 和相关组件
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 启动 Docker 并设置开机自启
sudo systemctl start docker
sudo systemctl enable docker

# 或者网络不好的直接配置阿里源docker镜像
https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.57e31b11CqWwJg

# 以下是正确步骤
##################################################

# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils

# Step 2: 添加软件源信息
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# Step 3: 安装Docker
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Step 4: 开启Docker服务
sudo service docker start

##################################################


#使用代理拉取镜像
mkdir -p /etc/systemd/system/docker.service.d
vim /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTPS_PROXY=http://192.168.1.11:7890/"

systemctl daemon-reload
systemctl restart docker
```



## 修改yum源

```bash
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rockylinux|g' \
    -i.bak \
    /etc/yum.repos.d/rocky*.repo

dnf makecache


```



## 脚本

```bash
cat /init_linux.sh

read -p 'please input ip:' my_ip
read -p 'please input hostname:' host_name

echo 'updating network config file'
sed -i "/addresses/s#32#${my_ip}#g" /etc/NetworkManager/system-connections/ens33.nmconnection 
echo 'network have been updated'
echo '++++++++++++++++++++++++++++++++++++++++++++'

echo 'updating hostname'
hostnamectl set-hostname ${host_name}
echo 'hostname have been updated'
echo '++++++++++++++++++++++++++++++++++++++++++++'

echo 'current network file:' `cat /etc/NetworkManager/system-connections/ens33.nmconnection`
echo '++++++++++++++++++++++++++++++++++++++++++++'

echo 'current hostname:' `hostname`


```



