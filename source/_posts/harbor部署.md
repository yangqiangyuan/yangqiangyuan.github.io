---
title: harbor部署
date: 2022-11-25 10:04:56
tags: harbor部署
---

# harbor部署

<!--more-->

```

harbor部署

# 安装docker
# yum install -y vim wget net-tools gcc gcc-c++ socat conntrack nfs-utils rpcbind yum-utils device-mapper-persistent-data lvm2 docker-ce

# 配置私有仓库地址
# mkdir -p /etc/docker/
# vi /etc/docker/daemon.json
{
        "exec-opts": ["native.cgroupdriver=systemd"],
        "log-driver": "json-file",
        "log-opts": {
                "max-size": "100m"
        },
        "storage-driver": "overlay2",
        "storage-opts": [
                "overlay2.override_kernel_check=true"
        ],
        "insecure-registries": ["192.168.0.183:8080","192.168.0.183:5000"]
}

# 启动docker
# systemctl daemon-reload
# systemctl restart docker
# systemctl status docker
# systemctl enable docker

# 下载docker-compose
# wget -O /usr/local/bin/docker-compose https://github.com/docker/compose/releases/download/1.29.0/docker-compose-Linux-x86_64 
# mv /usr/local/bin/docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
# chmod a+x /usr/local/bin/docker-compose
# docker-compose --version

# 上传harbor-offline-installer-v2.6.2.tgz registry.tar
# tar -xf /usr/local/harbor-offline-installer-v2.6.2.tgz -C /usr/local/  

# 新建数据与日志目录
# mkdir -p /data1/harbor/{logs,data}

# 修改配置文件
# cd /usr/local/harbor
# vi harbor.yml
hostname: 192.168.0.183

http:
  port: 8080

harbor_admin_password: password

database:
  password: password
  max_idle_conns: 500
  max_open_conns: 10000

data_volume: /data1/harbor/data

trivy:
  ignore_unfixed: false
  skip_update: false
  insecure: false

jobservice:
  max_job_workers: 100

notification:
  webhook_job_max_retry: 100

chart:
  absolute_url: disabled

log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /data1/harbor/logs
_version: 2.2.0
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy


# docker load -i harbor.v2.6.2.tar.gz
# ./prepare
# ./install.sh --with-chartmuseum
# netstat -tunlp|grep 8080

# 登录harbor
# docker login 192.168.0.183:8080 -u admin -p password

# 浏览器登录，然后创建项目
# http://192.168.0.183:8080/  admin password


```

