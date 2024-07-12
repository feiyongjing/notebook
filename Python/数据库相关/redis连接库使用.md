### 安装redis依赖（笔记中使用的redis库版本是5.0.4）
~~~shell
pip install redis
~~~

### redis连接使用
~~~python
import asyncio

import redis.asyncio as redis

redis_host: str="localhost"
redis_port: int=6379
redis_password: str=""
redis_db: int= 0

redis_dsn: str= f"redis://:{redis_password}@{redis_host}:{redis_port}/{redis_db}"

async def main():
    r = redis.from_url(redis_dsn, decode_responses=True)
    async with r:
        print(await r.get("a"))


if __name__ == "__main__":
    asyncio.run(main())


~~~

