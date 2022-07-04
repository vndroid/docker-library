FROM rust:1.61.0-alpine AS builder

LABEL maintainer="WavE Docker Maintainers <waveworkshop@outlook.com>"

ENV COMMIT_ID="5916909ebd44d449d46165dcb95f4259c8c73084"

RUN set -x \
    && apk add --no-cache build-base git \
    && mkdir /src \
    && cd /src \
    && git clone https://github.com/shadowsocks/shadowsocks-rust.git \
    && apkArch="$(apk --print-arch)" \
    && case "$apkArch" in \
        "x86") \
            RUST_TARGET="i686-unknown-linux-musl" \
            MUSL="i686-linux-musl" \
        ;; \
        "x86_64") \
            RUST_TARGET="x86_64-unknown-linux-musl" \
            MUSL="x86_64-linux-musl" \
        ;; \
        "aarch64") \
            RUST_TARGET="aarch64-unknown-linux-musl" \
            MUSL="aarch64-linux-musl" \
        ;; \
        *) \
            echo "ERROR: Doesn't support $apkArch architecture!" \
            exit 1 \
        ;; \
    esac \
    && wget -qO- "https://musl.cc/$MUSL-cross.tgz" | tar -xzC /src/ \
    && CC=/src/$MUSL-cross/bin/$MUSL-gcc \
    && cd shadowsocks-rust \
    && git reset --hard ${COMMIT_ID} \
    && rustup target add $RUST_TARGET \
    && RUSTFLAGS="-C linker=$CC" \
    && CC=$CC \
    && cargo build --target "$RUST_TARGET" --release --features "local-tun local-redir armv8 neon stream-cipher aead-cipher-2022" \
    && mv target/$RUST_TARGET/release/ss* target/release/ 

FROM alpine:3.16 AS ssserver

COPY --from=builder /src/shadowsocks-rust/target/release/sslocal /usr/local/bin/
COPY --from=builder /src/shadowsocks-rust/target/release/ssserver /usr/local/bin/
COPY --from=builder /src/shadowsocks-rust/target/release/ssmanager /usr/local/bin/
COPY --from=builder /src/shadowsocks-rust/target/release/ssservice /usr/local/bin/
COPY --from=builder /src/shadowsocks-rust/examples/config.json /etc/shadowsocks-rust/
COPY docker-entrypoint.sh /usr/local/bin/

RUN set -x \
    && apk add --no-cache iproute2-ss

ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 8388

STOPSIGNAL SIGINT

CMD ["ssserver", "--log-without-time", "-a", "nobody", "-c", "/etc/shadowsocks-rust/config.json"]