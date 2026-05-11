# MongoDB

- **NuGet**: `SqlSugar.MongoDbCore` + `SqlSugarCore`

- **DbType**: `DbType.MongoDb`

## 连接字符串
```csharp
//两种格式都可以
mongodb://root:123456@222.71.212.3:27017/testDB?authSource=admin
host=222.71.212.3;Port=27017;Database=testDB;Username=root;Password=123456;authSource=admin;replicaSet=
```

## 初始化
```csharp
//注册DLL（扔在程序启动时）
InstanceFactory.CustomAssemblies = new System.Reflection.Assembly[] {
    typeof(SqlSugar.MongoDb.MongoDbProvider).Assembly
};

var db = new SqlSugarClient(new ConnectionConfig()
{
    IsAutoCloseConnection = true,
    DbType = DbType.MongoDb,
    ConnectionString = SqlSugarConnectionString
});
```

## 注意事项

## 1. 功能支持
| 支持 | 不支持 |
|------|--------|
| 单表 CRUD、简单联表、分页、排序、简单分组、嵌套文档（JSON）、非对象方式更新/删除、事务、插入或更新 | 子查询、导航查询 |

### 2. 实体类定义

主键定义：继承 MongoDbBase 或自定义：
```csharp
public class Student : MongoDbBase
{
    public string Name { get; set; }
    //外键需要设置ObjectId类型
    [SqlSugar.SugarColumn(ColumnDataType = nameof(ObjectId))]
    public string SchoolId { get; set; }
}

public class MongoDbBase
{
    [SugarColumn(IsPrimaryKey = true, IsOnlyIgnoreInsert = true, ColumnName = "_id")]
    public string Id { get; set; }
}

//支持不同主键类型
public class MongoDbBaseLong  //5.1.4.227+版本
{
    [SqlSugar.SugarColumn(IsPrimaryKey = true, ColumnName = "_id")]
    public long Id { get; set; }
}

public class MongoDbBaseString  //5.1.4.227+版本
{
    [SqlSugar.SugarColumn(IsPrimaryKey = true, ColumnName = "_id")]
    public string Id { get; set; }
}

public class MongoDbBaseGuid  //5.1.4.227+版本
{
    [SqlSugar.SugarColumn(IsPrimaryKey = true, ColumnName = "_id")]
    public Guid Id { get; set; }
}
```

### 3. CRUD 用例

```csharp
//插入
db.Insertable(data).ExecuteCommand();
db.Insertable(data).ExecuteCommandIdentityIntoEntity();
var ids = db.Insertable(data).ExecuteReturnPkList<string>();

//查询
var data2 = db.Queryable<Student>().Where(it => it.Num == 1).ToList();
var count = 0;
var list = db.Queryable<School>().OrderBy(it => it.Name).ToPageList(1, 2, ref count);

//联表
var list = db.Queryable<Student>()
    .LeftJoin<School>((x, y) => x.SchoolId == y.Id)
    .Where((x, y) => y.Name == "TestSchool")
    .Select((x, y) => new
    {
        StudentName = x.Name,
        SchoolName = y.Name
    }).ToList();

//删除
db.Deleteable<Student>().In(ids).ExecuteCommandAsync();

//更新
db.Updateable<OrderInfo>()
    .SetColumns(it => it.Name == "xx")
    .Where(it => it.Id == id)
    .ExecuteCommand();
```

### 4. 嵌套对象

```csharp
//嵌套单个对象
[SugarColumn(IsJson = true)]
public Book Book { get; set; }

public class Book
{
    [BsonRepresentation(BsonType.ObjectId)]
    [SugarColumn(ColumnDataType = nameof(ObjectId))]
    public string SchoolId { get; set; }
    public decimal Price { get; set; }
}

//嵌套List
public class Student : MongoDbBase
{
    public string Name { get; set; }
    [SugarColumn(IsJson = true)]
    public Books Book { get; set; }
}

db.Queryable<Student>().Where(it => it.Books.Any(s => s.Price == 21)).ToList();

//嵌套数组
[SugarColumn(IsJson = true)]
public List<string> Ids { get; set; }
```

### 5. 多 IP 配置

```csharp
var str = "mongodb://root:123456@117.72.212.3:27017,117.72.212.4:27017,117.72.212.5:27017/testDB?authSource=admin&replicaSet=rs0";
```

### 6. 事务

MongoDB 事务需要副本集部署。

### 7. 常见问题

`<hidden>` 错误：把密码中的 `@` 替换为 `%40`

### 8. 批量分页写入

```csharp
//需要数据库支持事务配置
db.Insertable(list).PageSize(50).ExecuteCommand();

//无需数据库支持事务配置
db.Utilities.PageEach(list, 50, pageList => {
    db.Insertable(pageList).ExecuteCommand();
});
```
