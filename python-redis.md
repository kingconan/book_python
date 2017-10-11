## Python-Redis

```py
1. 机器上先安装redis并启动

2. test redis
>> redis-cli可以进入redis的命令行，进行简单的get set测试

3. pip install redis,安装python的redis包

import redis
r = redis.Redis(host="127.0.0.1", port=6379, db=0)
r.set("k1", "hello world")
r.get("k1")
```



