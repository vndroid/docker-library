FROM alpine:3.19
# https://github.com/zulu-openjdk/zulu-openjdk/blob/master/alpine/8-latest/Dockerfile

ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8
ENV TZ=Etc/UTC
ARG ZULU_KEY_SHA256=6c6393d4755818a15cf055a5216cffa599f038cd508433faed2226925956509a
RUN wget --quiet https://cdn.azul.com/public_keys/alpine-signing@azul.com-5d5dc44c.rsa.pub -P /etc/apk/keys/ && \
    echo "${ZULU_KEY_SHA256}  /etc/apk/keys/alpine-signing@azul.com-5d5dc44c.rsa.pub" | sha256sum -c - && \
    apk --repository https://repos.azul.com/zulu/alpine --no-cache add zulu8-jdk~=8.0.442 tzdata

ENV JAVA_HOME=/usr/lib/jvm/zulu8