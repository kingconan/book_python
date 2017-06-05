# Mongo DB CheatSheet

```js
db.CollectionName.count() //显示当前集合里item数量
db.CollectionName.find() //显示当前集合里所有元素
db.CollectionName.find(<query>,<projection>).sort(<sort order>).limit(<n>) //查找前<n>个并排序
//query examples
{ age : { $gt : 18 } },
{ 
```



