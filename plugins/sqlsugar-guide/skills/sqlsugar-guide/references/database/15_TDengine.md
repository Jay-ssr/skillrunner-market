# TDengine

- **NuGet**: `SqlSugar.TDengineCore` + `SqlSugarCore`

- **DbType**: `DbType.TDengine`

## 连接字符串
```csharp
Host=localhost;Port=6030;Username=root;Password=taosdata;Database=power
```

## 初始化
```csharp
//程序启动时加入
InstanceFactory.CustomAssemblies = new System.Reflection.Assembly[] {
    typeof(TDengineProvider).Assembly
};

var db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = SqlSugar.DbType.TDengine,
    ConnectionString = Config.ConnectionString,
    IsAutoCloseConnection = true
});
```

## 注意事项

### 1. 表模式

小写表（默认）：
```csharp
var db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = SqlSugar.DbType.TDengine,
    ConnectionString = Config.ConnectionString,
    IsAutoCloseConnection = true
});
```

驼峰表：
```csharp
MoreSettings = new ConnMoreSettings()
{
    PgSqlIsAutoToLower = false,
    PgSqlIsAutoToLowerCodeFirst = false
}
```

### 2. TDengine 客户端 SDK 安装

- TDengine-server 服务端（数据库服务器安装）
- TDengine-client 客户端（程序服务器安装）
- Client 版本 >= Server 版本

### 3. 时间精度配置

毫秒（默认）：
```csharp
[SugarColumn(IsPrimaryKey = true, InsertServerTime = true)]
public DateTime ts { get; set; }
```

微秒（连接字符串加上 `TsType=config_us`）：
```csharp
[SugarColumn(IsPrimaryKey = true, SqlParameterDbType = typeof(DateTime16))]
public DateTime ts { get; set; }
```

纳秒（连接字符串加上 `TsType=config_ns`）：
```csharp
[SugarColumn(IsPrimaryKey = true, SqlParameterDbType = typeof(DateTime19))]
public DateTime ts { get; set; }
```

### 4. 执行 SQL

```csharp
//建库
db.Ado.ExecuteCommand("CREATE DATABASE IF NOT EXISTS power WAL_RETENTION_PERIOD 3600");

//建超级表
db.Ado.ExecuteCommand("CREATE STABLE IF NOT EXISTS MyTable (ts TIMESTAMP, current FLOAT, voltage INT, phase FLOAT) TAGS (location BINARY(64), groupId INT)");

//创建子表
db.Ado.ExecuteCommand(@"create table IF NOT EXISTS MyTable01 using MyTable tags('California.SanFrancisco',1)");
```

### 5. 根据 Tag 建表

```csharp
//创建超级表
db.CodeFirst.InitTables<SUsingTagModel>();

//查询所有
var list1 = db.Queryable<SUsingTagModel>().AsTDengineSTable().ToList();

//查询子表A
var tagA = db.Queryable<SUsingTagModel>().AsTDengineSTable().Where(it => it.Tag1 == "a").ToList();

//插入并根据 Tag 的值创建子表
db.Insertable(new List<SUsingTagModel>(){
    new SUsingTagModel() { Boolean = true, Tag1 = "a", Ts = DateTime.Now.AddMilliseconds(1) },
    new SUsingTagModel() { Boolean = true, Tag1 = "b", Ts = DateTime.Now.AddMilliseconds(3) }
})
.SetTDengineChildTableName((stableName, it) => $"{stableName}_{it.Tag1}")
.ExecuteCommand();
```

```csharp
[STableAttribute(STableName = "SUsingTagModel", Tag1 = nameof(Tag1))]
public class SUsingTagModel
{
    [SqlSugar.SugarColumn(IsPrimaryKey = true)]
    public DateTime Ts { get; set; }
    public bool Boolean { get; set; }
    public string Tag1 { get; set; }
}
```

### 6. 子表实体

```csharp
public class MyTable02
{
    [SugarColumn(IsPrimaryKey = true)]
    public DateTime ts { get; set; }
    public string name { get; set; }
    public float phase { get; set; }
    [SugarColumn(IsOnlyIgnoreInsert = true, IsOnlyIgnoreUpdate = true)]  //Tags字段禁止插入
    public string location { get; set; }
    [SugarColumn(IsOnlyIgnoreInsert = true, IsOnlyIgnoreUpdate = true)]
    public int groupId { get; set; }
}
```
