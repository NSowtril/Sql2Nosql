# sql-2-nosql

本项目利用SQL中提供的查询来构建NoSql查询，目前支持MongoDB。

## Maven

添加依赖： `com.github.NSowtril:sql-2-nosql`. 

```
<dependency>
   <groupId>com.github.NSowtril</groupId>
   <artifactId>sql-2-nosql</artifactId>
</dependency>
```

## 环境要求
- JDK 1.7 及以上

## 从Java运行

```
QueryConverter queryConverter = new QueryConverter.Builder().sqlString("select column1 from my_table where value NOT IN ("theValue1","theValue2","theValue3")").build();
MongoDBQueryHolder mongoDBQueryHolder = queryConverter.getMongoQuery();
String collection = mongoDBQueryHolder.getCollection();
Document query = mongoDBQueryHolder.getQuery();
Document projection = mongoDBQueryHolder.getProjection();
Document sort = mongoDBQueryHolder.getSort();
```

## 作为standalone jar运行

示例：

```
java -jar sql-2-nosql-standalone.jar -s sql.file -d destination.json
```
### 选项

```
用法: com.github.NSowtril.query.mongodb.sql.converter.Main [-s
       <arg> | -sql <arg> | -i]   [-d <arg> | -h <arg>]  [-db <arg>] [-a
       <arg>] [-u <arg>] [-p <arg>] [-b <arg>]
 -s,--sourceFile <arg>        文件
 -sql,--sql <arg>             sql查询语句
 -i,--interactiveMode         交互模式
 -d,--destinationFile <arg>   输出文件。默认输出为System.out
 -h,--host <arg>              主机和端口号格（host:port）默认端口为27017
 -db,--database <arg>         mongo数据库
 -a,--auth database <arg>     auth mongo database
 -u,--username <arg>          用户名
 -p,--password <arg>          密码
 -b,--batchSize <arg>         查询结果的batch大小
```

## 交互模式

```
java -jar target/sql-2-nosql-standalone.jar -i
输入sql:


select object.key1, object2.key3, object1.key4 from my_collection where object.key2 = 34 AND object2.key4 > 5


******结果:*********

db.my_collection.find({
  "$and": [
    {
      "key2": {
        "$numberLong": "34"
      }
    },
    {
      "object2.key4": {
        "$gt": {
          "$numberLong": "5"
        }
      }
    }
  ]
} , {
  "_id": 0,
  "object.key1": 1,
  "object2.key3": 1,
  "object1.key4": 1
})

```

## 可选项

### 日期

```
select * from my_table where date(column,'YYY-MM-DD') >= '2019-12-12'


******结果:*********

db.my_table.find({
  "column": {
    "$gte": {
      "$date": 1452556800000
    }
  }
})
```

### 自然语言日期

```
select * from my_table where date(column,'natural') >= '5000 days ago'


******结果:*********

db.my_table.find({
  "column": {
    "$gte": {
      "$date": 1041700019654
    }
  }
})
```

### 正则表达式

```
select * from my_table where regexMatch(column,'^[ae"gaf]+$')


******结果:*********

db.my_table.find({
  "column": {
    "$regex": "^[ae\"gaf]+$"
  }
})
```

### Distinct

```
select distinct column1 from my_table where value IS NULL


******结果:*********

db.my_table.distinct("column1" , {
  "value": {
    "$exists": false
  }
})
```

### Like

```
select * from my_table where value LIKE 'start%'


******结果:*********

db.my_table.find({
  "value": {
    "$regex": "^start.*$"
  }
})
```

### In

```
select column1 from my_table where value IN ("theValue1","theValue2","theValue3")


******结果:*********

db.my_table.find({ 
	"value" : { 
		"$in" : ["theValue1","theValue2", "theValue3"] 
		}
})
```

### Not In

```
select column1 from my_table where value NOT IN ("theValue1","theValue2","theValue3")


******结果:*********

db.my_table.find({ 
	"value" : { 
		"$nin" : ["theValue1","theValue2", "theValue3"] 
		}
})
```

### Is True

```
select column1 from my_table where column = true


******结果:*********

db.my_table.find({ 
	"column" : true
})
```

### Is False

```
select column1 from my_table where column = false


******结果:*********

db.my_table.find({ 
	"column" : false
})
```

### Not True

```
select column1 from my_table where NOT column


******结果:*********

db.my_table.find({ 
	"value" : {$ne: true}
})
```


### ObjectId Support

```
select column1 from  where OBJECTID('_id') IN ('53102b43bf1044ed8b0ba36b', '54651022bffebc03098b4568')

******结果:*********

db.my_table.find({ 
	"_id" : {$in: [{$oid: "53102b43bf1044ed8b0ba36b"},{$oid: "54651022bffebc03098b4568"}]}
})
```

```
select column1 from  where OBJECTID('_id') = '53102b43bf1044ed8b0ba36b'

******结果:*********

db.my_table.find({ 
	"_id" : {$oid: "53102b43bf1044ed8b0ba36b"}
})
```

### Delete

```
delete from my_table where value IN ("theValue1","theValue2","theValue3")


******结果:*********

3 (number or records deleted)
```

### Group By (聚合函数)

```
select borough, cuisine, count(*) from my_collection WHERE borough LIKE 'Queens%' GROUP BY borough, cuisine ORDER BY count(*) DESC;


******Mongo 查询:*********

db.my_collection.aggregate([{
  "$match": {
    "borough": {
      "$regex": "^Queens.*$"
    }
  }
},{
  "$group": {
    "_id": {
      "borough": "$borough",
      "cuisine": "$cuisine"
    },
    "count": {
      "$sum": 1
    }
  }
},{
  "$sort": {
    "count": -1
  }
},{
  "$project": {
    "borough": "$_id.borough",
    "cuisine": "$_id.cuisine",
    "count": 1,
    "_id": 0
  }
}])
```

### 带子句的聚合函数

```
select Restaurant.cuisine, count(*) from Restaurants group by Restaurant.cuisine having count(*) > 3;


******Mongo 查询:*********

db.Restaurants.aggregate([
                           {
                             "$group": {
                               "_id": "$Restaurant.cuisine",
                               "count": {
                                 "$sum": 1
                               }
                             }
                           },
                           {
                             "$match": {
                               "$expr": {
                                 "$gt": [
                                   "$count",
                                   3
                                 ]
                               }
                             }
                           },
                           {
                             "$project": {
                               "Restaurant.cuisine": "$_id",
                               "count": 1,
                               "_id": 0
                             }
                           }
                         ])
```


### Joins

```
select t1.column1, t2.column2 from my_table as t1 inner join my_table2 as t2 on t1.column = t2.column


******结果:*********

db.my_table.aggregate([
                   {
                     "$match": {}
                   },
                   {
                     "$lookup": {
                       "from": "my_table2",
                       "let": {
                         "column": "$column"
                       },
                       "pipeline": [
                         {
                           "$match": {
                             "$expr": {
                               "$eq": [
                                 "$$column",
                                 "$column"
                               ]
                             }
                           }
                         }
                       ],
                       "as": "t2"
                     }
                   },
                   {
                     "$unwind": {
                       "path": "$t2",
                       "preserveNullAndEmptyArrays": false
                     }
                   },
                   {
                     "$project": {
                       "_id": 0,
                       "column1": 1,
                       "t2.column2": 1
                     }
                   }
                 ])


or

select t1.Column1, t2.Column2 from my_table as t1 inner join my_table2 as t2 on t1.nested1.Column = t2.nested2.Column inner join my_table3 as t3 on t1.nested1.Column = t3.nested3.Column where t1.nested1.whereColumn1 = "whereValue1" and t2.nested2.whereColumn2 = "whereValue2" and t3.nested3.whereColumn3 = "whereValue3"


******结果:*********

db.my_table.aggregate([
                        {
                          "$match": {
                            "nested1.whereColumn1": "whereValue1"
                          }
                        },
                        {
                          "$lookup": {
                            "from": "my_table2",
                            "let": {
                              "nested1_column": "$nested1.Column"
                            },
                            "pipeline": [
                              {
                                "$match": {
                                  "$and": [
                                    {
                                      "$expr": {
                                        "$eq": [
                                          "$$nested1_column",
                                          "$nested2.Column"
                                        ]
                                      }
                                    },
                                    {
                                      "nested2.whereColumn2": "whereValue2"
                                    }
                                  ]
                                }
                              }
                            ],
                            "as": "t2"
                          }
                        },
                        {
                          "$unwind": {
                            "path": "$t2",
                            "preserveNullAndEmptyArrays": false
                          }
                        },
                        {
                          "$lookup": {
                            "from": "my_table3",
                            "let": {
                              "nested1_column": "$nested1.Column"
                            },
                            "pipeline": [
                              {
                                "$match": {
                                  "$and": [
                                    {
                                      "$expr": {
                                        "$eq": [
                                          "$$nested1_column",
                                          "$nested3.Column"
                                        ]
                                      }
                                    },
                                    {
                                      "nested3.whereColumn3": "whereValue3"
                                    }
                                  ]
                                }
                              }
                            ],
                            "as": "t3"
                          }
                        },
                        {
                          "$unwind": {
                            "path": "$t3",
                            "preserveNullAndEmptyArrays": false
                          }
                        },
                        {
                          "$project": {
                            "_id": 0,
                            "c1": "$Column1",
                            "c2": "$t2.Column2",
                            "c3": "$t3.Column3"
                          }
                        }
                      ])


```


### 别名

```
select object.key1 as key1, object2.key3 as key3, object1.key4 as key4 from my_collection where object.key2 = 34 AND object2.key4 > 5;


******Mongo Query:*********

db.Restaurants.aggregate([{
  "$match": {
    "$and": [
      {
        "Restaurant.cuisine": "American"
      },
      {
        "Restaurant.borough": {
          "$gt": "N"
        }
      }
    ]
  }
},{
  "$project": {
    "_id": 0,
    "key1": "$Restaurant.borough",
    "key3": "$Restaurant.cuisine",
    "key4": "$Restaurant.address.zipcode"
  }
}])
```

### 别名 Group By (Aggregation)

```
select borough as b, cuisine as c, count(*) as co from my_collection WHERE borough LIKE 'Queens%' GROUP BY borough, cuisine ORDER BY count(*) DESC;


******Mongo 查询:*********

db.my_collection.aggregate([{
  "$match": {
    "borough": {
      "$regex": "^Queens.*$"
    }
  }
},{
  "$group": {
    "_id": {
      "borough": "$borough",
      "cuisine": "$cuisine"
    },
    "co": {
      "$sum": 1
    }
  }
},{
  "$sort": {
    "co": -1
  }
},{
  "$project": {
    "b": "$_id.borough",
    "c": "$_id.cuisine",
    "co": 1,
    "_id": 0
  }
}])
```

### Offset

```
select * from table limit 3 offset 4
or
select a, count(*) from table group by a limit 3 offset 4


******结果:*********

is equivalent to the $skip function in mongodb json query language
```

### 直接集成Mongo

可以在一个真正的mongodb数据库上执行查询并查看结果。

```
java -jar target/sql-2-sql-standalone.jar -i -h localhost:27017 -db hello 
输入sql:


select borough, cuisine, count(*) from my_collection GROUP BY borough, cuisine ORDER BY count(*) DESC;


******查询结果:*********

[{
	"borough" : "Manhattan",
	"cuisine" : "American ",
	"count" : 3205
},{
	"borough" : "Brooklyn",
	"cuisine" : "American ",
	"count" : 1273
},{
	"borough" : "Queens",
	"cuisine" : "American ",
	"count" : 1040
},{
	"borough" : "Brooklyn",
	"cuisine" : "Chinese",
	"count" : 763
},{
	"borough" : "Queens",
	"cuisine" : "Chinese",
	"count" : 728
}]

more results? (y/n): y
[{
	"borough" : "Manhattan",
	"cuisine" : "Café/Coffee/Tea",
	"count" : 680
},{
	"borough" : "Manhattan",
	"cuisine" : "Italian",
	"count" : 621
},{
	"borough" : "Manhattan",
	"cuisine" : "Chinese",
	"count" : 510
},{
	"borough" : "Manhattan",
	"cuisine" : "Japanese",
	"count" : 438
},{
	"borough" : "Bronx",
	"cuisine" : "American ",
	"count" : 411
}]

more results? (y/n): n
```
