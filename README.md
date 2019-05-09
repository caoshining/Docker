### Docker简介
- Docker 是使用  Go 语言 进行开发实现，基于 Linux 内核的 cgroup，namespace，以及 AUFS 类的 Union FS 等技术，对进程进行封装隔离，属于 操作系统层面的虚拟化技术。
- 隔离的进程独立于宿主和其它的隔离的进程。
- Docker 在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等
- 优点是没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

#### Docker的优势
- 高效的利用系统资源
- 快速的启动时间
- 一致的运行环境
- 持续交付和部署：对开发和运维（DevOps）人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。
使用 Docker 可以通过定制应用镜像来实现持续集成、持续交付、部署。开发人员可以通过 Dockerfile 来进行镜像构建，并结合 持续集成(Continuous Integration) 系统进行集成测试，而运维人员则可以直接在生产环境中快速部署该镜像，甚至结合 持续部署(Continuous Delivery/Deployment) 系统进行自动部署。
- Docker 确保了执行环境的一致性，使得应用的迁移更加容易
- Docker 使用的分层存储以及镜像的技术，使得应用重复部分的复用更为容易，也使得应用的维护更新更加简单，基于基础镜像进一步扩展镜像也变得非常简单。社区健壮，一堆开源项目团队维护一批高质量的官方景象。
----
#####对比图
| 特性 | 容器 | 虚拟机  |
| --- |--- |---|
| 启动  | 秒级  | 分钟级  |
| 硬盘使用 | MB | GB
| 性能 | 接近原生 | 较弱  |
| 系统支持量 |  单机支持上千个容器  | 一般几十个|
---- 

###相关基本概念
Docker 包括三个基本概念
- 镜像（Image）
- 容器（Container）
  - 容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
  - 容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。
- 仓库（Repository）
  - 一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。

三个概念包括Docker的生命周期

### 安装(CentOS)
- 卸载旧版本
``` linux
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```
- 使用 yum 安装
```
$sudo yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2
```
- 国内网络问题，强烈建议使用国内源
``` 
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
# 官方源
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
- 安装 Docker CE
```
sudo yum makecache fast
sudo yum install docker-ce
```
-  使用脚本自动安装
```
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```
- 启动 Docker CE
```
$ sudo systemctl enable docker
$ sudo systemctl start docker
```
- 建立 docker 用户组
  - 建立 `docker` 组：
  ```
  sudo groupadd docker
  ```
  - 将当前用户加入 `docker` 组：
  ```
  sudo usermod -aG docker $USER
  ```
-  测试Docker是否安装正确
```
$ docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
d1725b59e92d: Pull complete
Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
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
-  镜像加速器
   -  Ubuntu 14.04、Debian 7 Wheezy 配置
  ```
  DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com"
  ```
- 重新启动服务。
```
$ sudo service docker restart
```
- Ubuntu 16.04+、Debian 8+、CentOS 7 在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）
``` json
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}
```
- 重新启动服务。
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

- 添加内核参数
  - CentOS 使用 Docker CE 看到下面的这些警告信息：
  ```
    WARNING: bridge-nf-call-iptables is disabled
    WARNING: bridge-nf-call-ip6tables is disabled
  ```
  解决方法：内核配置参数以启用这些功能。
  ```
    sudo tee -a /etc/sysctl.conf <<-EOF
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
  ```
  然后重新加载sysctl.conf
  ```
    sudo sysctl -p
  ```
