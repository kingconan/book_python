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
r.keys()
r.dbsize()
r.delete("k1")
r.save() #将数据写回磁盘，保存时阻塞
r.flushdb() #清空r中数据


#redis学习再看文档请 https://pypi.python.org/pypi/redis/2.9.1
```



