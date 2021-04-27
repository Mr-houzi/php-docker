# FROM elasticsearch:7.12.0
FROM docker.elastic.co/elasticsearch/elasticsearch:7.12.0

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

EXPOSE 9200 9300
