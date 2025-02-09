FROM ruby:3.1.6-alpine3.21 AS builder

WORKDIR /build

RUN apk add --no-cache build-base

RUN wget -O - https://github.com/jemalloc/jemalloc/releases/download/5.3.0/jemalloc-5.3.0.tar.bz2 | tar -xj \
    && cd jemalloc-5.3.0 \
    && ./configure \
    && make \
    && make install \
    && strip /usr/local/lib/libjemalloc.so.2

FROM ruby:3.1.6-alpine3.21

COPY --from=builder /usr/local/lib/libjemalloc.so.2 /usr/local/lib/
ENV LD_PRELOAD=/usr/local/lib/libjemalloc.so.2

# https://github.com/jemalloc/jemalloc/issues/1443