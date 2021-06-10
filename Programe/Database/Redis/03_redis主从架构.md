# 主从架构配置

```shell
#复制一份redis.conf文件
#将相关配置修改为如下值
#修改端口
port 6380
# 把pid进程号写入pidfile配置的文件
pidfile /var/run/redis_6380.pid  
#配置日志文件名
logfile "redis.log"
# 指定持久化数据存放目录
dir /usr/local/redis-5.0.3/data 
# 需要注释掉bind
# bind 127.0.0.1（bind绑定的是自己机器网卡的ip，如果有多块网卡可以配多个ip，代表允许客户端通过机器的哪些网卡ip去访问，内网一般可以不配置bind，注释掉即可）
#配置主从复制
 # 从本机6379的redis实例复制数据，Redis 5.0之前使用slaveof
replicaof 192.168.0.60 6379  
# 配置从节点只读
replica-read-only yes
```

