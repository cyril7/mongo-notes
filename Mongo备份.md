##Mongodb启动##

如果从/bin/mongod启动mongo，可以指定以下参数：

- dbpath :指定数据目录，当mongod启动时，会在数据目录中创建mongod.lock文件，这个文件用于防止其他mongod进程使用该目录。如果使用同一个数据目录启动另一个Mongodb服务，则会报错。
- port：指定服务器监听的端口号
- fork：以守护进程的方式运行Mongodb，创建服务器进程
- logpath：指定日志的输出路径，而不是输出到命令行，这个命令用于新建或覆盖，如果想用追加，则应该使用logappend选项
- config：指定配置文件

这些参数也可以在/etc/mongod.conf 和mongodb.conf中指定

##监控##

那么如何监控系统的状态和性能呢？好在mongodb有很多功能，使得监控很easy，老板再也不用担心mongo会挂了！

默认情况下，监控的端口比服务的端口号大1000，因此，按照以下url即可

http://服务器ip:28017

打开监控页面了没？如果没有的话，好吧，可能还需要再配置文件中奖相应的参数设置一下。如HTTP 接口

httpinterface=true，另外将 rest=true，重启服务，大功告成！

##备份和修复##

做备份是管理任何数据存储系统的一项非常重要的任务。

首先，就是简单粗暴的停机备份了。

>在mongo中，所有数据都放在数据目录下，也就是说，要想备份MongoDB，只要简单的创建数据目录中所有文件的副本就可以了。but，这个时候，最好先stop掉服务，然后拷贝完数据文件后，再启动服务，防止在拷贝的时候，有数据写入。

有没有高端的，比如不用停机的？当然有了

>神器一mongodump，mongodb自带工具，能够对运行的mongo做查询，然后将查询到的文档写入磁盘。
>神器二mongorestore，获取mongodump的输出结果，并将备份的数据插入到运行的mongodb实例中。

demo实例：

<pre>
mongodump -d backupDemo -o . 将backupDemo数据库备份到当前目录

执行结果显示：
connected to: 127.0.0.1
2014-08-29T16:07:33.948+0800 DATABASE: backupDemo	 to 	./backupDemo
2014-08-29T16:07:33.949+0800 	backupDemo.system.indexes to ./backupDemo/system.indexes.bson
2014-08-29T16:07:33.949+0800 		 1 documents
2014-08-29T16:07:33.949+0800 	backupDemo.demo to ./backupDemo/demo.bson
2014-08-29T16:07:33.950+0800 		 1 documents
2014-08-29T16:07:33.950+0800 	Metadata for backupDemo.demo to ./backupDemo/demo.metadata.json

在当前目录下，生成一个数据文件，一个元数据文件，和一个索引文件。

然后通过mongorestore命令，将刚备份的数据库导入mongo，重命名为foo

mongorestore -d foo --drop ./backupDemo/

执行结果显示：
connected to: 127.0.0.1
2014-08-29T16:12:40.941+0800 ./backupDemo/demo.bson
2014-08-29T16:12:40.941+0800 	going into namespace [foo.demo]
2014-08-29T16:12:40.941+0800 	 dropping
1 objects found
2014-08-29T16:12:40.942+0800 	Creating index: { key: { _id: 1 }, name: "_id_", ns: "foo.demo" }

mongorestore使用-d指定了要恢复的数据库foo，可以进行重命名，--drop指明，恢复前删除foo的集合，否则，数据就会和现有集合的数据合并，会覆盖一些文档。
</pre>

虽然mongodump和mongorestore的组合拳已经很牛逼了，那能不能在不停机备份的同时，也能获取实时数据视图呢？mongodb的fsync能在mongodb运行时复制数据目录还不会损害数据。

fsync命令强制服务器将所有缓冲写入磁盘，还可以选择上锁阻止对数据库的进一步写入，直到释放锁为止。

实例：

<pre>
MongoDB shell version: 2.6.4
connecting to: test
> use admin
switched to db admin
> db
admin
> db.runCommand({'fsync':1,'lock':1})
{
	"info" : "now locked against writes, use db.fsyncUnlock() to unlock",
	"seeAlso" : "http://dochub.mongodb.org/core/fsynccommand",
	"ok" : 1
}
> db.fsyncUnlock()
{ "ok" : 1, "info" : "unlock completed" }
> db.currentOp()//使用这个是为查看当前操作，确保已经解锁了
{ "inprog" : [ ] }
</pre>

