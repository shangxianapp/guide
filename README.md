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
| `NGINX_VHOST_DIR` | `/home/xiaowu/local/nginx-vhost` | Nginx 虚拟主机目录，使用 Docker Nginx 映射到该目录 |
| `USER` | `xiaowu` | 当前用户 |
| `USER_GROUP` | `work` | 当前用户组 |

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
