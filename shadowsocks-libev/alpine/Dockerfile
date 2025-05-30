FROM alpine:3.19
LABEL maintainer="WavE Docker Maintainers <waveworkshop@outlook.com>"

ENV SERVER_ADDR=0.0.0.0
ENV SERVER_PORT=8388
ENV PASSWORD=
ENV METHOD=aes-256-gcm
ENV TIMEOUT=300
ENV DNS_ADDRS="8.8.8.8,8.8.4.4"
ENV TZ=UTC
ENV ARGS=

RUN set -x \
    && apk add --no-cache --virtual .build-deps \
        git \
        autoconf \
        automake \
        build-base \
        c-ares-dev \
        libcap \
        libev-dev \
        libtool \
        libsodium-dev \
        linux-headers \
        mbedtls-dev \
        pcre-dev \
    && mkdir /src \
    && cd /src \
    && git clone --recursive https://github.com/shadowsocks/shadowsocks-libev.git \
    && cd shadowsocks-libev \
    && ./autogen.sh \
    && ./configure --prefix=/usr/local --disable-documentation \
    && make -j$(getconf _NPROCESSORS_ONLN) \
    && make install \
    && cd /usr/local/bin \
    && ls /usr/local/bin/ss-* | xargs -n1 setcap cap_net_bind_service+ep \
    && strip $(ls /usr/local/bin | grep -Ev 'ss-nat') \
    && apk del --no-cache .build-deps \
    && apk add --no-cache \
        ca-certificates \
        iproute2-ss \
        $(scanelf --needed --nobanner /usr/local/bin/ss-* \
        | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
        | sort -u) \
    && rm -rf /src

COPY docker-entrypoint.sh /usr/local/bin/

ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 8388

STOPSIGNAL SIGINT

CMD ["ss-server"]
