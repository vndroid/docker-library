FROM php:8.2.8-fpm-alpine3.18

LABEL maintainer="WavE Container Maintainers <waveworkshop@outlook.com>"

ENV MOD_DATE 230714

RUN set -eux \
    && apk upgrade --no-cache \
    && apk add --no-cache linux-headers \
    && docker-php-ext-install bcmath calendar exif opcache mysqli pdo_mysql sockets \
    && apk add --no-cache libbz2 bzip2-dev \
    && docker-php-ext-install bz2 \
    && apk add --no-cache libxml2 libxml2-dev \
    && docker-php-ext-install xml \
    && apk add --no-cache freetype libavif libjpeg libpng libwebp libxpm \
    && apk add --no-cache libavif-dev libwebp-dev libjpeg-turbo-dev libpng-dev libxpm-dev freetype-dev \
    && docker-php-ext-configure gd --with-freetype --with-avif --with-webp --with-jpeg --with-xpm --enable-gd-jis-conv \
    && docker-php-ext-install gd \
    && apk add --no-cache gmp libgmpxx gmp-dev \
    && docker-php-ext-install gmp \
    && apk add --no-cache libpq libpq-dev \
    && docker-php-ext-install pgsql pdo_pgsql \
    && apk add --no-cache autoconf build-base \
    && pecl install -D 'enable-apcu-debug="no"' apcu \
    && docker-php-ext-enable apcu \
    && pecl install igbinary \
    && docker-php-ext-enable igbinary \
    && pecl install msgpack \
    && docker-php-ext-enable msgpack \
    && apk add --no-cache zstd-libs zstd-dev \
    && pecl install -D 'enable-redis-igbinary="yes" enable-redis-lzf="yes" enable-redis-zstd="yes"' redis \
    && docker-php-ext-enable redis \
    && apk add --no-cache libmemcached-libs \
        libmemcached-dev \
        libevent-dev \
        zlib-dev \
    && pecl install -D 'enable-memcached-igbinary="yes" enable-memcached-json="yes" enable-memcached-msgpack="yes"' memcached \
    && docker-php-ext-enable memcached \
    && apk add --no-cache libmcrypt libmcrypt-dev \
    && pecl install mcrypt \
    && apk add --no-cache snappy icu-libs \
    && apk add --no-cache icu-dev zstd-dev snappy-dev openssl-dev cyrus-sasl-dev \
    && docker-php-ext-enable mcrypt \
    && pecl install -D 'with-mongodb-ssl="openssl" with-mongodb-icu="yes" with-mongodb-sasl="cyrus"' mongodb \
    && docker-php-ext-enable mongodb \
    && apk add --no-cache curl-dev \
    && pecl install -D 'enable-sockets="yes" enable-openssl="yes" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="yes" enable-swoole-curl="yes"' swoole \
    && docker-php-ext-enable swoole \
    && apk add --no-cache yaml yaml-dev \
    && pecl install yaml \
    && docker-php-ext-enable yaml \
    ## apk add --update ca-certificates-bundle ca-certificates curl openssl linux-headers \
    #&& pecl install xdebug \
    #&& docker-php-ext-enable xdebug \
    && apk add --no-cache libzip libzip-dev \
    && pecl install zip \
    && docker-php-ext-enable zip \
    && strip /usr/local/lib/php/extensions/no-debug-non-zts-20*/*.so \
    && chmod +x /usr/local/lib/php/extensions/no-debug-non-zts-20*/*.so \
    && sed -i '/memcached/i\extension=msgpack' /usr/local/etc/php/conf.d/docker-php-ext-memcached.ini \
    && sed -i '/msgpack/d' /usr/local/etc/php/conf.d/docker-php-ext-msgpack.ini \
    && cp /usr/local/bin/docker-php-entrypoint /usr/local/bin/docker-entrypoint.sh \
    && apk del --no-cache autoconf build-base \
        bzip2-dev \
        curl-dev \
        cyrus-sasl-dev \
        freetype-dev \
        gmp-dev \
        icu-dev \
        libc-utils \
        libavif-dev \
        libevent-dev \
        libxml2-dev \
        libpng-dev \
        libmemcached-dev \
        libmcrypt-dev \
        libwebp-dev \
        libjpeg-turbo-dev \
        libpq-dev \
        libxpm-dev \
        libzip-dev \
        linux-headers \
        openssl-dev \
        snappy-dev \
        yaml-dev \
        zlib-dev \
        zstd-dev \
    && rm -rf /tmp/pear/

ENTRYPOINT ["docker-entrypoint.sh"]
WORKDIR /var/www/html

STOPSIGNAL SIGQUIT

EXPOSE 9000
CMD ["php-fpm"]