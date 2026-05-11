# Json 2 SQL

通过 JSON 结构生成 SQL。

## 初始化 JORM 对象（5.0.9.2）

```csharp
JsonClient jsonToSqlClient = new JsonClient();
jsonToSqlClient.Context = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.MySql,
    IsAutoCloseConnection = true,
    ConnectionString = "server=localhost;Database=db1;Uid=root;Pwd=123"
});
```

## 查询

### 带函数的查询

**Json：**

```json
{
    "Table":"order",
    Select:[  [{SqlFunc_AggregateMin:["id"]},"id"],  [{SqlFunc_GetDate:[]},"Date"]  ]
}
```

**代码：**

```csharp
jsonToSqlClient.Queryable(json).ToSql()

// Sql: SELECT MIN(`id`) AS `id` , NOW() AS `Date` FROM `Order`
```

### 带条件的查询

**Json：**

```json
{
    "Table":"order",
    Where:[  "name","=", "{string}:xxx"  ],
    Select:[  [{SqlFunc_AggregateMin:["id"]},"id"],  [{SqlFunc_GetDate:[]},"Date"]  ]
}
```

**代码：**

```csharp
jsonToSqlClient.Queryable(json).ToSql()

// Sql
// SELECT MIN(`id`) AS `id` , NOW() AS `Date` FROM `Order`   WHERE `name` =  @p0
```

条件查询支持 2 种语法：

#### 语法 1：表格查询语法

可完整支持 SqlSugar 表格查询语法。

```json
Where: [{ "FieldName":"id","ConditionalType":"0","FieldValue":"1"}]
```

#### 语法 2：逗号拼接语法

```json
Where:["name","=","{string}:a" , "&&" , "id" ,">", "{int}:1"]
```

运算符：`">"`, `">="`, `"<"`, `"<="`, `"("`, `")"`, `"="`, `"||"`, `"&&"`, `"&"`, `"|"`, `"null"`, `"is"`, `"isnot"`, `"+"`, `"-"`, `"*"`, `"/"`, `"%"`, `like`

- 字段名：字母、数字、下划线
- 参数值：`{int}:1` 表示类型为 `int`、值为 `1` 的参数
- 函数：`{SqlFunc_AggregateMin:["id"]}` 表示 `min(id)`

### 分页查询

**Json：**

```json
{
    "Table":"order",
    "PageNumber":"1",
    "PageSize":"100"
}
```

**代码：**

```csharp
var sqls=jsonToSqlClient.Queryable(json).ToSqlList()

// SELECT COUNT(1) FROM `Order`
// SELECT * FROM `Order`      LIMIT 0,100
```

### 分组查询

**Json：**

```json
{
    "Table":  "order" ,
    "GroupBy":["name"],
    "Having": [{SqlFunc_AggregateAvg:["id"]},">","{int}:1" ],
    "Select":[  [{SqlFunc_AggregateAvg:["id"]},"id"],"name" ]
}
```

**代码：**

```csharp
var sql= jsonToSqlClient.Queryable(json).ToSql()

// SELECT AVG(`id`) AS `id` , `name` AS `name` FROM `Order`
// GROUP BY  `name`  HAVING AVG(`id`) > @p0
```

### 联表查询

**Json：**

```json
{
    "Table":[  "order","o"],
    "LeftJoin01": ["orderdetail", "d", [  "d.orderid",">","o.id"  ]],
    "Select":["o.id" ,["d.itemid","newitemid"]]
}
// 多表联查追加 LeftJoin02、LeftJoin03...，InnerJoin 同理
```

**代码：**

```csharp
var sql= jsonToSqlClient.Queryable(json).ToSql();

// SELECT `o`.`id` AS `o_id` , `d`.`itemid` AS `newitemid` FROM `Order`
//  o Inner JOIN `orderdetail` d ON `d`.`orderid` > `o`.`id`
```

### 排序

```json
{Table:"order",OrderBy:[{FieldName:"id"},{FieldName:"name",OrderByType:"desc"}]}
```

### 授权查询

该功能持续完善中，保留示例供参考。

```csharp
var tableNames = jsonToSqlClient.GetTableNameList(json); // 通过JSON获取JSON所有表
var configs = GetConfigByUser(tableNames); // 通过表获取行列过滤备注等信息
var sqlList = jsonToSqlClient
    .Queryable(json)
    .UseAuthentication(configs) // 查询启用行列过滤
    .ShowDesciption() // 查询返回备注
    .ToResult();
```

## 插入

### 单条插入

```json
{
    "Table":"order",
    Columns:{name:"{string}:1",price:"{decimal}:1"}
}
// C# jsonToSqlClient.Insertable(json).ToSql()
```

### 批量插入

```json
{
    "Table":"order",
    Columns:[ {name:"{string}:2",price:"{decimal}:2"} ,
        {name:"{string}:1",price:"{decimal}:1"}  ]
}
// C# jsonToSqlClient.Insertable(json).ToSql()
```

### 带自增列

```json
{
    "Table":"order",
    "Identity":"id",
    Columns:  {name:"{string}:2",price:"{decimal}:2"}
}
// C# jsonToSqlClient.Insertable(json).ToSql()
```

## 更新

### 单个对象更新

```json
{
    "Table":"order",
    Columns: { id:"{int}:1" ,name:"{string}:1" },
    WhereColumns:["id"]
}
// C# jsonToSqlClient.Updateable(json).ToSql()
```

### 多个对象更新

```json
{
    "Table":"order",
    Columns:[ {id:2,name:"{string}:2",price:"{decimal}:2"}  ,
        {id:1,name:"{string}:1",price:"{decimal}:1"}  ],
    WhereColumns:["id"]
}
// jsonToSqlClient.Updateable(json).ToSql()
```

### SQL 语句方式更新

```json
{
    "Table":"order",
    Columns: {name:"{string}:2",price:"{decimal}:2"}  ,
    Where:["id","=","{int}:11"]
}
// C# jsonToSqlClient.Updateable(json).ToSql()
```

## 删除

`Where` 用法和查询一致。

**Json：**

```json
{
    "Table":"order",
    Where:[ "id"," = ","{int}:1" ]
}
```

**代码：**

```csharp
jsonToSqlClient.Deleteable(json).ToSqlList()

// DELETE FROM `order` WHERE `id` = @p0
```

## API 说明

| 方法 | 描述 |
|------|------|
| `List<SqlObjectResult> ToSqlList()` | 执行返回多个Sql |
| `SqlObjectResult ToSql()` | 返回单个SQL |
| `List<string> ToSqlString()` | 还未开发 |
| `T ToResult()` | 执行返回结果 |
