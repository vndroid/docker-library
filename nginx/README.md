## nginx-alpine-perl

### 特性

- 基于 Alpine Linux
- 基于 NGiNX 稳定分支
- 基于 OpenSSL 3.x 版本

### 上游

- [docker-nginx](https://github.com/nginxinc/docker-nginx/tree/master/stable)

## 补丁

制作补丁

```bash
diff -Naur alpine/ deluxe/ > reduce.patch
```