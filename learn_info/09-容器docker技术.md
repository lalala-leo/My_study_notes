## 01-docker安装部署

```bash
# 1.流量转发
cat > /etc/sysctl.d/docker.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1
EOF

# 2.还是流量转发
modprobe br_netfilter

# 3.下载repo
curl -o /etc/yum.repos.d/Centos-7.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 4.下载docker的repo
curl -o /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 5.清缓存
yum clean all && yum makecache

# 6.下载docker
sudo yum install docker-ce-20.10.12 --nogpgcheck -y

docker --version

# 7.改Docker Client的代理（不需要）
vim /etc/profile 

export https_proxy=http://127.0.0.1:7890 
export http_proxy=http://127.0.0.1:7890 

source /etc/profile 

# 8.改Docker Daemod代理
sudo mkdir -p /etc/systemd/system/docker.service.d

vim /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
#不知道为什么这个可以不需要，只需要底下那个
#Environment="HTTP_PROXY=http://192.168.1.14:7890/"
Environment="HTTPS_PROXY=http://192.168.1.14:7890/"

# 9.启动docker，测试拉镜像
systemctl enable docker  
systemctl daemon-reload
systemctl start docker 

docker search nginx

docker pull  nginx
```



### 自制一个dockers镜像

```bash
docker pull centos:7.4.1708

docker run -it centos:7.4.1708 bash

# 1.基础环有了

# 2.更新yum源，没有wget不建议装，选curl，为了精简化docker镜像体积

# 清空原有yum源
rm -rf /etc/yum.repo.d/*

# vim net-tools
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

# nginx
curl -o /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo

# 不需要缓存
yum install vim net-tools nginx -y

# 清空缓存，降低容器内资源占用
yum clean all




# 进入之前镜像
docker exec -it 426 bash

# 提交容器记录，生成新的镜像记录
docker commit 426a85f0a669 leo018/leotest-nginx-vim-centos7

# 删除所有容器
docker rm `docker ps -aq`

# 一条命令直接执行
docker run -d -p 80:80 leo018/leotest-nginx-vim-centos7 nginx -g "daemon off;"

```



### 推送练习镜像到docker hub

```bash
https://hub.docker.com/

docker pull hello-world

docker images

docker run hello-world  # 2个作用 1，下载镜像 2. 创建容器空间，执行镜像内容

# 改标签名，得将镜像名，改为以 docker hub 账号开头的规则
docker tag hello-world:latest leo018/leotest-hello-docker

docker login
leo018
Ww3581018

# 推送到docker hub
docker push leo018/leotest-hello-docker

# 拉取到本地
docker pull leo018/leotest-hello-docker:latest

```



## 02-docker的存储与网络



### 数据卷映射

```bash
mkdir /xiaoniao-all

mkdir -p /xiaoniao-all/xiaoniao

mkdir -p /xiaoniao-all/conf.d

mkdir -p /xiaoniao-all/logs

cat /xiaoniao-all/conf.d/xiaoniao.conf
server {
  listen 81;
  server_name localhost;
	
  root /xiaoniao;
  index index.html;
}


# -v 宿主机目录:容器内的目录 

docker run -d  -p 81:81  -p 80:80   \
-v /xiaoniao-all/conf.d/:/etc/nginx/conf.d/ \
-v /xiaoniao-all/xiaoniao/:/xiaoniao/ \
-v /xiaoniao-all/logs/:/var/log/nginx/ \
nginx:1.15.11


```





## 03-docker file学习

### 构建python应用镜像

```bash
# 需求，基于dockerfile，构建 python3的运行环境，安装好的一个django模块
# 跑一个能运行django 页面的平台，代码生成

# 思路，手工部署的逻辑
1. 基础运行系统环境，centos
2. 编译，rpm，yum， yum install python3 python3-devel 
3. 安装python3的模块  ，  pip3  install  django



```

