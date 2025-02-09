# docker使用

## 1.使用代理拉取镜像

```bash
mkdir -p /etc/systemd/system/docker.service.d
vim /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTPS_PROXY=http://192.168.1.13:7890/"

systemctl daemon-reload
systemctl restart docker
```



## 2.使用加速源

```bash
# 创建目录
sudo mkdir -p /etc/docker
# 写入配置文件
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
    	"https://docker.unsee.tech",
        "https://dockerpull.org",
        "docker.1panel.live",
        "https://dockerhub.icu"
    ]
}
EOF
# 重启docker服务
sudo systemctl daemon-reload && sudo systemctl restart docker
```

