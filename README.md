# 基于 Docker 构建的 PHP 环境

## 目录结构

```
php-docker/
├── data
├── docker-compose
│   ├── docker-compose.yml
│   ├── env-example
│   ├── mysql
│   │   └── Dockerfile
│   ├── nginx
│   │   ├── conf.d
│   │   │   ├── default.conf
│   │   │   ├── demo.conf
│   │   │   └── phpmyadmin.conf
│   │   ├── Dockerfile
│   │   └── nginx.conf
│   ├── php
│   │   ├── conf
│   │   ├── Dockerfile72
│   │   ├── Dockerfile74
│   │   └── php-fpm.conf
│   ├── phpmyadmin
│   │   └── Dockerfile
│   ├── redis
│   │   └── Dockerfile
│   └── workspace
│       ├── cron
│       └── Dockerfile
├── LICENSE
├── logs
└── README.md
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

以非root 用户进入容器 

```shell
docker exec -u www-data  -it docker-compose_php-fpm_1 /bin/bash
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

#### 版本切换和多版本PHP并存容器

默认 fpm 容器为 docker-compose.yml 中名为 php-fpm 的服务容器。如果想要切换版本可以在 .env 中修改 PHP_VERSION 值。

如果想要并存的多个版本的PHP容器时，则可以在 docker-compose.yml 复制一份 php-fpm 的配置，并改动如下。

```
// docker-compose.yml
……
  php-fpm74:
    build:
      context: ./php
      dockerfile: Dockerfile74
      args:
        INSTALL_EXT_PHPREDIS: ${INSTALL_EXT_PHPREDIS}
        INSTALL_EXT_SWOOLE: ${INSTALL_EXT_SWOOLE}
    ports:
      - "9074:9000"
    links:
      - mysql-db:mysql-db
      - redis-db:redis-db
    volumes:
      - ../../docker-www:/data/www:rw
      - ./php/php-fpm.conf:/usr/local/etc/php-fpm.conf:ro
      - ../logs/php-fpm:/var/log/php-fpm:rw
    networks:
      - backend
    restart: always
    command: php-fpm
……
```

主要改动了几点：

1. 服务名不能重名，改为 php-fpm74；
2. dockerfile 的构建文件改为你想构建的PHP版本，如：Dockerfile74；
3. 端口号映射（宿主机端口：容器端口），php-fpm74 宿主机端口不能和 php-fpm 端口相同，改为 9074，php-fpm 镜像默认 9000 端口，所以容器端口不用改变。

如果有项目想要用 php7.4，那么 Nginx 代理到 php-fpm74 容器，端口仍为 9000。

```
location ~ [^/]\.php(/|$)
    {
        ……
        fastcgi_pass php-fpm74:9000;
        fastcgi_index index.php;
        ……
    }
```

### mysql

容器内配置目录 /etc/mysql

建议将配置文件（如 my.cnf），通过 docker cp 复制到宿主机的 mysql/conf.d 目录进行修改，改完再复制回去，重启容器即可。

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

### elk

配置 Kibana 连接 Elasticsearch。在 .env 中配置 ELASTICSEARCH_HOSTS ，此值使一个数组，用 es 的服务名称+端口号连接。配置如下：

```
ELASTICSEARCH_HOSTS=["http://elasticsearch-elk:9200"]
```

## License

本项目基于 [MIT license](https://opensource.org/licenses/MIT).
