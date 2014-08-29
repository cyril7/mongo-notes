使用MongoVUE客户端访问mongo服务器，出现积极拒绝，原因有以下：

1. 你确定自己的mongo服务器没有挂掉
2. 你确定MongoVUE的连接mongo的配置都正确，如ip，端口号等
3. 如果上述都ok，那么你需要检查下mongo的配置文件，/etc/mongod.conf，看下是否bind_ip到127.0.0.1上了，即只监听本地服务了，没有监听来自网络的其他请求。