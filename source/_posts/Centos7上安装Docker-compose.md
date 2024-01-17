---
title: Centos7上安装Docker-compose
date: 2023-12-26 17:10:52
tags: docker
---

# 在 CentOS 7 上安装 Docker Compose 的步骤如下

<!--more-->

1. 确保已经安装了 Docker。如果还没有安装 Docker，请先按照上述步骤安装 Docker。

2. 下载 Docker Compose 的可执行文件：
   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

3. 添加执行权限：
   ```bash
   sudo chmod +x /usr/local/bin/docker-compose
   ```

4. 创建一个软链接，使得 docker-compose 命令可以直接在终端中使用：
   ```bash
   sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
   ```

5. 验证安装结果：
   ```bash
   docker-compose --version
   ```

以上步骤中，第 2 步是通过 curl 命令下载 Docker Compose 的可执行文件。请注意，这里的命令会下载最新版本的 Docker Compose，如果你想要安装特定版本，可以将 URL 中的 `latest` 替换为相应的版本号。第 3 步是为 Docker Compose 添加执行权限。第 4 步是创建软链接，使得 docker-compose 命令可以直接在终端中使用。最后一步是验证安装结果，确保 Docker Compose 成功安装。

完成上述步骤后，就成功在 CentOS 7 上安装了 Docker Compose。你可以使用 `docker-compose` 命令来管理和部署多个 Docker 容器。
