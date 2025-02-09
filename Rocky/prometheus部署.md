# Prometheus部署

## 安装配置

```bash
# 软件包安装
wget https://github.com/prometheus/prometheus/releases/download/v2.53.3/prometheus-2.53.3.linux-amd64.tar.gz

useradd -M -s /user/sbin/nologin prometheus
groupadd prometheus
chown -R prometheus:prometheus /opt/prometheus

# 写入配置文件
vim /etc/systemd/system/prometheus.service

[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
Restart=on-failure

ExecStart=/opt/prometheus/prometheus \
  --config.file=/opt/prometheus/prometheus.yml \
  --storage.tsdb.path=/opt/prometheus/data \
  --storage.tsdb.retention.time=60d \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target

# 重新加载
systemctl daemon-reload

# 启动
systemctl start prometheus

netstat -tunlp

# 防火墙别忘了关
iptables -F
```



## 添加exporter

```bash
# node_exporter
wget https://github.com/prometheus/node_exporter


```

