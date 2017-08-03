FROM wwj/jdk

ENV CATALINA_HOME /usr/local/tomcat8
ENV PATH $CATALINA_HOME/bin:$PATH
RUN mkdir -p $CATALINA_HOME
WORKDIR $CATALINA_HOME

# 安装 Tomcat Native
# Tomcat Native 是一个利用 APR 来提升Tomcat性能的本地API
# 单独配置 Tomcat Native 的环境变量
ENV TOMCAT_NATIVE_LIBDIR $CATALINA_HOME/native-jni-lib
ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH:+$LD_LIBRARY_PATH:}$TOMCAT_NATIVE_LIBDIR

# Tomcat Native 运行时的依赖关系
# Tomcat Native 1.2+ 需要 OpenSSL 较新的版本，在 debian:jessie 版本是可用的
# 检查 OpenSSL 的版本 是否大于 1.0.2
# 如果配置错误，您的OpenSSL版本与此版本的 Tomcat Native 不兼容 （eg：configure: error: Your version of OpenSSL is not compatible with this version of tcnative）
# 查看 http://tomcat.10.x6.nabble.com/VOTE-Release-Apache-Tomcat-8-0-32-tp5046007p5046024.html (and following discussion)
# 和 https://github.com/docker-library/tomcat/pull/31
ENV OPENSSL_VERSION 1.1.0f-3
ADD sources.list /etc/apt/sources.list.d/stretch.list

RUN { \
        echo 'Package: *'; \
        echo 'Pin: release n=stretch'; \
        echo 'Pin-Priority: -10'; \
        echo; \
        echo 'Package: openssl libssl*'; \
        echo "Pin: version $OPENSSL_VERSION"; \
        echo 'Pin-Priority: 990'; \   
    } > /etc/apt/preferences.d/stretch-openssl

RUN set -ex \
        \
        && buildDeps='wget' \
        && apt-get update && apt-get install -y --no-install-recommends \
                    ca-certificates \
                    wget \
                    libapr1 \
                    openssl="$OPENSSL_VERSION" \
            && rm -rf /var/lib/apt/lists/* 

# ENV GPG_KEYS 05AB33110949707C93A279E3D3EFE6B686867BA6 07E48665A34DCAFAE522E5E6266191C37C037D42 47309207D818FFD8DCD3F83F1931D684307A10A5 541FBE7D8F78B25E055DDEE13C370389288584E7 61B832AC2F1C5A90F0F9B00A1C506407564C17A3 713DA88BE50911535FE716F5208B0AB1D63011C7 79F7026C690BAA50B92CD8B66A3AD3F4F22C4FED 9BA44C2621385CB966EBA586F72C284D731FABEE A27677289986DB50844682F8ACB77FC2E86E29AC A9C5DF4D22E99998D9875A5110C01C5A2F6059E7 DCFD35E0BF8CA7344752DE8B6FB21E8933C60243 F3A04C595DB5B6A5F1ECA43E3B7BBB100D811BBE F7DA48BB64BCB84ECBA7EE6935CD23C10D498E23
# RUN set -ex; \
# 	for key in $GPG_KEYS; do \
# 		gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
# 	done

ENV TOMCAT_MAJOR 8
ENV TOMCAT_VERSION 8.0.44

ENV TOMCAT_TGZ_URL https://www.apache.org/dyn/closer.cgi?action=download&filename=tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz
# 并不是所有的镜像文件都带 asc文件
# 校验文件
ENV TOMCAT_ASC_URL https://www.apache.org/dist/tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz.asc

COPY apache-tomcat-8.0.44.tar.gz tomcat.tar.gz
RUN set -x \
        \
        # && wget -O tomcat.tar.gz "$TOMCAT_TGZ_URL" \
        # && wget -O tomcat.tar.gz.asc "$TOMCAT_ASC_URL" \
        # && gpg --batch --verify tomcat.tar.gz.asc tomcat.tar.gz \
        && tar -xvf tomcat.tar.gz --strip-components=1 \
        && rm bin/*.bat \
        && rm tomcat.tar.gz* \
        \
        && nativeBuildDir="$(mktemp -d)" \
        && tar -xvf bin/tomcat-native.tar.gz -C "$nativeBuildDir" --strip-components=1 \
        \
        && nativeBuildDeps=" \
            dpkg-dev \
            gcc \
            libapr1-dev \
            libssl-dev \
            make \
        " \
        && apt-get update && apt-get install -y --no-install-recommends $nativeBuildDeps && rm -rf /var/lib/apt/lists/* \
        && ( \
                export CATALINA_HOME="$PWD" \
                && export CATALINA_HOME \
                && cd "$nativeBuildDir/native" \
                && gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
                && ./configure \
                        --build="$gnuArch" \
                        --libdir="$TOMCAT_NATIVE_LIBDIR" \
                        --prefix="$CATALINA_HOME" \
                        --with-apr="$(which apr-1-config)" \
                        --with-java-home="$JAVA_HOME" \
                        --with-ssl=yes \
                && make -j "$(nproc)" \
                && make install \        
        ) \
        && apt-get purge -y --auto-remove $nativeBuildDeps $buildDeps \
	    && rm -rf "$nativeBuildDir" \
	    && rm bin/tomcat-native.tar.gz

RUN set -e \
	&& nativeLines="$(catalina.sh configtest 2>&1)" \
	&& nativeLines="$(echo "$nativeLines" | grep 'Apache Tomcat Native')" \
	&& nativeLines="$(echo "$nativeLines" | sort -u)" \
	&& if ! echo "$nativeLines" | grep 'INFO: Loaded APR based Apache Tomcat Native library' >&2; then \
		echo >&2 "$nativeLines"; \
		exit 1; \
	fi         

EXPOSE 8080
CMD ["catalina.sh", "run"]
