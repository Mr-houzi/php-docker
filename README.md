# 基于 Docker 构建的 PHP 环境

## 目录结构

```
php-docker
├── data # 服务的数据目录
├── docker-compose # 构建 docker-compose 的目录
│   ├── docker-compose.yml
│   ├── env-example
│   ├── mysql
│   │   └── Dockerfile
│   ├── nginx
│   │   ├── conf.d
│   │   ├── Dockerfile
│   │   └── nginx.conf
│   ├── php
│   │   ├── Dockerfile
│   │   ├── php-dev.ini
│   │   ├── php-fpm.conf
│   │   └── php.ini
│   └── redis
│       └── Dockerfile
├── logs # 日志目录
├── README.md

docker-www # PHP项目存放目录
```

代码目录：在 `php-docker`目录同级上创建 `docker-www`作为项目代码目录

## 使用

### 配置

1. 复制 env-example 文件重命名为 .env，并填写所需配置。

2. 连接 mysql 时，host 应为 mysql-db，而不是 localhost 或其他。因为这是 docker-compose.yml 中定义 mysql 服务名。

### 运行

docker-compose 须在 docker-build/docker-compose 目录下执行

启动

    docker-compose up
    
停止 

    docker-compose stop

停止并移除

    docker-compose down
    
### workspace

workspace 容器提供了 node、npm 等命令，容器默认以 root 执行，提供了 www 和 自定义（NOTROOT——USER）用户和组，可通过 su www 切换。

TODO：未来会把 composer 从 php 容器中移至 workspace。 
