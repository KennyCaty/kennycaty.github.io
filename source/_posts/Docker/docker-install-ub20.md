---
title: Ubuntu20安装docker（2023）
date: 2023-08-30 14:16:53
tags:Docker
---









看了很多安装docker并配置阿里云镜像的教程，许多都有问题，记录一下没问题的安装过程





# 卸载旧版本

Docker 的旧版本被称为 docker，docker.io 或 docker-engine 。如果已安装，请卸载它们：

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

当前称为 Docker Engine-Community 软件包 docker-ce 





# 设置仓库

在新主机上首次安装 Docker Engine-Community 之前，需要设置 Docker 仓库。之后，您可以从仓库安装和更新 Docker 。



## 安装依赖包

```bash
sudo apt update
```

```BASH
sudo apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common
```



## 添加GPG密钥

添加 Docker 的官方 GPG 密钥：

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

或者添加阿里云镜像证书

```bash
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```



## 添加软件源

清华大学TUNA软件仓库

```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```



阿里云的软件仓库

```bash
sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```



官方的软件仓库

```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
```







# 安装 DOCKER 引擎

要安装最新版本，请运行：

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```



# 启动服务

```bash
sudo service docker start
```



或者这样，当然这样需要使用 systemd 作为 init 系统

> 注意：如果使用WSL，默认不能使用systemd [使用 systemd 通过 WSL 管理 Linux 服务 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/systemd)
>
> [WSL2使用systemctl命令的方法，简单快捷 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/554640726)

```bash
sudo systemctl start docker
```





通过运行映像验证 Docker 引擎安装是否成功。

```bash
sudo docker run hello-world
```





需要等待一会

如果成功则输出

```
jc@JLab:/etc/apt$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
719385e32844: Pull complete
Digest: sha256:dcba6daec718f547568c562956fa47e1b03673dd010fe6ee58ca806767031d1c
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```



这几行说明第一次运行时没有找到镜像，则会从仓库中拉取镜像

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
```

查看一下下载的这个 hello-world镜像 使用 `docker images`

```bash
jc@JLab:~$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    9c7a54a9a43c   3 months ago   13.3kB
```





# 卸载

两步

```bash
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```



# 免SUDO运行

参考如下方法将用户添加到docke组

```bash
sudo usermod -aG docker [你的用户名]
```

重启docker

```bash
sudo service docker restart
```

或者这样重启

```bash
sudo systemctl restart docker
```



然后再重启一下终端 再运行一下hello world

```bash
docker run hello-world
```







# 阿里云镜像加速



阿里云-容器镜像加速服务-找到镜像加速器，根据教程使用

[容器镜像服务 (aliyun.com)](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)









