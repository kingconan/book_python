# Mongo DB CheatSheet

```js
> db.CollectionName.count() //显示当前集合里item数量
> db.CollectionName.find() //显示当前集合里所有元素
> db.CollectionName.find(<query>,<projection>).sort(<sort order>).limit(<n>) //查找前<n>个并排序
//例子，年龄大于18，返回结果只显示name和attr_2两个属性
{ age : { $gt : 18 } } , { name : 1, attr_2 : 1 }


> db.UserItem.find({},{follower_count:1,name:1}).sort({follower_count:-1}).limit(5);
{ "_id" : ObjectId("5931452190bd4b4e16868c3f"), "name" : "张佳玮", "follower_count" : 1350605 }
{ "_id" : ObjectId("5931452390bd4b4e16868c59"), "name" : "李开复", "follower_count" : 1006584 }
{ "_id" : ObjectId("5931453690bd4b4e16868cb2"), "name" : "黄继新", "follower_count" : 805432 }
{ "_id" : ObjectId("5931452390bd4b4e16868c57"), "name" : "周源", "follower_count" : 773221 }
{ "_id" : ObjectId("5931462690bd4b4e16868ebf"), "name" : "yolfilm", "follower_count" : 757138 }
```



