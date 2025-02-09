## ruoyi项目部署



### 项目仓库地址

```
https://gitee.com/y_project/RuoYi-Vue
```



### 部署帮助文档

```
https://doc.ruoyi.vip/ruoyi/document/hjbs.html#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C
```



### 部署操作

#### 部署前准备

```bash
# 安装git
yum install git -y

cd /opt

# 拉取项目源码
git clone https://gitee.com/y_project/RuoYi-Vue.git


vim /opt/RuoYi-Vue/ruoyi-admin/src/main/resources/logback.xml

    <!-- 日志存放路径 -->
        <property name="log.path" value="/var/log/ruoyi/logs" />
    <!-- 日志输出格式 -->


<charset>UTF-8</charset>

```



#### centos安装mysql8

```bash
# 下载安装
wget http://dev.mysql.com/get/mysql80-community-release-el7-8.noarch.rpm

yum localinstall -y mysql80-community-release-el7-8.noarch.rpm

yum -y install mysql-community-server --nogpgcheck

# 启动并登录
systemctl start mysqld

cat /var/log/mysqld.log | grep password

mysql -uroot -p'oBBk>Pw_y5(r'

# 修改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Password1@';

# 创建数据库表
CREATE DATABASE `ry-vue`;

# 查看表
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| ry-vue             |
| sys                |
+--------------------+
5 rows in set (0.00 sec)


# 导入数据
mysql -u root -p ry-vue < /opt/RuoYi-Vue/sql/quartz.sql

mysql -u root -p ry-vue < /opt/RuoYi-Vue/sql/ry_20240629.sql

====================================================

# 密码忘记，在/etc/my.cnf添加下免密登录
skip-grant-tables

systemctl restart mysqld

use mysql;
# 将密码置空
update user set authentication_string = '' where user = 'root';

# 删除my.cnf的免密登录，重新登录mysql
# 在修改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Password1@';


# 查看密码策略

SHOW VARIABLES LIKE 'validate_password%';

SET GLOBAL validate_password.policy = LOW;
SET GLOBAL validate_password.number_count = 0;
SET GLOBAL validate_password.special_char_count = 0;
SET GLOBAL validate_password.mixed_case_count = 0;

# 修改配置文件使修改的密码策略永久生效
# 打开MySQL的配置文件（通常是 mysqld.cnf 或 my.cnf），添加下面的内容到文件中：

validate_password.policy=LOW
validate_password.length=6
validate_password.number_count=1
validate_password.special_char_count=1
validate_password.mixed_case_count=1

```



#### 安装redis

```bash
yum install redis -y

systemctl start redis
```



#### 安装maven

```bash
yum install maven -y
```



#### 安装java

```bash
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm

rpm -ivh jdk-8u131-linux-x64.rpm
```



#### 打包java

```bash
# cd 进入/opt/RuoYi-Vuez
# 在有pom.xml这个目录下执行打包命令
cd /opt/RuoYi-Vue

# 执行打包命令
mvn clean package

# 打包后的文件会生成在 target 目录下生成jar包
# 运行项目
java -jar ruoyi-admin.jar 

nohup java -jar ruoyi-admin.jar &
```



#### 安装node

```bash
# 安装 nvm  这个需要梯子魔法才行。没有梯子魔法直接下载安装包安装可以去官网
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
# 加载 nvm
source ~/.bashrc
# 验证 nvm 安装
nvm --version
# 安装最新的 LTS（长期支持）版本：
nvm install --lts
# 安装特定版本的 Node.js（例如 18.x）
nvm install 16.14.0

node —v
```



#### 解决node版本低问题（没解决可以忽略）

```bash
node: /lib64/libm.so.6: version `GLIBC_2.27' not found (required by node)
node: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.20' not found (required by node)
node: /lib64/libstdc++.so.6: version `CXXABI_1.3.9' not found (required by node)
node: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by node)
node: /lib64/libc.so.6: version `GLIBC_2.28' not found (required by node)
node: /lib64/libc.so.6: version `GLIBC_2.25' not found (required by node)

```

报上边的错误

```bash
# 安装所需要的glibc-2.28
cd /opt
wget http://ftp.gnu.org/gnu/glibc/glibc-2.28.tar.gz
tar xf glibc-2.28.tar.gz 
cd glibc-2.28/ && mkdir build  && cd build
../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin

# 升级gcc与make
# 升级GCC(默认为4 升级为8)
yum install -y centos-release-scl
yum install -y devtoolset-8-gcc*
mv /usr/bin/gcc /usr/bin/gcc-4.8.5
ln -s /opt/rh/devtoolset-8/root/bin/gcc /usr/bin/gcc
mv /usr/bin/g++ /usr/bin/g++-4.8.5
ln -s /opt/rh/devtoolset-8/root/bin/g++ /usr/bin/g++

# 上边gcc安装报错就执行下边
# 配置CentOS-SCLo-scl.repo国内地址
vim /etc/yum.repos.d/CentOS-SCLo-scl.repo

[centos-sclo-sclo]
name=CentOS-7 - SCLo sclo
baseurl=https://mirrors.aliyun.com/centos/7/sclo/x86_64/sclo/
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo

# 配置CentOS-SCLo-scl-rh.repo
vim /etc/yum.repos.d/CentOS-SCLo-scl-rh.repo

[centos-sclo-rh]
name=CentOS-7 - SCLo rh
baseurl=https://mirrors.aliyun.com/centos/7/sclo/x86_64/rh/
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo


# 升级 make(默认为3 升级为4)
cd /opt
wget http://ftp.gnu.org/gnu/make/make-4.3.tar.gz
tar -xzvf make-4.3.tar.gz && cd make-4.3/
./configure  --prefix=/usr/local/make
make && make install
cd /usr/bin/ && mv make make.bak
ln -sv /usr/local/make/bin/make /usr/bin/make


cd /opt/glibc-2.28/build

../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin

# 还报错的话

yum install bison -y

make && make install
```





#### 安装前端

```bash
# 进入前端目录
cd /opt/RuoYi-Vue/ruoyi-ui

# 安装依赖
npm install

# 太慢了就配置加速源
npm install --registry=http://registry.npmmirror.com

# 启动前端
npm run dev

```

