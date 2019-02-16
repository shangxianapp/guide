# guide
Introduction to Shangxian APP

## Service

- Docker - 容器化
- Jenkins - 构建、编译工具
- Nginx - Web Server
- Node.js - 应用
- Kibana - 日志可视化
- Shadowsocks - v.p.n
- MTProxy - Telegram Proxy

## Domain

- `bj01.ddns.shangxian.app` - bj01 服务器映射
- `sg01.ddns.shangxian.app` - sg01 服务器映射
- `sg02.ddns.shangxian.app` - sg02 服务器映射
- <https://shangxian.app> - Shangxian APP
- <https://jenkins.shangxian.app> - Jenkins
- <https://netdata.shangxian.app> - Netdata 服务器监控
- <https://portainer.shangxian.app> - Docker Web 可视化管理平台，基于 [Portainer](https://github.com/portainer/portainer)

## 授权

为了避免 Jenkins Job Console Log 明文出现密钥，使用 Jenkins 凭据管理，且把密钥、Token 等也存放在凭证里，使用统一的命名规则，如：

- `shangxianapp-xiaowu-ssh-key` - 登录各个服务器 `@xiaowu` 帐号的密钥
- `shangxianapp-github-xuexb-ssh-key` - GitHub `@xuexb` 用户的密钥
- `shangxianapp-github-xuexb-access-token` - GitHub `@xuexb` 用户的 Access Token
- `shangxianapp-aliyun-xuexb-apikey`、`shangxianapp-aliyun-xuexb-secret` - 阿里云相关令牌
- `shangxianapp-namecom-xuexb-apikey`、`shangxianapp-namecom-xuexb-secret` - name.com 相关令牌
- `shangxianapp-shadowsocks-port`、`shangxianapp-shadowsocks-pwd` - SS 相关令牌

这些凭据可以直接在 Jenkins Job 或者 Jenkins Pipeline 脚本使用，并且会以 `****` 显示。

## Nginx Proxy

由于在服务器中肯定是有多个 Web Server 的，而使用 Docker 容器化后需要对外暴露端口，但由于端口只能占用一个（比如：80、443），那么就在每个服务器中专门启用一个 Nginx Proxy Docker 专门来反向代理各个虚拟主机（ Web Server ），约定好统一的目录，比如：

- `DOCKER_NGINX_VHOST_DIR` - 虚拟主机配置目录，以 `域名.conf` 存放配置文件
- `DOCKER_NGINX_CA_DIR` - SSL 证书存放目录，以 `域名` 为子目录存放
- `DOCKER_NGINX_DIR` - Nginx 运行目录，配置文件更新后可以运行 `/bin/sh ${DOCKER_NGINX_DIR}/reload.sh` 来刷新 Nginx Proxy Docker
- `WWWROOT_DIR` - 网站目录映射，以 `域名` 为子目录存放，可以在 Nginx 配置文件里使用，比如设置 `root` 目录

> 更多 Nginx Proxy Docker 请看：<https://github.com/shangxianapp/docker-sg02-nginx-proxy>

为了更好的在 Nginx 配置文件中读取变量，专门创建了支持 Lua、echo-nginx 的 Nginx:alpine 镜像：<https://github.com/shangxianapp/docker-nginx-alpine>

## SSL 证书同步

由于是需要多个服务器同步证书，启用了一个 Jenkins Job 来每三个月处理证书，由于证书有好几个，使用 Jenkins Pipeline 脚本生成，这样也可以直接使用 Jenkins 凭据密钥，再结合 Jenkins Pipeline parallel 并行生成证书，并同步到各个服务器，最后再重启 Nginx Proxy Docker 。

> 生成脚本见：<https://github.com/shangxianapp/docker-auto-ssl>

## Env

### common

| 变量名称 | 变量值 | 说明 |
| --- | --- | --- |
| `DOCKER_NGINX_CONTAINER_NAME` | `nginx` | Docker Nginx 容器名称 |
| `DOCKER_PRIVATE_NETWORK_NAME` | `network-private-01` | Docker 私有网络名称，用于链接多个容器 |
| `DOCKER_NGINX_DIR` | `/home/xiaowu/local/nginx` | Nginx Proxy 运行目录，可以进入该目录执行 `docker-compose up -d` 刷新启动 |
| `DOCKER_NGINX_VHOST_DIR` | `/home/xiaowu/local/nginx-vhost` | Nginx 虚拟主机目录，使用 Docker Nginx 映射到该目录 |
| `DOCKER_NGINX_CA_DIR` | `/home/xiaowu/local/nginx-ca` | Nginx SSL 证书目录 |
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
    git \
    vim \
    sudo \
    wget \
    yum-utils \
    device-mapper-persistent-data \
    htop \
    lvm2b

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

# 设置权限，让 Jenkins 中的用户可以直连 docker
chmod 777 /var/run/docker.sock

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
    ~/local/nginx \
    ~/local/nginx-vhost \
    ~/local/nginx-ca \
    ~/.ssh \
    ~/local/jenkins

# 配置 .ssh 权限
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# 创建环境变量
echo '# 环境变量配置
export DOCKER_JENKINS_CONTAINER_NAME=jenkins
export DOCKER_NGINX_CONTAINER_NAME=nginx
export DOCKER_PRIVATE_NETWORK_NAME=network-private-01
export DOCKER_NGINX_DIR=$HOME/local/nginx
export DOCKER_NGINX_VHOST_DIR=$HOME/local/nginx-vhost
export DOCKER_NGINX_CA_DIR=$HOME/local/nginx-ca
export DOCKER_JENKINS_DATA_DIR=$HOME/local/jenkins
export WWWROOT_DIR=$HOME/wwwroot
export LOCAL_DIR=$HOME/local
export PATH=$PATH:$HOME/bin' > ~/.env
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
