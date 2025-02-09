# rsync实战练习

## 服务端配置

脚本完成

```shell
#!/bin/bash 

# 1.安装rsync服务
yum install rsync -y


# 2.修改配置文件

cat > /etc/rsyncd.conf << 'EOF'
uid = www 
gid = www 
port = 873
fake super = yes
use chroot = no
max connections = 200
timeout = 600
ignore errors
read only = false
list = false
auth users = leo01
secrets file = /etc/leo_passwd
log file = /var/log/rsyncd.log
#####################################
[backup]
comment = leo backup lib
path = /backup

[data]
comment = this is secord backup dir,to website data..
path = /data
EOF


# 2.1 创建用户以及数据目录
useradd -u 1000 -M -s /sbin/nologin www

mkdir -p /backup  /data

chown -R www:www /backup
chown -R www:www /data

# touch /etc/leo_passwd
# chmod 600 /etc/leo_passwd

# 2.2 创建rsync服务模式专属账户密码

# echo 'leo01:leo666' > /etc/leo_passwd

cat > /etc/leo_passwd << EOF
leo01:leo666
EOF

chmod 600 /etc/leo_passwd


# 3.重新启动服务
systemctl start rsyncd


```



脚本权限添加

````shell
chmod u+x sever_rsync.sh
````







## 客户端配置



```shell
yum install rsync -y
```



```shell
# 1.密码配置文件
echo 'leo666'  > /etc/my_rsync.pwd

chmod 600 /etc/my_rsync.pwd

# 2.密码变量
export RSYNC_PASSWORD='leo666'


```



## 客户端需求配置

### 拆分步骤

```shell
# 文件名
主机名
$(hostname)
当前ip地址
$(ifconfig eth0 | awk  'NR==2 {print $2}')
当前时间
$(date '+%F')

拼起来

$(hostname)_$(ifconfig eth0 | awk  'NR==2 {print $2}')_$(date '+%F')

midir /backup/$(hostname)_$(ifconfig eth0 | awk  'NR==2 {print $2}')_$(date '+%F')


# 打包
cd / && tar -czf /backup/$(hostname)_$(ifconfig eth0 | awk  'NR==2 {print $2}')_$(date '+%F')/etc.tgz etc

cd / && tar -czf /backup/$(hostname)_$(ifconfig eth0 | awk  'NR==2 {print $2}')_$(date '+%F')log.tgz var/log

#md5校验
md5sum /backup/$(hostname)_$(ifconfig eth0 | awk  'NR==2 {print $2}')_$(date '+%F')/*.tgz > /backup/$(hostname)_$(ifconfig eth0 | awk  'NR==2 {print $2}')_$(date '+%F')/all_data_check.txt




# 发送
rsync -avzF /backup/  leo01@rsync-41::backup


#删除
find /backup -type f -mtime +7 -delete

```



### 整合脚本

client_bak_rsync.sh

```shell
#!/bin/bash

# 1. 安装服务
yum install rsync -y

# 主动在脚本中，定义path变量，防止命令无法执行
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

# 2. 创建打包目录及打包文件
mkdir -p /backup/$(hostname)_$(ifconfig eth0 | awk  'NR==2 {print $2}')_$(date '+%F')

cd / && tar -czf /backup/$(hostname)_$(ifconfig eth0 | awk  'NR==2 {print $2}')_$(date '+%F')/etc.tgz etc

cd / && tar -czf /backup/$(hostname)_$(ifconfig eth0 | awk  'NR==2 {print $2}')_$(date '+%F')/log.tgz var/log



# 2.1 md5校验文件
md5sum /backup/$(hostname)_$(ifconfig eth0 | awk  'NR==2 {print $2}')_$(date '+%F')/*.tgz > /backup/$(hostname)_$(ifconfig eth0 | awk  'NR==2 {print $2}')_$(date '+%F')/all_data_md5.txt



# 3. 密码配置，发送备份数据到rsync服务器
export RSYNC_PASSWORD=leo666
rsync -avzF /backup/  leo01@rsync-41::backup

# 4. 删除过期文件
find /backup -type f -mtime +7 -delete

```



### 测试脚本

```shell
bash -x client_bak_rsync.sh
```





## 服务端需求配置

```shell
bash -x sever_rsync.sh
```

### 邮件配置

```shell
#1.安装配置mailx：
yum install mailx -y

#2.邮箱配置文件，给你的自己的信息

cat > /etc/mail.rc << 'EOF' 
set from=2366993844@qq.com
set smtp=smtps://smtp.qq.com:465
set smtp-auth-user=2366993844@qq.com
set smtp-auth-password=cbdyiplucxxqeadd
set smtp-auth=login
set ssl-verify=ignore
set nss-config-dir=/etc/pki/nssdb/
EOF

```



### 整合脚本

sever_bak_rsync.sh

```shell
#!/bin/bash

# 1. 校验备份的数据
md5sum -c /backup/nfs-31_10.0.0.31_$(date "+%F")/all_data_md5.txt  > /backup/nfs-31_10.0.0.31_$(date "+%F")/check_md5_result.txt

# 2. 发邮件
mail -s "check-rsync-$(date +%F)" 2366993844@qq.com < /backup/nfs-31_10.0.0.31_$(date "+%F")/check_md5_result.txt

# 3. 删除旧资料
find /backup -type f -mtime +180 -delete
```







加定时任务

