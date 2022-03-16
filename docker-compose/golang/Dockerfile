FROM golang

ENV TZ=Asia/Shanghai

# 开启 go module ，go 1.13之后默认开启
# 替换为国内七牛源
RUN go env -w GO111MODULE=on \
    && go env -w GOPROXY=https://goproxy.cn,direct

RUN apt-get update --fix-missing && \
    apt-get install -y --no-install-recommends \
        wget \
        git \
        --fix-missing

WORKDIR /data/www

# debug 工具 2345 端口
RUN git clone https://github.com/go-delve/delve && \
    cd delve && go install github.com/go-delve/delve/cmd/dlv && \
    dlv version

EXPOSE 8088
