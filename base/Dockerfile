# 引用基础镜像
# FROM debian:jessie-20170606
FROM debian:jessie-slim
# 创建者
MAINTAINER wwj (wwj_007176@qq.com)

# Ubuntu的Docker官方镜像中是没有缓存Apt的软件包列表
# 因此在做其他任务基础软件的安装前，都需要至少先做一次apt-get update。
# 有时为了加快 apt-get安装软件的速度，还需要修改Apt源的列表文件/etc/apt/sources.list
ADD sources.list /etc/apt/sources.list
# 安装依赖包
# 安装依赖包参数说明
# -y 消除更新过程中所有向用户提问的部分，在这里还可以使用-qqy 隐藏Apt更新缓存过程中打印的日志
# --no-install-recommends 避免安装非必要的文件
# RUN apt-get update && apt-get install -y --no-install-recommends \
# 		ca-certificates \
# 		wget \
#                 curl \
#                 vim \
# 	&& rm -rf /var/lib/apt/lists/*
# grab gosu for easy step-down from root
ENV GOSU_VERSION 1.10
# 因为github releases下载太慢，估计是被墙了，所以提取把文件下好
ADD gosu-amd64 /usr/local/bin/gosu
ADD gosu-amd64.asc /usr/local/bin/gosu.asc

RUN set -x; \
        \
        buildDeps='ca-certificates wget curl vim'; \
        apt-get update; \
        apt-get install -y --no-install-recommends $buildDeps; \
        rm -rf /var/lib/apt/lists/*; \
        dpkgArch="$(dpkg --print-architecture | awk -F- '{print $NF }')"; \
        # echo "$dpkgArch";\
        # wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch";\
        # wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc";\
        export GNUPGHOME="$(mktemp -d)"; \
        gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4; \
        gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu; \
        rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc; \
        \
        chmod +x /usr/local/bin/gosu; \
        gosu nobody true; \
        \
        apt-get purge -y --auto-remove $buildDeps
