##背景

>mongo支持在多个机器中通过异步复制达到故障转移和实现冗余，该架构被称作副本集，副本集中有只有一个主节点和多个从节点。只有主节点可以用于写操作，因此能保障mongo的数据一致性。主节点将改写数据库的动作分发给从数据库。

本部分演示经典的1主2从架构的副本集，在正式环境中，肯定“不会将鸡蛋全部放到一个篮子中”，但是为了演示的需要，将这三个数据库节点部署到同一台机器上。

##部署之路

- 创建数据文件存储路径
<pre>
sudo mkdir -p data/data/node1 data/data/node2 data/data/node3
</pre>

- 创建日志文件路径
<pre>
mkdir -p data/log/
</pre>

- 启动3个实例
<pre>
>sudo ./mongod -replSet rs0 --fork --port 10001 --smallfiles --dbpath /home/dialog/data/data/node1 --logpath=/home/dialog/data/log/node1.log --logappend

forked process: 635
child process started successfully, parent exiting

>sudo ./mongod -replSet rs0 --fork --port 10002 --smallfiles --dbpath /home/dialog/data/data/node2 --logpath=/home/dialog/data/log/node2.log --logappend

>sudo ./mongod -replSet rs0 --fork --port 10003 --smallfiles --dbpath /home/dialog/data/data/node3 --logpath=/home/dialog/data/log/node3.log --logappend

</pre>

- 配置及初始化副本集
<pre>
sudo ./mongo --port 10001//连接node1节点
[sudo] password for dialog: 
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10001/test
> rs.initiate()//初始化，集合只包含当前node1节点，使用默认参数配置
{
	"info2" : "no configuration explicitly specified -- making one",
	"me" : "ubuntu-dialog:10001",
	"info" : "Config now saved locally.  Should come online in about a minute.",
	"ok" : 1
}
rs0:OTHER> rs.conf()//使用conf()函数查看副本集配置
{
	"_id" : "rs0",
	"version" : 1,
	"members" : [
		{
			"_id" : 0,
			"host" : "ubuntu-dialog:10001"
		}
	]
}
rs0:PRIMARY> rs.add("ubuntu-dialog:10002")//增加其它节点
{ "ok" : 1 }
rs0:PRIMARY> rs.add("ubuntu-dialog:10003")
{ "ok" : 1 }
</pre>

- 查看副本集状态
<pre>
rs0:PRIMARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2014-09-02T03:31:55Z"),
        "myState" : 1,
        "members" : [
                {
                        "_id" : 0,
                        "name" : "ubuntu-dialog:10001",
                        "health" : 1,//1表示正常，0表示异常
                        "state" : 1,//1表示主节点，2表示从节点
                        "stateStr" : "PRIMARY",//PRIMARY是主节点，SECONDARY是从节点
                        "uptime" : 64419,
                        "optime" : Timestamp(1409565324, 1),
                        "optimeDate" : ISODate("2014-09-01T09:55:24Z"),
                        "electionTime" : Timestamp(1409565122, 2),
                        "electionDate" : ISODate("2014-09-01T09:52:02Z"),
                        "self" : true
                },
                {
                        "_id" : 1,
                        "name" : "ubuntu-dialog:10002",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 63401,
                        "optime" : Timestamp(1409565324, 1),
                        "optimeDate" : ISODate("2014-09-01T09:55:24Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T03:31:54Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T03:31:53Z"),
                        "pingMs" : 0,
                        "syncingTo" : "ubuntu-dialog:10001"
                },
                {
                        "_id" : 2,
                        "name" : "ubuntu-dialog:10003",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 63391,
                        "optime" : Timestamp(1409565324, 1),
                        "optimeDate" : ISODate("2014-09-01T09:55:24Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T03:31:54Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T03:31:54Z"),
                        "pingMs" : 0,
                        "syncingTo" : "ubuntu-dialog:10001"
                }
        ],
        "ok" : 1
}

调用isMaster()方法查看副本集状态

rs0:PRIMARY> rs.isMaster()
{
        "setName" : "rs0",
        "setVersion" : 3,
        "ismaster" : true,
        "secondary" : false,
        "hosts" : [
                "ubuntu-dialog:10001",
                "ubuntu-dialog:10003",
                "ubuntu-dialog:10002"
        ],
        "primary" : "ubuntu-dialog:10001",
        "me" : "ubuntu-dialog:10001",
        "maxBsonObjectSize" : 16777216,
        "maxMessageSizeBytes" : 48000000,
        "maxWriteBatchSize" : 1000,
        "localTime" : ISODate("2014-09-02T03:38:36.100Z"),
        "maxWireVersion" : 2,
        "minWireVersion" : 0,
        "ok" : 1
}


也可以从从节点查询副本集的状态。

sudo mongo --port 10002
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10002/test
rs0:SECONDARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2014-09-02T03:36:28Z"),
        "myState" : 2,
        "syncingTo" : "ubuntu-dialog:10001",
        "members" : [
                {
                        "_id" : 0,
                        "name" : "ubuntu-dialog:10001",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 63671,
                        "optime" : Timestamp(1409565324, 1),
                        "optimeDate" : ISODate("2014-09-01T09:55:24Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T03:36:27Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T03:36:28Z"),
                        "pingMs" : 0,
                        "electionTime" : Timestamp(1409565122, 2),
                        "electionDate" : ISODate("2014-09-01T09:52:02Z")
                },
                {
                        "_id" : 1,
                        "name" : "ubuntu-dialog:10002",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 64535,
                        "optime" : Timestamp(1409565324, 1),
                        "optimeDate" : ISODate("2014-09-01T09:55:24Z"),
                        "self" : true
                },
                {
                        "_id" : 2,
                        "name" : "ubuntu-dialog:10003",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 63663,
                        "optime" : Timestamp(1409565324, 1),
                        "optimeDate" : ISODate("2014-09-01T09:55:24Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T03:36:27Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T03:36:26Z"),
                        "pingMs" : 0,
                        "syncingTo" : "ubuntu-dialog:10001"
                }
        ],
        "ok" : 1
}

</pre>

- over，至此，已经在当台机器上，部署完成了三个mongodb数据库实例。

##探索同步的实现原理

>mongodb是如何维持集群之间数据的同步呢？这些架构都是基于1主节点N从节点，只有主节点可以接受增删改的写操作，因此数据库变化的部分可以记录在主节点的oplog集合中，其它从节点，按照周期轮询主节点的oplog，发现这些操作，然后作用到各种的数据库上，从而实现数据的同步。

oplog数据结合位于local数据库中。

<pre>
rs0:PRIMARY> use local
switched to db local
rs0:PRIMARY> show collections
me
oplog.rs//oplog数据集合
slaves
startup_log//启动log
system.indexes
system.replset//副本集信息

rs0:PRIMARY> db.oplog.rs.find()//查看op动作列表
{ "ts" : Timestamp(1409565122, 1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "initiating set" } }
{ "ts" : Timestamp(1409565314, 1), "h" : NumberLong("-7761836713755803631"), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "Reconfig set", "version" : 2 } }
{ "ts" : Timestamp(1409565324, 1), "h" : NumberLong("-8773000451936185356"), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "Reconfig set", "version" : 3 } }
</pre>

oplog字段说明：

- ts：某个字段的时间戳
- op：操作类型
	- i，插入
	- u，更新
	- d，删除
	- n
- ns：要操作的数据集合，命名空间
- o：要操作的文档内容

如对主节点进行插入操作，在test数据库的demo集合，插入文档{"target":"this is a replset set insert data demo"}

<pre>
rs0:PRIMARY> use test
switched to db test
rs0:PRIMARY> db.demo.insert({"targer":"this is a replset set insert data demo"})
WriteResult({ "nInserted" : 1 })

插入成功后，查看节点的oplog集合验证
rs0:PRIMARY> use local
switched to db local
rs0:PRIMARY> db.oplog.rs.find()
{ "ts" : Timestamp(1409565122, 1), "h" : NumberLong(0), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "initiating set" } }
{ "ts" : Timestamp(1409565314, 1), "h" : NumberLong("-7761836713755803631"), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "Reconfig set", "version" : 2 } }
{ "ts" : Timestamp(1409565324, 1), "h" : NumberLong("-8773000451936185356"), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "Reconfig set", "version" : 3 } }
{ "ts" : Timestamp(1409630582, 1), "h" : NumberLong("8319640463217202093"), "v" : 2, "op" : "i", "ns" : "test.demo", "o" : { "_id" : ObjectId("54054176f281f21d518d5426"), "targer" : "this is a replset set insert data demo" } }
</pre>

由于资源限制，oplog的数据集合是固定大小的，因此，只能保留一定大小的op历史记录，之前的都被冲洗掉了，下面演示查看oplog的元数据信息。

<pre>
rs0:PRIMARY> db.printReplicationInfo()
configured oplog size:   17806.373046875MB
log length start to end: 65460secs (18.18hrs)
oplog first event time:  Mon Sep 01 2014 17:52:02 GMT+0800 (CST)
oplog last event time:   Tue Sep 02 2014 12:03:02 GMT+0800 (CST)
now:                     Tue Sep 02 2014 12:13:09 GMT+0800 (CST)
</pre>

字段说明：

- configured oplog size，配置oplog的文件大小
- log length start to end，记录的log时间段，从第一条op开始，到最后一条op结束的时间跨度
- oplog first event time：当前记录中第一条op时间
- oplog last event time：当前记录中最后一条op时间
- now：当前时间

查看Save的同步状态

<pre>
rs0:PRIMARY> db.printSlaveReplicationInfo()
source: ubuntu-dialog:10002
        syncedTo: Tue Sep 02 2014 12:17:04 GMT+0800 (CST)
        0 secs (0 hrs) behind the primary 
source: ubuntu-dialog:10003
        syncedTo: Tue Sep 02 2014 12:17:04 GMT+0800 (CST)
        0 secs (0 hrs) behind the primary 
</pre>

字段说明：

- source：从节点的主机名和端口
- syncedTo：目前的同步状况，延迟了多久

查看主从节点的配置信息

<pre>
rs0:PRIMARY> use local
switched to db local
rs0:PRIMARY> db.system.replset.findOne()
{
        "_id" : "rs0",
        "version" : 3,
        "members" : [
                {
                        "_id" : 0,
                        "host" : "ubuntu-dialog:10001"
                },
                {
                        "_id" : 1,
                        "host" : "ubuntu-dialog:10002"
                },
                {
                        "_id" : 2,
                        "host" : "ubuntu-dialog:10003"
                }
        ]
}

同样，也可以通过rs.conf()进行查看
</pre>

##进阶

###进行读写分离设置

考虑集群架构，如果能将读和写操作分散到不同的机器去，那势必会提供系统的整体性能，这也是分布式处理的追求。mongo副本集要求只能从一个主服务器进行写操作，那默认可以从从节点读取吗？

<pre>
sudo mongo --port 10002
[sudo] password for dialog: 
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10002/test
rs0:SECONDARY> db.demo.find()
error: { "$err" : "not master and slaveOk=false", "code" : 13435 }
</pre>

出错了，看来默认不能从从节点读取数据。但是，通过以下操作，便可以从从数据库读取数据了。

<pre>
rs0:SECONDARY> db.setSlaveOk()
rs0:SECONDARY> db.demo.find()
{ "_id" : ObjectId("54054176f281f21d518d5426"), "targer" : "this is a replset set insert data demo" }
{ "_id" : ObjectId("540544c02c17ee576a70f6f9"), "what" : "log length test" }
</pre>

###系统鲁棒性

如上述建立的三节点的mongodb集群，其中主节点挂掉了，那么系统会从剩下的两个节点选取一个新的节点作为主节点，实现系统的正常运转。

- 关闭主节点

<pre>
>ps -aux | grep mongo

root       414  0.2  0.2 37361492 36660 ?      Sl   Sep01   3:01 ./mongod -replSet rs0 --fork --port 10001 --smallfiles --dbpath /home/dialog/data/data/node1 --logpath=/home/dialog/data/log/node1.log --logappend
root       635  0.2  0.2 36288792 35876 ?      Sl   Sep01   2:48 ./mongod -replSet rs0 --fork --port 10002 --smallfiles --dbpath /home/dialog/data/data/node2 --logpath=/home/dialog/data/log/node2.log --logappend
root       979  0.2  0.2 34192660 35528 ?      Sl   Sep01   2:47 ./mongod -replSet rs0 --fork --port 10003 --smallfiles --dbpath /home/dialog/data/data/node3 --logpath=/home/dialog/data/log/node3.log --logappend
mongodb   5888  0.3  0.8 855420 144088 ?       Ssl  Sep01   5:52 /usr/bin/mongod --config /etc/mongodb.conf
dialog   13357  0.0  0.0   9392   936 pts/0    S+   13:26   0:00 grep --color=auto mongo

kill -9 414
</pre>

- 查看副本集状态

<pre>
sudo mongo --port 10002
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10002/test
rs0:SECONDARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2014-09-02T05:29:47Z"),
        "myState" : 2,
        "syncingTo" : "ubuntu-dialog:10003",
        "members" : [
                {
                        "_id" : 0,
                        "name" : "ubuntu-dialog:10001",
                        "health" : 0,
                        "state" : 8,
                        "stateStr" : "(not reachable/healthy)",
                        "uptime" : 0,
                        "optime" : Timestamp(1409631424, 1),
                        "optimeDate" : ISODate("2014-09-02T04:17:04Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T05:29:45Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T05:28:15Z"),
                        "pingMs" : 0
                },
                {
                        "_id" : 1,
                        "name" : "ubuntu-dialog:10002",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 71334,
                        "optime" : Timestamp(1409631424, 1),
                        "optimeDate" : ISODate("2014-09-02T04:17:04Z"),
                        "infoMessage" : "syncing to: ubuntu-dialog:10003",
                        "self" : true
                },
                {
                        "_id" : 2,
                        "name" : "ubuntu-dialog:10003",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 70462,
                        "optime" : Timestamp(1409631424, 1),
                        "optimeDate" : ISODate("2014-09-02T04:17:04Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T05:29:46Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T05:29:46Z"),
                        "pingMs" : 0,
                        "electionTime" : Timestamp(1409635700, 1),
                        "electionDate" : ISODate("2014-09-02T05:28:20Z")
                }
        ],
        "ok" : 1
}
</pre>

可以看到，原先的主节点挂掉了，stateStr是不可达，系统推选原先的从节点（id为2）作为新的主节点。系统照样能正常运转

- 在新的主节点上操作数据，查看同步状况

<pre>
sudo mongo --port 10003
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10003/test
rs0:PRIMARY> use test
switched to db test
rs0:PRIMARY> db.demo.insert({"what":"a new primary node"})//插入新的文档
WriteResult({ "nInserted" : 1 })
rs0:PRIMARY> use local
switched to db local
rs0:PRIMARY> db.oplog.rs.find()//oplog中出现新插入文档的动作
{ "ts" : Timestamp(1409565324, 1), "h" : NumberLong("-8773000451936185356"), "v" : 2, "op" : "n", "ns" : "", "o" : { "msg" : "Reconfig set", "version" : 3 } }
{ "ts" : Timestamp(1409630582, 1), "h" : NumberLong("8319640463217202093"), "v" : 2, "op" : "i", "ns" : "test.demo", "o" : { "_id" : ObjectId("54054176f281f21d518d5426"), "targer" : "this is a replset set insert data demo" } }
{ "ts" : Timestamp(1409631424, 1), "h" : NumberLong("-448748148541609464"), "v" : 2, "op" : "i", "ns" : "test.demo", "o" : { "_id" : ObjectId("540544c02c17ee576a70f6f9"), "what" : "log length test" } }
{ "ts" : Timestamp(1409636015, 1), "h" : NumberLong("7452122732312765355"), "v" : 2, "op" : "i", "ns" : "test.demo", "o" : { "_id" : ObjectId("540556af1379022542143fa6"), "what" : "a new primary node" } }

sudo mongo --port 10002//连接另外一个从节点，查看数据同步情况
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10002/test
rs0:SECONDARY> use test
switched to db test
rs0:SECONDARY> db.demo.find()
error: { "$err" : "not master and slaveOk=false", "code" : 13435 }
rs0:SECONDARY> db.setSlaveOk()
rs0:SECONDARY> db.demo.find()
{ "_id" : ObjectId("54054176f281f21d518d5426"), "targer" : "this is a replset set insert data demo" }
{ "_id" : ObjectId("540544c02c17ee576a70f6f9"), "what" : "log length test" }
{ "_id" : ObjectId("540556af1379022542143fa6"), "what" : "a new primary node" }//发现同步数据
</pre>

- 那如果原先的主节点恢复连接了，那么数据会同步吗？还是主节点吗？

<pre>
//准备工作，删除node1的mongod.lock锁
sudo rm -rf  ~/data/data/node1/mongod.lock 


//呼唤原先的主节点node1
sudo ./mongod -replSet rs0 --fork --port 10001 --smallfiles --dbpath /home/dialog/data/data/node1 --logpath=/home/dialog/data/log/node1.log --logappend

sudo mongo --port 10001
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10001/test
rs0:SECONDARY> show dbs
admin   (empty)
local  17.523GB
test    0.031GB
rs0:SECONDARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2014-09-02T05:46:16Z"),
        "myState" : 2,
        "syncingTo" : "ubuntu-dialog:10002",
        "members" : [
                {
                        "_id" : 0,
                        "name" : "ubuntu-dialog:10001",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 61,
                        "optime" : Timestamp(1409636015, 1),
                        "optimeDate" : ISODate("2014-09-02T05:33:35Z"),
                        "infoMessage" : "syncing to: ubuntu-dialog:10002",
                        "self" : true
                },
                {
                        "_id" : 1,
                        "name" : "ubuntu-dialog:10002",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 59,
                        "optime" : Timestamp(1409636015, 1),
                        "optimeDate" : ISODate("2014-09-02T05:33:35Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T05:46:15Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T05:46:15Z"),
                        "pingMs" : 0,
                        "syncingTo" : "ubuntu-dialog:10003"
                },
                {
                        "_id" : 2,
                        "name" : "ubuntu-dialog:10003",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 57,
                        "optime" : Timestamp(1409636015, 1),
                        "optimeDate" : ISODate("2014-09-02T05:33:35Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T05:46:15Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T05:46:16Z"),
                        "pingMs" : 0,
                        "electionTime" : Timestamp(1409635700, 1),
                        "electionDate" : ISODate("2014-09-02T05:28:20Z")
                }
        ],
        "ok" : 1
}

可见，node1的状态已经是从节点了，而不是之前的主节点了。
通过查看从节点的同步状态，发现虽然中间node1断开连接了，但是重新加入副本集后，数据依然完成了同步更新。

rs0:SECONDARY> db.printSlaveReplicationInfo()
source: ubuntu-dialog:10001
        syncedTo: Tue Sep 02 2014 13:33:35 GMT+0800 (CST)
        0 secs (0 hrs) behind the primary 
source: ubuntu-dialog:10002
        syncedTo: Tue Sep 02 2014 13:33:35 GMT+0800 (CST)
        0 secs (0 hrs) behind the primary 

查看数据验证
rs0:SECONDARY> use test
switched to db test
rs0:SECONDARY> db.setSlaveOk()
rs0:SECONDARY> db.demo.find()
{ "_id" : ObjectId("54054176f281f21d518d5426"), "targer" : "this is a replset set insert data demo" }
{ "_id" : ObjectId("540544c02c17ee576a70f6f9"), "what" : "log length test" }
{ "_id" : ObjectId("540556af1379022542143fa6"), "what" : "a new primary node" }//同步数据
</pre>

###系统扩展

副本集提供了鲁棒性，同时通过增减节点提供负载均衡。当应用的读操作暴增时，3台机器已不能满足需求，可以通过增加一些节点将压力平均分配一下，当压力小的时候，就踢掉一些节点来减少硬件资源成本。总之，这就是运维的艺术。

####增加新节点

<pre>
//分配数据目录
mkdir -p ~/data/data/node4

//启动服务
sudo ./mongod -replSet rs0 --fork --port 10004 --smallfiles --dbpath /home/dialog/data/data/node4 --logpath=/home/dialog/data/log/node4.log --logappend

//连接到集群的一个节点，如主节点，将node4添加进集群
//首先查看当前状态
sudo mongo --port 10003
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10003/test
rs0:PRIMARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2014-09-02T05:59:52Z"),
        "myState" : 1,
        "members" : [
                {
                        "_id" : 0,
                        "name" : "ubuntu-dialog:10001",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 874,
                        "optime" : Timestamp(1409636015, 1),
                        "optimeDate" : ISODate("2014-09-02T05:33:35Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T05:59:52Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T05:59:51Z"),
                        "pingMs" : 0,
                        "syncingTo" : "ubuntu-dialog:10002"
                },
                {
                        "_id" : 1,
                        "name" : "ubuntu-dialog:10002",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 72265,
                        "optime" : Timestamp(1409636015, 1),
                        "optimeDate" : ISODate("2014-09-02T05:33:35Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T05:59:50Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T05:59:51Z"),
                        "pingMs" : 0,
                        "syncingTo" : "ubuntu-dialog:10003"
                },
                {
                        "_id" : 2,
                        "name" : "ubuntu-dialog:10003",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 72773,
                        "optime" : Timestamp(1409636015, 1),
                        "optimeDate" : ISODate("2014-09-02T05:33:35Z"),
                        "electionTime" : Timestamp(1409635700, 1),
                        "electionDate" : ISODate("2014-09-02T05:28:20Z"),
                        "self" : true
                }
        ],
        "ok" : 1
}

//添加node4到集群
rs0:PRIMARY> rs.add("ubuntu-dialog:10004")
{ "ok" : 1 }

中间经过初始化阶段，进行数据同步，初始化同步完成，节点添加完成阶段，进入最终的同步完成！这个阶段的时间长短依赖于数据集合的大小，耐心等待。

rs0:PRIMARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2014-09-02T06:03:26Z"),
        "myState" : 1,
        "members" : [
                {
                        "_id" : 0,
                        "name" : "ubuntu-dialog:10001",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 1088,
                        "optime" : Timestamp(1409637733, 1),
                        "optimeDate" : ISODate("2014-09-02T06:02:13Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T06:03:26Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T06:03:25Z"),
                        "pingMs" : 0,
                        "syncingTo" : "ubuntu-dialog:10002"
                },
                {
                        "_id" : 1,
                        "name" : "ubuntu-dialog:10002",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 72479,
                        "optime" : Timestamp(1409637733, 1),
                        "optimeDate" : ISODate("2014-09-02T06:02:13Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T06:03:26Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T06:03:25Z"),
                        "pingMs" : 0,
                        "syncingTo" : "ubuntu-dialog:10003"
                },
                {
                        "_id" : 2,
                        "name" : "ubuntu-dialog:10003",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 72987,
                        "optime" : Timestamp(1409637733, 1),
                        "optimeDate" : ISODate("2014-09-02T06:02:13Z"),
                        "electionTime" : Timestamp(1409635700, 1),
                        "electionDate" : ISODate("2014-09-02T05:28:20Z"),
                        "self" : true
                },
                {
                        "_id" : 3,
                        "name" : "ubuntu-dialog:10004",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 73,
                        "optime" : Timestamp(1409637733, 1),
                        "optimeDate" : ISODate("2014-09-02T06:02:13Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T06:03:25Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T06:03:26Z"),
                        "pingMs" : 0,
                        "syncingTo" : "ubuntu-dialog:10003"
                }
        ],
        "ok" : 1
}

验证新节点，完成数据同步

sudo mongo --port 10004
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10004/test
rs0:SECONDARY> use test
switched to db test
rs0:SECONDARY> db.demo.find()
error: { "$err" : "not master and slaveOk=false", "code" : 13435 }
rs0:SECONDARY> db.setSlaveOk()
rs0:SECONDARY> db.demo.find()
{ "_id" : ObjectId("54054176f281f21d518d5426"), "targer" : "this is a replset set insert data demo" }
{ "_id" : ObjectId("540544c02c17ee576a70f6f9"), "what" : "log length test" }
{ "_id" : ObjectId("540556af1379022542143fa6"), "what" : "a new primary node" }
</pre>


####通过数据库快照(--fastsync和oplog增加节点)

>上面讲述的通过oplog进行节点新增，不是万能的。因为oplog集合是固定集合，受困于资源，其只能记录一部分op操作，当oplog写满后，会覆盖掉集合一部分记录。那么当集群已经运行很长时间了，这时再加入新节点，如果只是通过oplog进行数据同步，相比，必定丢失部分数据，造成数据同步不一致，这种怎么破？

<pre>
首先创建node5的数据节点
mkdir -p ~/data/data/node5

拷贝node3的数据快照到node5
sudo  cp -rf  ~/data/data/node3/* ~/data/data/node5

拷贝完成后，在node3新增一个文档，这个文档是和node5中快照的区别点
sudo mongo --port 10003
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10003/test
rs0:PRIMARY> use test
switched to db test
rs0:PRIMARY> db.demo.find()
{ "_id" : ObjectId("54054176f281f21d518d5426"), "targer" : "this is a replset set insert data demo" }
{ "_id" : ObjectId("540544c02c17ee576a70f6f9"), "what" : "log length test" }
{ "_id" : ObjectId("540556af1379022542143fa6"), "what" : "a new primary node" }
rs0:PRIMARY> db.demo.insert({'what':'how to add a node'})
WriteResult({ "nInserted" : 1 })

在10005端口启动mongo数据实例
sudo ./mongod -replSet rs0 --fork --port 10005 --smallfiles --dbpath /home/dialog/data/data/node5 --logpath=/home/dialog/data/log/node5.log --logappend --fastsync

将node5添加到副本集中
rs.add('ubuntu-dialog:10005')
{ "ok" : 1 }

添加成功！
rs0:PRIMARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2014-09-02T06:29:37Z"),
        "myState" : 1,
        "members" : [
                {
                        "_id" : 0,
                        "name" : "ubuntu-dialog:10001",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 123,
                        "optime" : Timestamp(1409639320, 1),
                        "optimeDate" : ISODate("2014-09-02T06:28:40Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T06:29:36Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T06:29:35Z"),
                        "pingMs" : 0,
                        "lastHeartbeatMessage" : "syncing to: ubuntu-dialog:10003",
                        "syncingTo" : "ubuntu-dialog:10003"
                },
                {
                        "_id" : 1,
                        "name" : "ubuntu-dialog:10002",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 123,
                        "optime" : Timestamp(1409639320, 1),
                        "optimeDate" : ISODate("2014-09-02T06:28:40Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T06:29:36Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T06:29:35Z"),
                        "pingMs" : 0,
                        "lastHeartbeatMessage" : "syncing to: ubuntu-dialog:10003",
                        "syncingTo" : "ubuntu-dialog:10003"
                },
                {
                        "_id" : 2,
                        "name" : "ubuntu-dialog:10003",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 74558,
                        "optime" : Timestamp(1409639320, 1),
                        "optimeDate" : ISODate("2014-09-02T06:28:40Z"),
                        "electionTime" : Timestamp(1409635700, 1),
                        "electionDate" : ISODate("2014-09-02T05:28:20Z"),
                        "self" : true
                },
                {
                        "_id" : 3,
                        "name" : "ubuntu-dialog:10005",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 57,
                        "optime" : Timestamp(1409639320, 1),
                        "optimeDate" : ISODate("2014-09-02T06:28:40Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T06:29:36Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T06:29:35Z"),
                        "pingMs" : 0,
                        "lastHeartbeatMessage" : "syncing to: ubuntu-dialog:10001",
                        "syncingTo" : "ubuntu-dialog:10001"
                }
        ],
        "ok" : 1
}

数据同步成功
sudo mongo --port 10005
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10005/test
rs0:SECONDARY> db.demo.find()
error: { "$err" : "not master and slaveOk=false", "code" : 13435 }
rs0:SECONDARY> db.setSlaveOk()
rs0:SECONDARY> db.demo.find()
{ "_id" : ObjectId("54054176f281f21d518d5426"), "targer" : "this is a replset set insert data demo" }
{ "_id" : ObjectId("540544c02c17ee576a70f6f9"), "what" : "log length test" }
{ "_id" : ObjectId("540556af1379022542143fa6"), "what" : "a new primary node" }
{ "_id" : ObjectId("54056279e98d0f48c30a10d0"), "what" : "how to add a node" }

</pre>

####删除副本集成员

过河拆桥呀！so easy

<pre>
 sudo mongo --port 10003
MongoDB shell version: 2.6.4
connecting to: 127.0.0.1:10003/test
rs0:PRIMARY> rs.remove('ubuntu-dialog:10005')
2014-09-02T14:34:42.968+0800 DBClientCursor::init call() failed
2014-09-02T14:34:42.969+0800 Error: error doing query: failed at src/mongo/shell/query.js:81
2014-09-02T14:34:42.971+0800 trying reconnect to 127.0.0.1:10003 (127.0.0.1) failed
2014-09-02T14:34:42.971+0800 reconnect 127.0.0.1:10003 (127.0.0.1) ok
rs0:PRIMARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2014-09-02T06:34:49Z"),
        "myState" : 1,
        "members" : [
                {
                        "_id" : 0,
                        "name" : "ubuntu-dialog:10001",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 7,
                        "optime" : Timestamp(1409639682, 1),
                        "optimeDate" : ISODate("2014-09-02T06:34:42Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T06:34:48Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T06:34:47Z"),
                        "pingMs" : 0,
                        "lastHeartbeatMessage" : "syncing to: ubuntu-dialog:10003",
                        "syncingTo" : "ubuntu-dialog:10003"
                },
                {
                        "_id" : 1,
                        "name" : "ubuntu-dialog:10002",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 7,
                        "optime" : Timestamp(1409639682, 1),
                        "optimeDate" : ISODate("2014-09-02T06:34:42Z"),
                        "lastHeartbeat" : ISODate("2014-09-02T06:34:48Z"),
                        "lastHeartbeatRecv" : ISODate("2014-09-02T06:34:47Z"),
                        "pingMs" : 0,
                        "lastHeartbeatMessage" : "syncing to: ubuntu-dialog:10003",
                        "syncingTo" : "ubuntu-dialog:10003"
                },
                {
                        "_id" : 2,
                        "name" : "ubuntu-dialog:10003",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 74870,
                        "optime" : Timestamp(1409639682, 1),
                        "optimeDate" : ISODate("2014-09-02T06:34:42Z"),
                        "electionTime" : Timestamp(1409635700, 1),
                        "electionDate" : ISODate("2014-09-02T05:28:20Z"),
                        "self" : true
                }
        ],
        "ok" : 1
}
rs0:PRIMARY> 
</pre>

ok。副本集的操作已经完成了。吐血整理完。。。