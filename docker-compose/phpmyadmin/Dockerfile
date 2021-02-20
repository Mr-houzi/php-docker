FROM phpmyadmin/phpmyadmin
MAINTAINER mr-houzi "ghosthouzi@foxmail.com"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
