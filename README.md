# guide
Introduction to shangxian.app

## Service

- Docker - 容器化
- Jenkins - 构建、编译工具
- Nginx - Web Server
- Node.js - 应用
- Kibana - 日志可视化
- Shadowsocks - v.p.n
- MTProxy - Telegram Proxy

## Env

### common

| 变量名称 | 变量值 | 说明 |
| --- | --- | --- |
| `DOCKER_NGINX_CONTAINER_NAME` | `nginx` | Docker Nginx 容器名称 |
| `DOCKER_PRIVATE_NETWORK_NAME` | `network-private-01` | Docker 私有网络名称，用于链接多个容器 |
| `DOCKER_NGINX_VHOST_DIR` | `/home/xiaowu/local/nginx-vhost` | Nginx 虚拟主机目录，使用 Docker Nginx 映射到该目录 |
| `WWWROOT_DIR` | `/home/xiaowu/wwwroot` | 网站根目录，以域名存放子目录 |
| `LOCAL_DIR` | `/home/xiaowu/local` | 应用本地目录 |
| `USER` | `xiaowu` | 当前用户 |

### bj01

| 变量名称 | 变量值 | 说明 |
| --- | --- | --- |
| `HOSTNAME` | `bj01` | 主机名 |

### sg01

| 变量名称 | 变量值 | 说明 |
| --- | --- | --- |
| `HOSTNAME` | `sg01` | 主机名 |

### sg02

| 变量名称 | 变量值 | 说明 |
| --- | --- | --- |
| `HOSTNAME` | `sg02` | 主机名 |
| `DOCKER_JENKINS_CONTAINER_NAME` | `Jenkins` | Docker Jenkins 容器名称 |
| `DOCKER_JENKINS_DATA_DIR` | `/home/xiaowu/local/jenkins` | Jenkins 数据 |

## Bash

```bash
# root
# 更新源
yum update -y

# 安装依赖
yum install -y \
    vim \
    sudo \
    wget \
    yum-utils \
    device-mapper-persistent-data \
    htop \
    lvm2 

# 安装 Docker
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce docker-compose

# 查看版本
docker version
docker-compose version

# 创建帐号
useradd xiaowu
passwd xiaowu
echo 'xiaowu    ALL=(ALL)       ALL' >> /etc/sudoers

# 添加 xiaowu 到 docker 分组，方便不用 sudo 执行 docker 命令
groupadd docker
gpasswd -a xiaowu docker

# 开机启动 Docker
systemctl enable docker

# 启动 Docker
systemctl start docker

# 切换为 xiaowu 用户
su xiaowu

# 创建所需目录
mkdir -p \
    ~/bin \
    ~/wwwroot \
    ~/local/nginx-vhost \
    ~/local/nginx-cache \
    ~/local/jenkins

# 创建环境变量
echo '# 环境变量配置
DOCKER_JENKINS_CONTAINER_NAME=jenkins
DOCKER_NGINX_CONTAINER_NAME=nginx
DOCKER_PRIVATE_NETWORK_NAME=network-private-01
DOCKER_NGINX_VHOST_DIR=$HOME/local/nginx-vhost
DOCKER_JENKINS_DATA_DIR=$HOME/local/jenkins
WWWROOT_DIR=$HOME/wwwroot
LOCAL_DIR=$HOME/local
PATH=$PATH:$HOME/bin' > ~/.env
echo '[ -s "$HOME/.env" ] && \. "$HOME/.env"' >> ~/.bashrc
source ~/.bashrc

# 创建私有网络
docker network create "${DOCKER_PRIVATE_NETWORK_NAME}"

# 运行个 Demo 查看
docker run hello-world

# 运行一个 Nginx 示例查看
docker run -d \
    --name nginx \
    -p 80:80 \
    nginx
```
