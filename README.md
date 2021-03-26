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

workspace 容器提供了 node、npm、yarn 等命令，容器默认以 root 执行，提供了 www 和 自定义（NOTROOT——USER）用户和组，可通过 su www 切换。

TODO：未来会把 composer 从 php 容器中移至 workspace。 

### php

- 7.2 稳定版
- 7.4 测试中

进入 php-fpm 容器

```shell
docker exec -it docker-compose_php-fpm_1 /bin/bash
```

php.ini 配置文件位于 /usr/local/etc/php/php.ini

修改配置时，建议将其复制到容器外对应的目录进行修改，修改完毕将其再复制回容器。

docker to host，在 docker-compose/php 目录下执行：

```shell
docker cp docker-compose_php-fpm_1:/usr/local/etc/php/php.ini php.ini
```

host to docker，在 docker-compose/php 目录下执行：

```shell
docker cp php.ini docker-compose_php-fpm_1:/usr/local/etc/php/php.ini
```

#### php扩展

以 phpredis 为例，若要安装则在环境变量 .env 文件中，将 `INSTALL_EXT_PHPREDIS` 设置为 true 即可。

安装完毕后会将 redis.so 文件下载至 `/usr/local/lib/php/extensions/no-debug-non-zts-20170718`，
并在容器 `/usr/local/etc/php/conf.d` 目录下生成 `docker-php-ext-redis.ini`，自动引入扩展。

若安装后，使用过程中若需要关闭 phpredis ，则需要将 `docker-php-ext-redis.ini` 中的扩展引入注释掉即可。

```
;extension=redis
```

### redis

```shell
docker exec -it docker-compose_redis-db_1 redis-cli
```

### phpmyadmin

#### 访问

- 内部访问 localhost:8080
- 外部访问 phpmyadmin.docker.com （需绑定hosts）

登录时，若要连接本环境下的 mysql 容器，则`服务器`选项填写为 mysql 的服务名，即为 `mysql-db`。

#### 修改配置

phpmyadmin 容器中自己维护了一个 php 。在使用时，可能会遇到要求改 php.ini 的需求（如：phpmyadmin 根据 php.ini 默认限制了文件上传大小）。
phpmyadmin 容器中，在 `/usr/local/etc/php` 目录下存在 `php.ini-development` 、`php.ini-production` 。
若为开发环境，在宿主机上使用 `docker ps`，将 `php.ini-development` 从容器中复制到宿主机，然后进行编辑，编辑完成后，在复制回容器中。
**在宿主机中，复制 `php.ini-development` 并重命名为 `php.ini`，重启容器生效**。

PS:建议将 `php.ini-development` 复制到 phpmyadmin/conf 下进行修改，此目录会被忽略，不会被 git 记录。 

#### 三种连接方式

支持三种方式

- 连接任意数据库服务器
- 连接单个数据库服务器
- 连接多个数据库服务器（可切换）

连接任意数据库服务器， 将环境变量 `PMA_ARBITRARY` 设置为 1；

连接单个数据库服务器， 将环境变量 `PMA_ARBITRARY` 设置为 0，配置 PMA_HOST、PMA_VERBOSE（数据库服务器别名）、PMA_PORT；

连接多个数据库服务器， 将环境变量 `PMA_ARBITRARY` 设置为 0，PMA_HOST、PMA_VERBOSE、PMA_PORT 需要为空，否则不生效，配置 PMA_HOSTS、PMA_VERBOSES、PMA_PORTS。
