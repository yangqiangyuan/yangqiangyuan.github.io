---
title: registry部署
date: 2022-11-25 09:59:52
tags: registry部署
---

# registry部署

背景：使用registry作为部署kubesphere所需要镜像的镜像仓库。

<!--more-->

```

registry部署
背景：使用registry作为部署kubesphere所需要镜像的镜像仓库。



# 创建docker数据目录
# mkdir -p /data1/docker
# ln -s /data1/docker /var/lib/docker

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

# 上传registry.tar到/usr/local/目录

# 新建镜像仓库目录
# mkdir -p /data1/registry

# 导入镜像
# docker load -i /usr/local/registry.tar

# 创建registry容器
# docker run -d --name  training-dev-registry --restart=always -v /data1/registry:/var/lib/registry -p 5000:5000 registry:2

# 验证镜像仓库是否可用
# docker tag registry:2 192.168.0.183:5000/official/registry:2
# docker push 192.168.0.183:5000/official/registry:2


```

