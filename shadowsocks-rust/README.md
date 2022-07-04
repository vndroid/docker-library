## shadowsocks-rust

同步于官方 upstream [shadowsocks-rust](https://github.com/shadowsocks/shadowsocks-rust)

使用方法与官方镜像 100% 兼容，去掉多余的 buildx 参数，使用常规构建方法即可构建镜像。

```bash
docker image build -t shadowsocks:rust-alpine .
```