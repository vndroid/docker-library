## memtier_benchmark

upstream: https://github.com/RedisLabs/memtier_benchmark

### How to use

```bash
## memcached
memtier_benchmark -s memcached -p 11211 --protocol=memcache_text --threads=4 --clients=50 --requests=100000 --ratio=1:10
## redis
memtier_benchmark -s redis -p 6379 --ratio=1:10 -c 100 -t 8 -n 20000
```