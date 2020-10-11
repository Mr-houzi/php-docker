ARG PHP_VERSION

FROM php:${PHP_VERSION}-fpm
MAINTAINER mr-houzi "ghosthouzi@foxmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 使用 阿里源 替换
RUN sed -i "s@http://deb.debian.org@http://mirrors.aliyun.com@g" /etc/apt/sources.list && rm -Rf /var/lib/apt/lists/* &&  cat /etc/apt/sources.list

# 安装扩展（php扩展依赖的系统扩展）
RUN apt-get update --fix-missing && \
    apt-get install -y --no-install-recommends \
        wget \
        rsyslog \
        libjpeg-dev libpng-dev libwebp-dev \
        libfreetype6-dev \
        libzip-dev \
        --fix-missing

# 安装PHP扩展
RUN docker-php-ext-configure gd --with-webp=/usr/include/ --with-freetype=/usr/include/ --with-jpeg=/usr/include/ \
  && docker-php-ext-install -j$(nproc) gd \
  && docker-php-ext-install pdo_mysql \
  && docker-php-ext-install bcmath \
  && docker-php-ext-install sockets \
  && docker-php-ext-install zip \
  && docker-php-ext-install pcntl \
  && docker-php-ext-install sysvmsg

# 安装 Composer
# 1. 下载安装脚本 － composer-setup.php － 到当前目录。
# 2. 执行安装过程。
# 3. 删除安装脚本。
# 4. 移动命令
# 5. 切换阿里源
ENV COMPOSER_HOME /root/Composer
RUN php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');" && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');" && \
    mv composer.phar /usr/local/bin/composer && \
    /usr/local/bin/composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

# WORKDIR指令用于指定容器的一个目录
# 容器启动时执行的命令会在该目录下执行
WORKDIR /data/www

# 容器内创建 www 用户和组
ARG RUNTIME_USER
RUN useradd -U ${RUNTIME_USER}

# 指定运行时用户，默认为root，建议设置为普通用户
USER ${RUNTIME_USER}
