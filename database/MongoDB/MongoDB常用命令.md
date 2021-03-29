[toc]

# 数据库连接
标准 URI 连接语法：
```
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```
- mongodb:// 这是固定的格式，必须要指定。
- 
- username:password@ 可选项，如果设置，在连接数据库服务器之后，驱动都会尝试登录这个数据库
- 
- host1 必须的指定至少一个host, host1 是这个URI唯一要填写的。它指定了要连接服务器的地址。如果要连接复制集，请指定多个主机地址。
- 
- portX 可选的指定端口，如果不填，默认为27017
- 
- /database 如果指定username:password@，连接并验证登录指定数据库。若不指定，默认打开 test 数据库。
- 
- ?options 是连接选项。如果不使用/database，则前面需要加上/。所有连接选项都是键值对name=value，键值对之间通过&或;（分号）隔开

# 命令操作
## 数据库操作

```
//显示所有数据库
show dbs

//显示当前数据库
db

//切换到test数据库，如果不存在则创建，创建后需要插入数据，才能show dbs中显示
use test

//删除数据库
db.dropDatabase()
```

## 集合操作

```
//查看当前数据库的集合
show collections

//创建集合
db.createCollection(name, options)
//eg 创建固定集合 mycol，整个集合空间大小 6142800 KB, 文档最大个数为 10000 个。
db.createCollection("mycol", { capped : true, autoIndexId : true, size : 6142800, max : 10000 } )

//删除集合
db.collection.drop()
//eg 删除集合[mycol2]
db.mycol2.drop()
```

## 文档操作
### 添加
```
//添加文档
db.COLLECTION_NAME.insert(document)

//eg 以下文档可以存储在 MongoDB 的 runoob 数据库 的 col 集合中：
db.col.insert({title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})

db.collection.insertOne():向指定集合中插入一条文档数据
db.collection.insertMany():向指定集合中插入多条文档数据

#  插入单条数据

> var document = db.collection.insertOne({"a": 3})
> document
{
        "acknowledged" : true,
        "insertedId" : ObjectId("571a218011a82a1d94c02333")
}

#  插入多条数据
> var res = db.collection.insertMany([{"b": 3}, {'c': 4}])
> res
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("571a22a911a82a1d94c02337"),
                ObjectId("571a22a911a82a1d94c02338")
        ]
}
```
### 更新

```
db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}})


//只更新第一条记录：
db.col.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );

//全部更新：
db.col.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );

//只添加第一条：
db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );

//全部添加进去:
db.col.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );

//全部更新：
db.col.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );

//只更新第一条记录：
db.col.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );

```

### 删除

```
db.col.remove({'title':'MongoDB 教程'})

//删除第一条找到的记录可以设置 justOne 为 1
db.COLLECTION_NAME.remove(DELETION_CRITERIA,1)

//删除所有数据
db.col.remove({})
```

### 查询

```
//以易读的方式来读取数据
db.col.find().pretty()

//多条件查询 （and）
db.col.find({"by":"菜鸟教程", "title":"MongoDB 教程"}).pretty()

//多条件查询 （or）
db.col.find({$or:[{"by":"菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()

//多条件查询 （and 和 or）
db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
```
