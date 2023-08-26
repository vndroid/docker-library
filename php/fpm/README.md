## php-fpm

## 特性

- 基于 Alpine Linux 3.18
- 基于 PHP 8.2
- 添加常规扩展 bcmath calendar exif opcache mysqli pdo_mysql sockets bz2 gd xml gmp pgsql pdo_pgsql
- 添加 pecl 扩展 igbinary msgpack redis xmlrpc memcached swoole apcu yaml xdebug
- 修复 msgpack 与 memcached 扩展冲突的问题