使用复制，不仅可以应对故障切换，数据集成，还可以用来做读扩展，热备份或作为离线批处理的数据源。

##主从复制

最基本的设置是建立一个主节点和一个或多个从节点，每个从节点要知道主节点的地址。

正式环境有多个服务器，我们为了演示的需要，现在本机上试验好了。

<pre>
sudo mkdir -p ~/dbs/master
sudo mongod --dbpath ~/dbs/master --port 10000 --master

sudo mkdir -p ~/dbs/master
sudo mongod --dbpath ~/dbs/slave --port 10001 --slave --source localhost:10000
</pre>

ok！所有从节点都从主节点复制内容。目前，还没有能够从从节点复制的机制，原因就是从节点并不保存自己的oplog。

一个集群中有多少个从节点并没有明确的限制，但是上千个（妈蛋，你以为自己是NBA球星张伯伦呀！）从节点对单个主机点发起查询也会让其吃不消。所有，在实践中12个从节点的集群就可以良好的运行了。

主从复制的一些选项：

- only 指定只复制特定某个数据库(默认复制所有的数据库)
- slavedelay 用在从节点上，当应用主节点的操作时增加延时（秒）、这样就能设置延时从节点了，这种节点对用户无意删除某个文档或者插入垃圾数据等事故有很重要的防护作用。这些不良操作都会复制到所有从节点，通过延缓执行操作，可以有个恢复的时间差。
- fastsync
- autoresync
- oplogSize

添加及删除源

启动从节点时，可以用source指定主节点，也可以在shell中配置这个源。

假设主节点绑定在localhost:10000，启动从节点时可以不添加源，而是随后向source集合添加主节点信息：

<pre>
sudo mongod --slave --dbpath ~/dbs/slave --port 10001

通过shell运行，将localhost:10000作为源添加到从节点上
use local
db.sources.insert({'host':'localhost:10000'})
</pre>

可以看到，在source集合可以被当做普通集合进行操作，而且为管理从节点提供了很大的灵活性！这就是活雷锋呀！

##副本集（Replica Set）

副本集有自动故障恢复功能的主从集群。主从集群和副本集最为明显的区别是副本没有固定的主节点：整个集群会选举出一个”主节点“，当其不能工作时，则变更到其它节点。然而，二者看上去很相似，副本集总有一个活跃节点和一个活多个备份节点。

副本集最牛逼的地方就是所有东西都可以自动化。具体体现在：

- 自动提升备份节点为活跃节点，以确保运行正常。
- 对维护人员来说，仅需要为副本集指定一下服务器，驱动程序就会自动找到服务器，在当前活跃节点死机时自动处理故障恢复这类事情。

先从最简单的例子开始：两个服务器。

ps。不能用localhost地址作为成员，得用机器的主机名。 cat /etc/hostname 获得主机名 tinygeek

<pre>
首先为每一个服务器创建数据目录，选择端口
mkdir -p ~/dbs/node1 ~/dbs/node2

还要发挥自己的聪明才智，给副本集启个名字。名字是为了易于与别的副本集区分，也是为了方便地将整个集合视为一个整体。本demo起名"cluster001"，是不是巨恶！

然后就可以启动服务器了。

主机名叫tinygeek，先启动节点1，在10001端口
sudo mongod --dbpath ~/dbs/node1 --port 10001 --replSet cluster001/tinygeek:10002
再启动节点2，在10002端口
sudo mongod --dbpath ~/dbs/node2 --port 10002 --replSet cluster001/tinygeek:10001
如果，人家还想添加更多的节点，怎么破？
sudo mongod --dbpath ~/dbs/node3 --port 10003 --replSet cluster001/tinygeek:10001 或者
sudo mongod --dbpath ~/dbs/node3 --port 10003 --replSet cluster001/tinygeek:10001，tinygeek:10002 都可以滴！

副本集的一个亮点就是有自检测功能，在其中指定的单台服务器后，MongoDB就会自动搜索并连接其余节点！

最后，还要在shell中初始化副本集。

在shell中，连接其中一个服务器，初始化命令只能执行一次：

sudo mongo tinygeek:10001/admin

</pre>

**待续**


