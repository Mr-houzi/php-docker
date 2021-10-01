FROM golang

# 开启 go module ，go 1.13之后默认开启
# 替换为国内七牛源
RUN go env -w GO111MODULE=on \
    && go env -w GOPROXY=https://goproxy.cn,direct

WORKDIR /data/www

EXPOSE 8088
