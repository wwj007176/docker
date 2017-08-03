# 获取基础镜像
FROM wwj/base
# 维护者
MAINTAINER wwj (wwj_007176@qq.com)
# 安装JDK 
# 安装依赖包
RUN apt-get update && apt-get install -y --no-install-recommends \
		bzip2 \
		unzip \
		xz-utils \
	&& rm -rf /var/lib/apt/lists/*
ENV LANG C.UTF-8

# 添加源文件
ENV JAVA_VERSION 1.7.0_45
ADD jdk1.7.0_45.tar.gz /usr/local/

# 设置环境变量        
ENV JAVA_HOME /usr/local/jdk1.7.0_45
ENV PATH $PATH:$JAVA_HOME/bin:
ENV CLASSPATH .:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
RUN set -ex; \
        \
        java -version; 
