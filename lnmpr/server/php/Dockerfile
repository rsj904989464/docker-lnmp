FROM php:7.3.33-fpm-alpine

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
    sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

COPY ./tgz /root


#pecl 依赖 autoconf gcc g++ make
RUN apk update && \
    apk add --no-cache libstdc++ freetype libpng libjpeg-turbo freetype-dev libpng-dev jpeg-dev libjpeg libjpeg-turbo-dev libwebp libwebp-dev gmp-dev icu-dev gettext gettext-dev imagemagick imagemagick-dev libzip libzip-dev git vim wget net-tools supervisor && \
    apk add --no-cache --virtual .build-deps $PHPIZE_DEPS openssl-dev pcre-dev pcre2-dev zlib-dev && \
               docker-php-ext-configure gd --with-gd --with-webp-dir --with-jpeg-dir --with-png-dir --with-zlib-dir --with-freetype-dir && \
               NUMPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) && \
               docker-php-ext-install -j${NUMPROC} gd sockets opcache pdo_mysql pcntl mysqli gmp zip


#源码包安装方式
#RUN wget http://pecl.php.net/get/msgpack-2.0.3.tgz -O msgpack-2.0.3.tgz \
#	&& mkdir -p msgpack \
#	&& tar -xf msgpack-2.0.3.tgz -C msgpack --strip-components=1 \
#	&& ( \
#        cd msgpack \
#        && phpize \
#        && ./configure \
#        && make \
#        && make install \
#    ) \
#	&& rm -r msgpack \
#    && docker-php-ext-enable msgpack


#swoole扩展
RUN docker-php-source extract && \
    mkdir /usr/src/php/ext/swoole && \
    #github下载网络问题
#    curl -sfL https://github.com/swoole/swoole-src/archive/v4.8.13.tar.gz -o swoole.tar.gz && \
    tar xfz /root/swoole.tar.gz --strip-components=1 -C /usr/src/php/ext/swoole && \
    docker-php-ext-configure swoole --enable-http2 --enable-mysqlnd --enable-openssl --enable-sockets --enable-swoole-json && \
    docker-php-ext-install -j$(nproc) swoole && \
    rm -f /root/swoole.tar.gz $HOME/.composer/*-old.phar && \
    docker-php-source delete && \
    apk del .build-deps

#redis扩展
RUN docker-php-source extract && \
    mkdir /usr/src/php/ext/redis && \
    tar xfz /root/redis.tgz --strip-components=1 -C /usr/src/php/ext/redis && \
    docker-php-ext-configure redis && \
    docker-php-ext-install -j$(nproc) redis && \
    rm -f /root/redis.tgz $HOME/.composer/*-old.phar && \
    docker-php-source delete

