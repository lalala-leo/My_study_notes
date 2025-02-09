# jumpserver部署

## 安装core

```bash
cd /opt
mkdir /opt/jumpserver-v3.10.15
wget -O /opt/jumpserver-v3.10.15.tar.gz https://github.com/jumpserver/jumpserver/archive/refs/tags/v3.10.15.tar.gz
tar -xf jumpserver-v3.10.15.tar.gz -C /opt/jumpserver-v3.10.15 --strip-components 1
cd jumpserver-v3.10.15
rm -f apps/common/utils/ip/geoip/GeoLite2-City.mmdb apps/common/utils/ip/ipip/ipipfree.ipdb
wget https://download.jumpserver.org/files/ip/GeoLite2-City.mmdb -O apps/common/utils/ip/geoip/GeoLite2-City.mmdb
wget https://download.jumpserver.org/files/ip/ipipfree.ipdb -O apps/common/utils/ip/ipip/ipipfree.ipdb


# 安装依赖
cd requirements/

bash deb_pkg.sh

apt-get install -y pkg-config libxmlsec1-dev libpq-dev libffi-dev libxml2 libxslt-dev libldap2-dev libsasl2-dev sshpass mariadb-client bash-completion g++ make sshpass


```



## 安装mysql

```bash
# 默认装的就是8.0
apt install mysql-server mysql-client -y 

sudo apt-get install libmysqlclient-dev


# 进入mysql
mysql

# 创建用户
CREATE USER 'jumpserver'@'localhost' IDENTIFIED BY 'jumpserver';

# 授予用户'jumpserver'适当的权限。例如，如果你想将其作为数据库管理员：
GRANT ALL PRIVILEGES ON * . * TO 'jumpserver'@'localhost';

# 创建库
CREATE DATABASE jumpserver;

# 查看库
SHOW DATABASES;

# 刷新权限以使更改生效
FLUSH PRIVILEGES;

# 退出
exit;
```



## 安装redis

```bash
sudo apt update
sudo apt install redis-server -y 
sudo systemctl start redis
```





## 编译安装python

```bash
# 安装依赖
sudo apt update
sudo apt install build-essential zlib1g-dev libffi-dev libssl-dev -y

# 下载 Python 3.11 源代码：
wget https://www.python.org/ftp/python/3.11.0/Python-3.11.0.tgz
tar -xzvf Python-3.11.0.tgz
cd Python-3.11.0

# 配置、编译并安装：
./configure
make
sudo make install

# 验证安装：
python3.11 --version

# 安装语言包-这是一套处理多国语言的库和工具
sudo apt-get install gettext -y


# 创建虚拟环境
python3.11 -m venv /opt/py3
source /opt/py3/bin/activate


# 安装poetry
pip3 install poetry -i https://mirrors.aliyun.com/pypi/simple/

cd /opt/jumpserver-v3.10.15

poetry install

# 修改配置文件
cp config_example.yml config.yml
vi config.yml




# 处理国际化。他官网路径问题
rm -f apps/locale/zh/LC_MESSAGES/django.mo apps/locale/zh/LC_MESSAGES/djangojs.mo
python apps/manage.py compilemessages

# 我们需要删除自己安装路径下的文件即可
cd /opt/jumpserver-v3.10.15/apps/locale/en/LC_MESSAGES
# 删除
rm -rf djangojs.mo django.mo
# 然后在启动
python apps/manage.py compilemessages
```



## 生成密钥

```bash
# SECRET_KEY
cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 48;echo

m8NafdF8lSjX2Y6Fzo6T3FCyV2Ixec1nawuTwMoiWlT989jsi

# BOOTSTRAP_TOKEN
cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 24;echo

ShPf3u9cHDZJZ93WqumgHneS
```



## 运行core

```bash
./jms start

# 后台运行
nohup ./jms start &
```





## 部署lina

```bash
cd /opt
mkdir /opt/lina-v3.10.15
wget -O /opt/lina-v3.10.15.tar.gz https://github.com/jumpserver/lina/archive/refs/tags/v3.10.15.tar.gz
tar -xf lina-v3.10.15.tar.gz -C /opt/lina-v3.10.15 --strip-components 1
```



## 安装node

```bash
# 安装Node.js的包管理器npm：  yum  apt  pip 
sudo apt install npm -y

# 添加Node.js版本管理器nvm（Node Version Manager）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash

# 重新加载你的终端配置：
source ~/.bashrc

# 安装Node.js 16.13.0
nvm install 16.14.0

# 设定当前使用的Node.js版本为16.13.0
nvm use 16.14.0


# 验证安装是否成功，检查Node.js和npm版本
node -v
npm -v

# 配置源
npm install -g cnpm --registry=https://registry.npmmirror.com


# 安装依赖。
cd /opt/lina-v3.10.15/
npm install -g yarn
yarn install

# 修改配置文件
sed -i "s@Version <strong>.*</strong>@Version <strong>v3.10.15</strong>@g" src/layout/components/Footer/index.vue


cp .env.development.example .env.development


vim .env.development
```



## 运行lina

```bash
# 运行Lina
yarn serve

# 后台运行
nohup yarn serve &
```



## 安装luna

```bash
cd /opt
mkdir /opt/luna-v3.10.15
wget -O /opt/luna-v3.10.15.tar.gz https://github.com/jumpserver/luna/archive/refs/tags/v3.10.15.tar.gz
tar -xf luna-v3.10.15.tar.gz -C /opt/luna-v3.10.15 --strip-components 1

# 安装依赖。
cd /opt/luna-v3.10.15/
yarn install

# 修改配置文件
sed -i "s@[0-9].[0-9].[0-9]@v3.10.15@g" src/environments/environment.prod.ts

vi proxy.conf.json
```



## 运行luna

```bash
./node_modules/.bin/ng serve

# 后台运行
nohup ./node_modules/.bin/ng serve &
```



## 安装KoKo

```bash
cd /opt
wget https://download.jumpserver.org/public/kubectl-linux-amd64.tar.gz -O kubectl.tar.gz
tar -xf kubectl.tar.gz
mv kubectl /usr/local/bin/rawkubectl
wget https://download.jumpserver.org/public/helm-v3.9.0-linux-amd64.tar.gz
tar -xf helm-v3.9.0-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/rawhelm
chmod 755 /usr/local/bin/rawkubectl /usr/local/bin/rawhelm
chown root:root /usr/local/bin/rawkubectl /usr/local/bin/rawhelm
rm -rf linux-amd64
wget https://github.com/jumpserver/koko/releases/download/v3.10.15/koko-v3.10.15-linux-amd64.tar.gz
tar -xf koko-v3.10.15-linux-amd64.tar.gz -C /opt
cd koko-v3.10.15-linux-amd64
mv kubectl /usr/local/bin/kubectl

# 修改配置文件
cp config_example.yml config.yml
vi config.yml

ShPf3u9cHDZJZ93WqumgHneS

```



## 运行KoKo

```bash
./koko

# 后台运行
nohup ./koko &

```



## 安装lion

```bash
mkdir /opt/guacamole-v3.10.15
cd /opt/guacamole-v3.10.15
wget https://apache.org/dyn/closer.lua/guacamole/1.5.5/source/guacamole-server-1.5.5.tar.gz?action=download
tar -xzf guacamole-server-1.5.5.tar.gz
cd guacamole-server-1.5.5/

# 安装依赖
apt-get install -y libcairo2-dev libjpeg-turbo8-dev libpng-dev libtool-bin libossp-uuid-dev

apt-get install -y libavcodec-dev libavformat-dev libavutil-dev libswscale-dev freerdp2-dev libpango1.0-dev libssh2-1-dev libtelnet-dev libvncserver-dev libwebsockets-dev     libpulse-dev libssl-dev libvorbis-dev libwebp-dev

# 构建
./configure --with-init-dir=/etc/init.d
make
make install
ldconfig

# 下载lion
cd /opt
wget https://github.com/jumpserver/lion-release/releases/download/v3.10.15/lion-v3.10.15-linux-amd64.tar.gz
tar -xf lion-v3.10.15-linux-amd64.tar.gz
cd lion-v3.10.15-linux-amd64

# 修改配置文件
cp config_example.yml config.yml
vi config.yml

ShPf3u9cHDZJZ93WqumgHneS

```



## 运行lion

```bash 
# 启动这个
/etc/init.d/guacd start

# 后台运行
nohup ./lion &
```



## 安装Magnus

```bash
cd /opt
wget https://github.com/jumpserver/magnus-release/releases/download/v3.10.10/magnus-v3.10.10-linux-amd64.tar.gz
tar -xf magnus-v3.10.10-linux-amd64.tar.gz
cd magnus-v3.10.10-linux-amd64

wget https://github.com/jumpserver/wisp/releases/download/v0.1.16/wisp-v0.1.16-linux-amd64.tar.gz
tar -xf wisp-v0.1.16-linux-amd64.tar.gz
mv wisp-v0.1.16-linux-amd64/wisp /usr/local/bin/
chown root:root /usr/local/bin/wisp /opt/magnus-v3.10.10-linux-amd64/magnus
chmod 755 /usr/local/bin/wisp /opt/magnus-v3.10.10-linux-amd64/magnus

# 修改配置文件
cp config_example.yml config.yml
vi config.yml

BOOTSTRAP_TOKEN: ShPf3u9cHDZJZ93WqumgHneS



export CORE_HOST="http://127.0.0.1:8080"
export BOOTSTRAP_TOKEN=UMUL3XurotYq2su6IIQL3fAh     
export WORK_DIR="/opt/magnus-v3.10.10-linux-amd64"
export COMPONENT_NAME="magnus"
export EXECUTE_PROGRAM="/opt/magnus-v3.10.10-linux-amd64/magnus"
wisp

# 后台运行
nohup wisp &

```



## 安装nginx

```bash
apt-get install -y curl gnupg2 ca-certificates lsb-release ubuntu-keyring
echo "deb http://nginx.org/packages/ubuntu focal nginx" > /etc/apt/sources.list.d/nginx.list
curl -o /etc/apt/trusted.gpg.d/nginx_signing.asc https://nginx.org/keys/nginx_signing.key
apt-get update
apt-get install -y nginx
echo > /etc/nginx/conf.d/default.conf



# 缺少东西nginx不行

lsb_release -a

vim /etc/apt/sources.list
deb http://archive.ubuntu.com/ubuntu/ focal main universe

sudo apt update

sudo apt install libssl1.1

sudo apt --fix-broken install

# 手动安装
wget http://nginx.org/download/nginx-1.20.2.tar.gz
tar -xzvf nginx-1.20.2.tar.gz
cd nginx-1.20.2

sudo apt install build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev

./configure --with-http_ssl_module
make
sudo make install

sudo /usr/local/nginx/sbin/nginx
```



## 环境整合

```bash
vi /etc/nginx/conf.d/jumpserver.conf


server {
  listen 80;
  # server_name _;

  client_max_body_size 5000m; # 文件大小限制

  # Luna 配置
  location /luna/ {
    # 注意将模板中的组件名称替换为服务实际 ip 地址， 如都在本机部署
    # proxy_pass       http://127.0.0.1:4200;
    proxy_pass http://127.0.0.1:4200;
  }

  # Core data 静态资源
  location /media/replay/ {
    add_header Content-Encoding gzip;
    root /opt/jumpserver-v3.10.15/data/;
  }

  location /static/ {
    root /opt/jumpserver-v3.10.15/data/;
  }

  # KoKo Lion 配置
  location /koko/ {
    # 注意将模板中的组件名称替换为服务实际 ip 地址， 如都在本机部署
    # proxy_pass       http://127.0.0.1:5000;
    proxy_pass       http://127.0.0.1:5000;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_http_version 1.1;
    proxy_buffering off;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }

  # lion 配置
  location /lion/ {
    # 注意将模板中的组件名称替换为服务实际 ip 地址， 如都在本机部署
    # proxy_pass       http://127.0.0.1:8081;
    proxy_pass http://127.0.0.1:8081;
    proxy_buffering off;
    proxy_request_buffering off;
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
    proxy_ignore_client_abort on;
    proxy_connect_timeout 600;
    proxy_send_timeout 600;
    proxy_read_timeout 600;
    send_timeout 6000;
  }

  location /ws/ {
    # 注意将模板中的组件名称替换为服务实际 ip 地址， 如都在本机部署
    # proxy_pass       http://127.0.0.1:8080;
    proxy_pass http://127.0.0.1:8080;
    proxy_buffering off;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  location ~ ^/(core|api|media)/ {
    # 注意将模板中的组件名称替换为服务实际 ip 地址， 如都在本机部署
    # proxy_pass       http://127.0.0.1:8080;
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # 前端 Lina
  location /ui/ {
    # 注意将模板中的组件名称替换为服务实际 ip 地址， 如都在本机部署
    # proxy_pass       http://127.0.0.1:9528;
    proxy_pass http://127.0.0.1:9528;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  location / {
    rewrite ^/(.*)$ /ui/$1 last;
  }
}



nginx -t

nginx -s reload

systemctl restart nginx
```

