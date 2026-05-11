# QuestDb

- **NuGet**: `SqlSugarCore`

- **DbType**: `DbType.QuestDB`

## 连接字符串
```csharp
host=localhost;port=8812;username=admin;password=quest;database=qdb;ServerCompatibilityMode=NoTypeLoading;
```

## 初始化
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.QuestDB,
    ConnectionString = "host=localhost;port=8812;username=admin;password=quest;database=qdb;ServerCompatibilityMode=NoTypeLoading;",
    IsAutoCloseConnection = true
});
```

## 限制
1. 不支持删操作（只能 truncate table 或删除分区）
2. 避免修改表结构和 truncate/drop 操作，容易锁表

## Symbol 类型
symbol 只能存储重复率很高的值，去重后小于 6 万：
```csharp
[SugarIndex(null, nameof(IndexClass.Name), OrderByType.Asc)]
public class IndexClassTest
{
    public int Id { get; set; }
    //只能是string类型并且 datatype=symbol
    [SugarColumn(ColumnDataType = "symbol")]
    public string Name { get; set; }
}
```

## 创建分表
```csharp
public class SplitTableEntity
{
    public string Id { get; set; }
    public string name { get; set; }
    [TimeDbSplitField(DateType.Day)]  //按天分表
    public DateTime Ts { get; set; }  //只能是时间类型一个字段
}

db.CodeFirst.InitTables<IndexClassTest>();
```

## 高级时间统计 SampleBy
```csharp
var list = db.Queryable<UnitSiafayyy>()
    .SampleBy(1, SampleByUnit.Day)
    .Select(it => new
    {
        Id = SqlFunc.AggregateMin(it.Id),
        Count = SqlFunc.AggregateCount(it.Id)
    })
    .ToList();

//统计后过滤
var list = db.Queryable<UnitSiafayyy>()
    .SampleBy(1, SampleByUnit.Day)
    .Select(it => new
    {
        Id = SqlFunc.AggregateMin(it.Id),
        Count = SqlFunc.AggregateCount(it.Id)
    })
    .MergeTable().Where(it => it.Count > 1)
    .ToList();
```

## Last On（开窗函数）
```csharp
var list = db.Queryable<Users>()
    .PartitionBy(it => new { 时间戳字段, 分组字段1, 分组字段2 }).ToList();

var pageList = db.Queryable<QuestdbTestData3>()
    .PartitionBy(it => new { 时间戳字段, 分组字段1, 分组字段2 })
    .MergeTable()
    .ToPageList(1, 2);
```

## 大数据插入
```csharp
//Nuget安装：SqlSugar.QuestDb.RestAPI
db.RestApi().BulkCopy(list);
db.RestApi().PageSize(50000).BulkCopy(list);

//程序启动时配置
QuestDbRestAPI.HttpPort = 9000;
QuestDbRestAPI.UserName;
QuestDbRestAPI.Password;

//传统写法
db.Insertable(updateObjs).UseParameter().ExecuteCommand();
```

## 常见错误

1. 文件被锁：
   - 重启服务器
   - 避免使用 truncate table 和修改表操作
   - 用 C# 程序插入一条记录解除锁

2. Bool 类型 BUG：QuestDB 8.2.1 版本存在 BUG

3. 插入延迟：WAL 引起的，关闭 WAL：`ALTER TABLE entity SET TYPE WAL;`
