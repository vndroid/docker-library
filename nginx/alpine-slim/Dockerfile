# Layer 1 (tcmalloc)
FROM alpine:3.16 AS builder1

ARG TCMALLOC_COMMIT bf8b714bf5075d0a6f2f28504b43095e2b1e11c5

WORKDIR /src

RUN set -x \
    && apk add --no-cache build-base git \
    && apk add --no-cache libunwind-dev \
    && apk add --no-cache autoconf automake libtool linux-headers \
    && git clone --depth 1 https://github.com/gperftools/gperftools.git \
    && cd gperftools \
    && git fetch --depth 1 origin $TCMALLOC_COMMIT \
    && ./autogen.sh \
    && ./configure \
    && make -j$(getconf _NPROCESSORS_ONLN) \
    && strip /src/gperftools/.libs/libprofiler.so.0.* \
    && strip /src/gperftools/.libs/libtcmalloc.so.4.* \
    && strip /src/gperftools/.libs/libtcmalloc_minimal.so.4.*

# Layer 2 (boringssl)
FROM golang:alpine3.16 AS builder2

ARG BORINGSSL_COMMIT 0bfc9b32c4ec860dc46338d15320532c5e4496c8

WORKDIR /src

RUN set -x \
    && apk add --no-cache libunwind-dev \
    && apk add --no-cache build-base cmake perl git linux-headers pkgconfig \
    && apk add --no-cache ninja \
    && git clone --depth 1 https://boringssl.googlesource.com/boringssl \
    && cd boringssl \
    && git fetch --depth 1 origin $BORINGSSL_COMMIT \
    && mkdir build \
    && cd build \
    && cmake -GNinja -DBUILD_SHARED_LIBS=1 -DCMAKE_INSTALL_PREFIX=/usr .. \
    && ninja \
    && ninja install \
    && strip /src/boringssl/build/crypto/libcrypto.so \
    && strip /src/boringssl/build/ssl/libssl.so

# Layer 3 (modules)

FROM nginx:1.22.0-alpine-perl AS builder3

COPY --from=builder2 /src/boringssl /src/boringssl

# https://www.nginx.com/nginx-wiki/build/dirhtml/modules/
ARG NGX_BROTLI_COMMIT 6e975bcb015f62e1f303054897783355e2a877dc
ARG NGX_NDK_COMMIT b4642d6ca01011bd8cd30b253f5c3872b384fd21
ARG NGX_NAXSI_COMMIT 29793dcb2d9bdfa02c9b8d6968063ce3daa0a77b
ARG NGX_SMNM_COMMIT 3ea21393340a22a6a4dc3f9c1e07b3aac26a5c56
ARG NGX_HMNM_COMMIT d502e41996d24a382bd9c632e3ae3efa0a5fca66
ARG NGX_ESNM_COMMIT 8250e6a1d03b0d82358ac9dd478d93038da3ae24
ARG NGX_TANM_COMMIT d53a4a6e05d2571b9c7a090a8127e51597cbe9eb


WORKDIR /src

RUN set -x \
    && apk add --no-cache build-base cmake linux-headers pkgconfig libunwind-static \
    && apk add --no-cache git mercurial perl-dev pcre-dev pcre2-dev zlib-dev libxslt-dev gd-dev geoip-dev \
    && hg clone https://hg.nginx.org/nginx \
    && hg clone https://hg.nginx.org/njs -r $NJS_VERSION \
    && git clone --recursive https://github.com/google/ngx_brotli.git \
    && git clone --recursive https://github.com/vision5/ngx_devel_kit.git \
    && git clone --recursive https://github.com/nbs-system/naxsi.git \
    && git clone --recursive https://github.com/openresty/set-misc-nginx-module.git \
    && git clone --recursive https://github.com/openresty/headers-more-nginx-module.git \
    && git clone --recursive https://github.com/openresty/encrypted-session-nginx-module.git \
    && git clone --recursive https://github.com/Lax/traffic-accounting-nginx-module.git \
    && cd /src/ngx_brotli \
    && git fetch --depth 1 origin $NGX_BROTLI_COMMIT \
    && cd /src/ngx_devel_kit \
    && git fetch --depth 1 origin $NGX_NDK_COMMIT \
    && cd /src/naxsi \
    && git fetch --depth 1 origin $NGX_NAXSI_COMMIT \
    && cd /src/set-misc-nginx-module \
    && git fetch --depth 1 origin $NGX_SMNM_COMMIT \
    && cd /src/headers-more-nginx-module \
    && git fetch --depth 1 origin $NGX_HMNM_COMMIT \
    && cd /src/encrypted-session-nginx-module \
    && git fetch --depth 1 origin $NGX_ESNM_COMMIT \
    && cd /src/traffic-accounting-nginx-module \
    && git fetch --depth 1 origin $NGX_TANM_COMMIT \
    && cd /src/nginx \
    && hg up release-$NGINX_VERSION \
    && auto/configure `nginx -V 2>&1 | sed "s/ \-\-/ \\\ \n\t--/g" | grep "\-\-" | grep -ve opt= -e param= -e build=` \
                      --add-dynamic-module=/src/njs/nginx \
                      --add-dynamic-module=/src/ngx_brotli \
                      --add-dynamic-module=/src/ngx_devel_kit \
                      --add-dynamic-module=/src/naxsi/naxsi_src \
                      --add-dynamic-module=/src/set-misc-nginx-module \
                      --add-dynamic-module=/src/headers-more-nginx-module \
                      --add-dynamic-module=/src/encrypted-session-nginx-module \
                      --add-dynamic-module=/src/traffic-accounting-nginx-module \
                      --with-cc-opt="-I/src/boringssl/include" \
                      --with-ld-opt="-L/src/boringssl/build/ssl -L/src/boringssl/build/crypto" \
    && make modules -j$(getconf _NPROCESSORS_ONLN) \
    && strip /src/nginx/objs/*.so \
    && cd /src/njs/ \
    && apk add --no-cache readline-dev \
    && ./configure \
       --cc-opt="-I/src/boringssl/include" \
       --ld-opt="-L/src/boringssl/build/ssl -L/src/boringssl/build/crypto" \
    && make -j$(getconf _NPROCESSORS_ONLN) \
    && strip /src/njs/build/njs
# For latest build deps, see https://github.com/nginxinc/docker-nginx/blob/master/stable/alpine-perl/Dockerfile

# Layer 4 (main)
FROM nginx:1.22.0-alpine-perl AS builder4

COPY --from=builder1 /src/gperftools/.libs/libprofiler.so.0.5.5 /usr/local/lib/
COPY --from=builder1 /src/gperftools/.libs/libtcmalloc.so.4.5.10 /usr/local/lib/
COPY --from=builder1 /src/gperftools/.libs/libtcmalloc_minimal.so.4.5.10 /usr/local/lib/
COPY --from=builder2 /usr/include/openssl/* /usr/include/openssl/
COPY --from=builder2 /usr/lib/cmake/OpenSSL/* /usr/lib/cmake/OpenSSL/
COPY --from=builder2 /usr/lib/libcrypto.so /usr/lib/
COPY --from=builder2 /usr/lib/libssl.so /usr/lib/
COPY --from=builder2 /usr/bin/bssl /usr/bin/

WORKDIR /src

RUN set -x \
    && apk add --no-cache libstdc++ libunwind \
    && cd /usr/local/lib && { ln -s libtcmalloc.so.4.5.10 libtcmalloc.so.4 || { rm -f libtcmalloc.so.4 && ln -s libtcmalloc.so.4.5.10 libtcmalloc.so.4; }; } \
    && cd /usr/local/lib && { ln -s libtcmalloc.so.4.5.10 libtcmalloc.so || { rm -f libtcmalloc.so && ln -s libtcmalloc.so.4.5.10 libtcmalloc.so; }; } \
    && cd /usr/local/lib && { ln -s libtcmalloc_minimal.so.4.5.10 libtcmalloc_minimal.so.4 || { rm -f libtcmalloc_minimal.so.4 && ln -s libtcmalloc_minimal.so.4.5.10 libtcmalloc_minimal.so.4; }; } \
    && cd /usr/local/lib && { ln -s libtcmalloc_minimal.so.4.5.10 libtcmalloc_minimal.so || { rm -f libtcmalloc_minimal.so && ln -s libtcmalloc_minimal.so.4.5.10 libtcmalloc_minimal.so; }; } \
    && cd /usr/local/lib && { ln -s libprofiler.so.0.5.5 libprofiler.so.0 || { rm -f libprofiler.so.0 && ln -s libprofiler.so.0.5.5 libprofiler.so.0; }; } \
    && cd /usr/local/lib && { ln -s libprofiler.so.0.5.5 libprofiler.so || { rm -f libprofiler.so && ln -s libprofiler.so.0.5.5 libprofiler.so; }; }

RUN set -x \
    && apk add --no-cache build-base cmake perl git libunwind-static linux-headers pkgconfig \
    && apk add --no-cache mercurial perl-dev pcre2-dev zlib-dev libxslt-dev gd-dev geoip-dev \
    && hg clone https://hg.nginx.org/nginx \
    && cd nginx \
    && hg up release-$NGINX_VERSION \
    && auto/configure `nginx -V 2>&1 | sed "s/ \-\-/ \\\ \n\t--/g" | grep "\-\-" | grep -ve opt= -e param= -e build=` \
                      --build=nginx-plus \
                      --with-google_perftools_module \
                      --with-cc-opt='-Os -fomit-frame-pointer -g' \
                      --with-ld-opt=-Wl,--as-needed,-O1,--sort-common \
    && make -j$(getconf _NPROCESSORS_ONLN) \
    && strip /src/nginx/objs/nginx

# Layer 5 (final)

FROM alpine:3.16

LABEL maintainer="WavE Docker Maintainers <waveworkshop@outlook.com>"
# copy tcmalloc shared Libraries
COPY --from=builder1 /src/gperftools/.libs/libprofiler.so.0.5.5 /usr/local/lib/
COPY --from=builder1 /src/gperftools/.libs/libtcmalloc.so.4.5.10 /usr/local/lib/
COPY --from=builder1 /src/gperftools/.libs/libtcmalloc_minimal.so.4.5.10 /usr/local/lib/
# copy boringssl shared Libraries
COPY --from=builder2 /src/boringssl/build/crypto/libcrypto.so /usr/local/lib/
COPY --from=builder2 /src/boringssl/build/ssl/libssl.so /usr/local/lib/
# Google tcmalloc need libc Support
RUN apk add --no-cache libstdc++ libunwind \
    && cd /usr/local/lib && { ln -s libtcmalloc.so.4.5.10 libtcmalloc.so.4 || { rm -f libtcmalloc.so.4 && ln -s libtcmalloc.so.4.5.10 libtcmalloc.so.4; }; } \
    && cd /usr/local/lib && { ln -s libtcmalloc.so.4.5.10 libtcmalloc.so || { rm -f libtcmalloc.so && ln -s libtcmalloc.so.4.5.10 libtcmalloc.so; }; } \
    && cd /usr/local/lib && { ln -s libtcmalloc_minimal.so.4.5.10 libtcmalloc_minimal.so.4 || { rm -f libtcmalloc_minimal.so.4 && ln -s libtcmalloc_minimal.so.4.5.10 libtcmalloc_minimal.so.4; }; } \
    && cd /usr/local/lib && { ln -s libtcmalloc_minimal.so.4.5.10 libtcmalloc_minimal.so || { rm -f libtcmalloc_minimal.so && ln -s libtcmalloc_minimal.so.4.5.10 libtcmalloc_minimal.so; }; } \
    && cd /usr/local/lib && { ln -s libprofiler.so.0.5.5 libprofiler.so.0 || { rm -f libprofiler.so.0 && ln -s libprofiler.so.0.5.5 libprofiler.so.0; }; } \
    && cd /usr/local/lib && { ln -s libprofiler.so.0.5.5 libprofiler.so || { rm -f libprofiler.so && ln -s libprofiler.so.0.5.5 libprofiler.so; }; }

# Origin Source

ENV NGINX_VERSION 1.22.0
ENV NJS_VERSION   0.7.5
ENV PKG_RELEASE   1

RUN set -x \
# create nginx user/group first, to be consistent throughout docker variants
    && addgroup -g 101 -S nginx \
    && adduser -S -D -H -u 101 -h /var/cache/nginx -s /sbin/nologin -G nginx -g nginx nginx \
    && apkArch="$(cat /etc/apk/arch)" \
    && nginxPackages=" \
        nginx=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-xslt=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-geoip=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-image-filter=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-perl=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-njs=${NGINX_VERSION}.${NJS_VERSION}-r${PKG_RELEASE} \
    " \
# install prerequisites for public key and pkg-oss checks
    && apk add --no-cache --virtual .checksum-deps \
        openssl \
    && case "$apkArch" in \
        x86_64|aarch64) \
# arches officially built by upstream
            set -x \
            && KEY_SHA512="e7fa8303923d9b95db37a77ad46c68fd4755ff935d0a534d26eba83de193c76166c68bfe7f65471bf8881004ef4aa6df3e34689c305662750c0172fca5d8552a *stdin" \
            && wget -O /tmp/nginx_signing.rsa.pub https://nginx.org/keys/nginx_signing.rsa.pub \
            && if [ "$(openssl rsa -pubin -in /tmp/nginx_signing.rsa.pub -text -noout | openssl sha512 -r)" = "$KEY_SHA512" ]; then \
                echo "key verification succeeded!"; \
                mv /tmp/nginx_signing.rsa.pub /etc/apk/keys/; \
            else \
                echo "key verification failed!"; \
                exit 1; \
            fi \
            && apk add -X "https://nginx.org/packages/alpine/v$(egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release)/main" --no-cache $nginxPackages \
            ;; \
        *) \
# we're on an architecture upstream doesn't officially build for
# let's build binaries from the published packaging sources
            set -x \
            && tempDir="$(mktemp -d)" \
            && chown nobody:nobody $tempDir \
            && apk add --no-cache --virtual .build-deps \
                gcc \
                libc-dev \
                make \
                openssl-dev \
                pcre2-dev \
                zlib-dev \
                linux-headers \
                libxslt-dev \
                gd-dev \
                geoip-dev \
                perl-dev \
                libedit-dev \
                bash \
                alpine-sdk \
                findutils \
            && su nobody -s /bin/sh -c " \
                export HOME=${tempDir} \
                && cd ${tempDir} \
                && curl -f -O https://hg.nginx.org/pkg-oss/archive/696.tar.gz \
                && PKGOSSCHECKSUM=\"fabf394af60d935d7c3f5e36db65dddcced9595fd06d3dfdfabbb77aaea88a5b772ef9c1521531673bdbb2876390cdea3b81c51030d36ab76cf5bfc0bfe79230 *696.tar.gz\" \
                && if [ \"\$(openssl sha512 -r 696.tar.gz)\" = \"\$PKGOSSCHECKSUM\" ]; then \
                    echo \"pkg-oss tarball checksum verification succeeded!\"; \
                else \
                    echo \"pkg-oss tarball checksum verification failed!\"; \
                    exit 1; \
                fi \
                && tar xzvf 696.tar.gz \
                && cd pkg-oss-696 \
                && cd alpine \
                && make all \
                && apk index -o ${tempDir}/packages/alpine/${apkArch}/APKINDEX.tar.gz ${tempDir}/packages/alpine/${apkArch}/*.apk \
                && abuild-sign -k ${tempDir}/.abuild/abuild-key.rsa ${tempDir}/packages/alpine/${apkArch}/APKINDEX.tar.gz \
                " \
            && cp ${tempDir}/.abuild/abuild-key.rsa.pub /etc/apk/keys/ \
            && apk del .build-deps \
            && apk add -X ${tempDir}/packages/alpine/ --no-cache $nginxPackages \
            ;; \
    esac \
# remove checksum deps
    && apk del .checksum-deps \
# if we have leftovers from building, let's purge them (including extra, unnecessary build deps)
    && if [ -n "$tempDir" ]; then rm -rf "$tempDir"; fi \
    && if [ -n "/etc/apk/keys/abuild-key.rsa.pub" ]; then rm -f /etc/apk/keys/abuild-key.rsa.pub; fi \
    && if [ -n "/etc/apk/keys/nginx_signing.rsa.pub" ]; then rm -f /etc/apk/keys/nginx_signing.rsa.pub; fi \
# Bring in gettext so we can get `envsubst`, then throw
# the rest away. To do this, we need to install `gettext`
# then move `envsubst` out of the way so `gettext` can
# be deleted completely, then move `envsubst` back.
    && apk add --no-cache --virtual .gettext gettext \
    && mv /usr/bin/envsubst /tmp/ \
    \
    && runDeps="$( \
        scanelf --needed --nobanner /tmp/envsubst \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | sort -u \
            | xargs -r apk info --installed \
            | sort -u \
    )" \
    && apk add --no-cache $runDeps \
    && apk del .gettext \
    && mv /tmp/envsubst /usr/local/bin/ \
# Bring in tzdata so users could set the timezones through the environment
# variables
    && apk add --no-cache tzdata \
# Bring in curl and ca-certificates to make registering on DNS SD easier
    && apk add --no-cache curl ca-certificates \
# Add iproute2 Support
    && apk add --no-cache iproute2-ss pcre2 readline \
# Replace Nginx Exec
    && (test ! -f '/usr/sbin/nginx' || rm -f '/usr/sbin/nginx') \
# Make cache directory (https://www.nginx.com/resources/wiki/start/topics/examples/reverseproxycachingexample/)
    && mkdir -p /tmp/cache/ \
# forward request and error logs to docker log collector
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \
# create a docker-entrypoint.d directory
    && mkdir /docker-entrypoint.d

COPY --from=builder3 /src/nginx/objs/*.so /usr/lib/nginx/modules/
COPY --from=builder3 /src/njs/build/njs /usr/bin/
COPY --from=builder4 /src/nginx/objs/nginx /usr/sbin/

COPY 10-listen-on-ipv6-by-default.sh /docker-entrypoint.d
COPY 20-envsubst-on-templates.sh /docker-entrypoint.d
COPY 30-tune-worker-processes.sh /docker-entrypoint.d
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 80

STOPSIGNAL SIGQUIT

CMD ["nginx", "-g", "daemon off;"]
