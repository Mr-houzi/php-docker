FROM ubuntu:latest

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 使用 阿里源 替换
RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && apt-get update --fix-missing -y && apt-get upgrade -y

# 安装cron
RUN apt-get -y install cron \
    curl wget \
    --fix-missing \
    && apt-get update

# node npm yarn
ARG NODE_VERSION
RUN curl -sL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash  - \
    # yarn config
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    # ==========
    && apt-get update \
    && apt-get install -y nodejs yarn \
    # npm 更换源
    && npm config set registry http://registry.npm.taobao.org \
    # yarn 更换源
    && yarn config set registry https://registry.npm.taobao.org

# Copy hello-cron file to the cron.d directory
COPY cron /etc/cron.d/cron

# Give execution rights on the cron job
RUN chmod 0777 /etc/cron.d/cron

# Apply cron job
RUN crontab /etc/cron.d/cron

# Create the log file to be able to run tail
RUN touch /var/log/cron.log

# Run the command on container startup
#CMD cron && tail -f /var/log/cron.log

CMD ["cron", "-f"]

# WORKDIR指令用于指定容器的一个目录
# 容器启动时执行的命令会在该目录下执行
WORKDIR /data/www

# 容器内创建 www、非root 用户和组
ARG NOTROOT_USER
RUN useradd -U ${NOTROOT_USER} && \
    useradd -U www

USER root
# 指定运行时用户
