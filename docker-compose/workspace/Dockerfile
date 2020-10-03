FROM ubuntu:latest
MAINTAINER docker@ekito.fr

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 使用 阿里源 替换
RUN sed -i s/archive.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list \
    && sed -i s/security.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list \
    && apt-get update && apt-get upgrade

#安装node、npm、cron
RUN apt-get update && apt-get -y install cron nodejs npm

RUN npm config set registry https://registry.npm.taobao.org && \
    npm install n -g

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