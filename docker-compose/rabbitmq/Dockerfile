FROM rabbitmq:3.9

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 安装图形管理界面插件 UI port 15672
# 默认用户和密码 guest
RUN rabbitmq-plugins enable rabbitmq_management

RUN echo management_agent.disable_metrics_collector = false > /etc/rabbitmq/conf.d/management_agent.disable_metrics_collector.conf