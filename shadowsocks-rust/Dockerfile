FROM --platform=$BUILDPLATFORM rust:1.61.0-alpine AS build

ARG TARGETARCH

RUN set -x \
    && apk add --no-cache build-base git 

WORKDIR /root/

RUN cd /root/ \
    && git clone https://github.com/shadowsocks/shadowsocks-rust.git \
    && case "$TARGETARCH" in \
    "386") \
        RUST_TARGET="i686-unknown-linux-musl" \
        MUSL="i686-linux-musl" \
        ;; \
    "amd64") \
        RUST_TARGET="x86_64-unknown-linux-musl" \
        MUSL="x86_64-linux-musl" \
        ;; \
    "arm64") \
        RUST_TARGET="aarch64-unknown-linux-musl" \
        MUSL="aarch64-linux-musl" \
        ;; \
    *) \
        echo "Doesn't support $TARGETARCH architecture" \
        exit 1 \
        ;; \
    esac \
    && wget -qO- "https://musl.cc/$MUSL-cross.tgz" | tar -xzC /root/ \
    && CC=/root/$MUSL-cross/bin/$MUSL-gcc \
    && rustup target add $RUST_TARGET \
    && RUSTFLAGS="-C linker=$CC" \
    && CC=$CC \
    && cargo build --target "$RUST_TARGET" --release --features "local-tun local-redir armv8 neon stream-cipher aead-cipher-2022" \
    && mv target/$RUST_TARGET/release/ss* target/release/ 

FROM alpine:3.16 AS ssserver

COPY --from=build /root/shadowsocks-rust/target/release/sslocal /usr/bin
COPY --from=build /root/shadowsocks-rust/target/release/ssserver /usr/bin
COPY --from=build /root/shadowsocks-rust/examples/config.json /etc/shadowsocks-rust/

COPY docker-entrypoint.sh /usr/local/bin/

ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["ssserver"]