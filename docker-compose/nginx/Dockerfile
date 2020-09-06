FROM nginx:1.18

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 容器内创建 www 用户和组
RUN useradd -U www
