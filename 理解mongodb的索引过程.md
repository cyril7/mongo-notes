理解mongodb索引过程
====

Index能够有效的支持mongodb中的查询操作。如果没有索引，那么mongodb将从集合中扫描每个文档，并返回满足查询条件的文档。这样就比较低效了，需要扫描比索引多的多的数据。

mongodb采用B树这种结构进行索引，B树能够有效的支持等值查询和范围查询。和其它数据库的索引类似，mongodb以集合为索引的建立对象，可以建立在文档的域或者子域上。

例如，如果存在对集合A中的age域的索引，那么查询集合A中年龄小于30的对象的操作就可以只操作age域的值在30以内的文档，可以有效的缩小需要查询的文档个数。

为什么使用索引进行查询会比较快呢？从下面两个方面来说明。

**排序方面**->索引本身就维持着某种顺序，如上述提到的针对age域建立索引，那么age的索引就可能按照年龄的大小进行排序。如果查询年龄小于30并且按照年龄递增的顺序生成结果，那么查询下索引就可以轻松的返回满足条件的文档。

```
db.A.find({age:{"lt":30}}).sort({age:1})
```

**直接从索引生成检索结果**->如果查询的全部是索引中的域，那么可以直接生成检索结果。如查询A中所有年龄小于30的年龄值。连根据索引去集合中查找相关数据的动作也省略了，能不快吗？！

##索引类型##

###Default _id###

>默认的\_id索引，唯一性索引，用来区分不同的文档。如果用户没有指定，那么系统会创建并指定_id的值。实质上就相当于集合的主键，不能删除。

###Single Field###

>针对单独域的索引，mongodb支持对集合中任何一个域建立索引。不管域的类型是单独域，子域或者是子文档类型。

例如针对如下case

<pre>
{ "_id" : ObjectId(...),
  "name" : "Alice"
  "age" : 27
}
</pre>

可以在name字段上创建索引，并按照名字字母序排列

<pre>
db.friends.ensureIndex( { "name" : 1 } )
</pre>

那么可以对子域(Embedded Fields)建立索引吗？

当然可以，和对域建立索引类似。例如case

<pre>
{"_id": ObjectId(...)
 "name": "John Doe"
 "address": {
 	"street": "Main",
	"zipcode": "53511",
    "state": "WI"
	}
}
</pre>

如果对查询address域中的zipcode子域有需求，就可以建立如下索引

<pre>
db.people.ensureIndex({"address.zipcode":1})
</pre>

那么也能对子文档(Subdocuments)建立索引了，如factories集合的文档中包含metro的域，如

<pre>
{
  _id: ObjectId(...),
  metro: {
           city: "New York",
           state: "NY"
         },
  name: "Giant Factory"
}
</pre>

metro域就是一个subdocument，包含子域city和state，可以在metro域上建立索引

<pre>
db.factories.ensureIndex({metro:1})
</pre>

那么就可以愉快的使用该索引进行如下查询

<pre>
db.factories.find( { metro: { city: "New York", state: "NY" } } )
</pre>

**注意**，如果进行上述在域上的值查询，那么子域的顺序必须一致，如下面的查询则找不到相应的结果

<pre>
db.factories.find( { metro: { state: "NY", city: "New York" } } )
</pre>

###Compound Index###

>mongodb支持组合索引，即一个索引可以建立在多个域上。

针对以下格式的文档，如果需要在item和stock字段进行查询，那么可以建立组合索引。

<pre>
{
 "_id": ObjectId(...),
 "item": "Banana",
 "category": ["food", "produce", "grocery"],
 "location": "4th Street Store",
 "stock": 4,
 "type": "cases",
 "arrival": Date(...)
}
</pre>

该索引也可以满足单独针对item或者stock域的查询需求。

<pre>
db.products.ensureIndex( { "item": 1, "stock": 1 } )
</pre>

>注意，如果一个域上已经建立了hash索引，那么就不能在这个域上再建立组合索引了。

另外，在组合索引中，还需要注意指定域的内容的顺序。在mongodb中，1表示升序，-1表示降序。在单独域的索引中，顺序的方法不太重要，因为mongo可以自行选择方向，但是组合索引中，域的顺序可以影响索引支持的排序操作！

如以下两种查询需求，查询1->按username升序，date降序排列，查询2->按照username降序，date升序排序。

<pre>
查询1：db.events.find().sort( { username: 1, date: -1 } )
查询2：db.events.find().sort( { username: -1, date: 1 } )
</pre>

那么只要建立一个索引即可满足上述两种查询的需求：

<pre>
db.events.ensureIndex( { "username" : 1, "date" : -1 } )
</pre>

但是，上述索引不能满足按username升序，date升序的排序需求。

<pre>
db.events.find().sort( { username: 1, date: 1 } )
</pre>

**索引的前缀性质**是指，如有如下索引：

<pre>
{ "item": 1, "location": 1, "stock": 1 }
</pre>

那么针对该索引支持的查询有：

- item
- item，location
- item，location，stock
- item，stock。这种情况支持的不是很有效，没有针对item和stock的组合索引有效

该索引不支持的查询有：

- location
- stock
- location，stock

###Multikey Index###

>针对数组类型的域建立索引，Mongodb会为数组的每个元素分别建立索引。该类型索引可以针对数组的中某个元素或某些元素满足查询条件的情况。Mongodb会自动根据数组域建立Multikey索引，无需用户指定。

数组类型分为两类，如基本数组类型，和子文档数组类型。

>针对基本数据类型

<pre>
{
  "_id" : ObjectId("..."),
  "name" : "Warm Weather",
  "author" : "Steve",
  "tags" : [ "weather", "hot", "record", "april" ]
}
</pre>

那么针对tags的多值索引，会包括4单独的实体，weather,hot,record，april

这种索引会满足如下查询需求：

<pre>
db.demo.find({'tags':'weather'})
</pre>

>子文档数组类型的多值索引

那如果数组的元素是子文档类型的，那么多值索引依然可用。

<pre>
{
 "_id": ObjectId(...),
 "title": "Grocery Quality",
 "comments": [
    { author_id: ObjectId(...),
      date: Date(...),
      text: "Please expand the cheddar selection." },
    { author_id: ObjectId(...),
      date: Date(...),
      text: "Please expand the mustard selection." },
    { author_id: ObjectId(...),
      date: Date(...),
      text: "Please expand the olive selection." }
 ]
}
</pre>

可以在comments.text上建立索引，即可满足以下查询：

<pre>
db.feedback.find( { "comments.text": "Please expand the olive selection." } )
</pre>

###geospatial Index###

###text indexes###

>全文索引，索引中停用词被过滤掉了，并将集合中的词提取，只存储词根

一个集合只能有一个全文索引，全文索引建立在域的值为String类型，或者域的类型为String数组。建立方式如下：

<pre>
db.reviews.ensureIndex( { comments: "text" } )
</pre>

###Hashed Indexes###

只支持相等匹配，不支持范围匹配

##索引属性##

- TTL索引 ->指定有效时间的集合，如session信息等，这些在存储在数据库中的数据只需存活一段时间，即可被mongodb主动删除。
- 唯一性索引
- Sparse indexes->如果文档在索引域上的值为空，则索引会跳过这些文档

例如有以下集合demo

<pre>
{ "_id" : ObjectId("523b6e32fb408eea0eec2647"), "userid" : "newbie" }
{ "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }
{ "_id" : ObjectId("523b6e6ffb408eea0eec2649"), "userid" : "nina", "score" : 90 }
</pre>

如果针对demo建立稀疏索引，

<pre>
db.scores.ensureIndex( { score: 1 } , { sparse: true } 
</pre>

那么针对该索引，进行查询年龄小于90的对象，

<pre>
db.scores.find( { score: { $lt: 90 } } )

结果不包含score值为null的对象：
{ "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }
</pre>

但是如果执行如下查询，按得分递减排序对象，可以看到也包含了得分为null的对象
<pre>
db.demo.find().sort({score:-1})

结果表明在进行全部查询时，虽然排序的时候指定了域，并且域也建立了稀疏索引，但结果中依然全部包括。
{ "_id" : ObjectId("523b6e6ffb408eea0eec2649"), "userid" : "nina", "score" : 90 }
{ "_id" : ObjectId("523b6e61fb408eea0eec2648"), "userid" : "abby", "score" : 82 }
{ "_id" : ObjectId("523b6e32fb408eea0eec2647"), "userid" : "newbie" }

如果想使用稀疏索引的话，可以这样做
db.demo.find().sort({score:-1}).hint({score:1})
</pre>

现在增加unique索引约束，如

<pre>
db.demo.ensureIndex({score:1},{sparse:true,unique:true})

那么在上例中，再进行插入以下数据
db.scores.insert( { "userid": "AAAAAAA", "score": 43 } )
db.scores.insert( { "userid": "BBBBBBB", "score": 34 } )
db.scores.insert( { "userid": "CCCCCCC" } )
db.scores.insert( { "userid": "DDDDDDD" } )

但是，以下操作失败了
db.scores.insert( { "userid": "AAAAAAA", "score": 82 } )//因为score中已经有了82,90了
db.scores.insert( { "userid": "BBBBBBB", "score": 90 } )
</pre>

##创建索引##

默认情况下，在某个数据库的集合上建立索引时，那么针对该数据库的其他集合上的操作也被阻塞了。

关于索引的名字
<pre>
db.products.ensureIndex( { item: 1, quantity: -1 } )

按照上述命令建立索引的名字为item_1_quantity_-1，也可以指定名字

db.products.ensureIndex( { item: 1, quantity: -1},{name,"inventory"})
</pre>

获取集合上的索引可以使用如下函数：

<pre>
db.collections.getIndexes()
</pre>

##索引管理##

单独域
>db.collections.ensureIndex({field:1})

组合域
>db.collections.ensureIndex({field1:1,field2:-1,field3:-1})

唯一索引
>db.collections.ensureIndex({field:1},{unique:true})

稀疏索引
>db.collections.ensureIndex({field,1},{spare:true})

hash索引
>db.collections.ensureIndex({_id:"hashed"})

后台创建索引
>db.collection.ensureIndex({a:1},{background:true})

删除指定的索引,如欲删除tax-id域上的升序索引
>db.collection.dropIndex({"tax-id",1})

删除所有索引
>db.collection.dropIndexes()

修改一个索引
<pre>
如之前创建了一个索引
db.orders.ensureIndex({"cust_id":1,"ord_date":-1,"items":1},{unique:true})
现在想去除掉unique的约束，尝试如下修改出错
db.orders.ensureIndex({"cust_id":1,"ord_date":-1,"items":1})
需要先删除原先的index，然后再新建（ps。这还有你告诉我）
db.orders.dropIndex({"cust_id":1,"ord_date":-1,"items":1})
db.orders.ensureIndex({"cust_id":1,"ord_date":-1,"items":1})
</pre>

查询当前collection的所有索引
>db.collection.getIndexes()

查询当前数据库的所有索引
>db.system.indexes.find()

