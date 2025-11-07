FROM alpine:3.21 AS builder

ENV VERSION=2.2.0

WORKDIR /usr/local/src

RUN set -x \
    && apk add --no-cache git build-base autoconf automake libevent-dev pkgconf zlib-dev openssl-dev pcre-dev \
    && git clone -b ${VERSION} --single-branch --depth=1 https://github.com/RedisLabs/memtier_benchmark.git . \
    && autoreconf -ivf \
    && ./configure \
    && make -j $(nproc) \
    && strip memtier_benchmark

FROM alpine:3.21

RUN set -x \
    && apk add --no-cache libevent libstdc++

COPY --from=builder /usr/local/src/memtier_benchmark /usr/local/bin/
COPY --chown=root:root --chmod=755 docker-entrypoint.sh /usr/local/bin/

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["memtier_benchmark"]