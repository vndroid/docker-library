## php

### 存档

- PHP 7.4 存档

### 上游

- [docker-php](https://github.com/docker-library/php)

### 健康检查

- [php-fpm-healthcheck](https://github.com/renatomefi/php-fpm-healthcheck)

构建方法

```dockerfile
FROM php:7.2-fpm-alpine3.8

# Required software
RUN apk add --no-cache fcgi

# Enable php fpm status page
RUN set -xe && echo "pm.status_path = /status" >> /usr/local/etc/php-fpm.d/zz-docker.conf

COPY ./php-fpm-healthcheck /usr/local/bin/
```

使用方法

```bash
php-fpm-healthcheck
## 正常则返回 0
echo $?
```