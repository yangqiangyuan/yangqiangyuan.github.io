---
title: Centos7上安装Docker
date: 2023-12-26 17:08:37
tags: docker
---

# 在 CentOS 7 上安装 Docker 的步骤如下

<!--more-->

1. 更新系统：
   ```bash
   sudo yum update
   ```

2. 安装需要的软件包：
   ```bash
   sudo yum install -y yum-utils device-mapper-persistent-data lvm2
   ```

3. 添加 Docker 的 YUM 软件源：
   ```bash
   sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   ```

4. 安装 Docker：
   ```bash
   sudo yum install docker-ce docker-ce-cli containerd.io
   ```

5. 启动 Docker 服务：
   ```bash
   sudo systemctl start docker
   ```

6. 设置 Docker 开机自启：
   ```bash
   sudo systemctl enable docker
   ```

7. 测试 Docker 是否安装成功：
   ```bash
   sudo docker run hello-world
   ```

以上步骤中，第 1 步是更新系统。第 2 步是安装必要的软件包，其中 `yum-utils` 提供了 `yum-config-manager` 工具，而 `device-mapper-persistent-data` 和 `lvm2` 则是 Docker 的依赖项。第 3 步是添加 Docker 的 YUM 软件源。第 4 步是安装 Docker。第 5 步是启动 Docker 服务。第 6 步是设置 Docker 开机自启。最后一步是测试 Docker 是否安装成功。

需要注意的是，在某些情况下，Docker 可能需要使用代理才能正常工作。如果你的网络环境需要使用代理，请在安装 Docker 之前配置相应的代理设置。
