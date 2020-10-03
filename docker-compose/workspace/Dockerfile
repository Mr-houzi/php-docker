FROM ubuntu:latest
MAINTAINER docker@ekito.fr

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 使用 阿里源 替换
RUN sed -i s/archive.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list \
    && sed -i s/security.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list \
    && apt-get update && apt-get upgrade

# 安装cron
RUN apt-get update && apt-get -y install cron \
    curl wget \
    --fix-missing && \
    apt-get update

# NVM管理node、npm
ENV NVM_DIR /usr/local/nvm
ARG NODE_VERSION
RUN mkdir -p /usr/local/nvm && \
    curl -o- https://gitee.com/mirrors/nvm/raw/master/install.sh | bash && \
    . $NVM_DIR/nvm.sh && \
    nvm install $NODE_VERSION && \
    nvm alias default $NODE_VERSION && \
    nvm use default
#    && \
#    # 更换源
#    nvm node_mirror https://npm.taobao.org/mirrors/node/ && \
#    nvm npm_mirror https://npm.taobao.org/mirrors/npm/

ENV NODE_PATH $NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH      $NVM_DIR/v$NODE_VERSION/bin:$PATH

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