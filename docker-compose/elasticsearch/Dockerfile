ARG ELK_VERSION
# version @link https://www.docker.elastic.co/r/elasticsearch?limit=50&offset=0&show_snapshots=false
FROM docker.elastic.co/elasticsearch/elasticsearch:${ELK_VERSION}

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN yum install -y unzip wget;

# 安装ik插件
ARG ELK_VERSION
RUN mkdir -p ./plugins/ik \
    && wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v${ELK_VERSION}/elasticsearch-analysis-ik-${ELK_VERSION}.zip --no-check-certificate \
    && unzip -o elasticsearch-analysis-ik-${ELK_VERSION}.zip -d ./plugins/ik

# EXPOSE 9200 9300
