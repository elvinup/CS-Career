## Install

Installs Redis with the RedisInsight GUI tool 
```
docker run -d --name redis-stack -p 6379:6379 -p 8001:8001 -e REDIS_ARGS="--requirepass mypassword" redis/redis-stack:latest
```

Install redis-cli
```
rtx global redis-cli@latest
```