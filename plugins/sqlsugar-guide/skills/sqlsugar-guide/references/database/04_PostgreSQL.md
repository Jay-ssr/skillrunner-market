# PostgreSQL

- **NuGet**: `SqlSugarCore`

- **DbType**: `DbType.PostgreSQL`

## 连接字符串
```csharp
PORT=5432;DATABASE=SqlSugar4xTest;HOST=localhost;PASSWORD=haosql;USER ID=postgres
//支持schema用法：searchpath=架构名
```

## 初始化
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.PostgreSQL,
    ConnectionString = "PORT=5432;DATABASE=SqlSugar4xTest;HOST=localhost;PASSWORD=haosql;USER ID=postgres",
    IsAutoCloseConnection = true
});
```

## 注意事项

### 1. 不规范表和规范表

规范（小写表）推荐写法：
```csharp
//自动将 StudentName 变成 student_name
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.PostgreSQL,
    ConnectionString = Config.ConnectionString3,
    IsAutoCloseConnection = true,
    ConfigureExternalServices = new ConfigureExternalServices()
    {
        EntityService = (x, p) =>
        {
            p.DbColumnName = UtilMethods.ToUnderLine(p.DbColumnName);  //驼峰转下划线
        },
        EntityNameService = (x, p) =>
        {
            p.DbTableName = UtilMethods.ToUnderLine(p.DbTableName);
        }
    }
});
```

不规范（不自动转小写）：
```csharp
var db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = SqlSugar.DbType.PostgreSQL,
    ConnectionString = Config.ConnectionString,
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        PgSqlIsAutoToLower = false,               //增删查改支持驼峰表
        PgSqlIsAutoToLowerCodeFirst = false       //建表建驼峰表
    }
});
```

### 2. JSON 类型

```csharp
[SugarColumn(IsJson = true)]  //低版本要加 ColumnDataType = "json"
public List<Order> JsonText { get; set; }
```

### 3. 数组类型

```csharp
[SugarColumn(ColumnDataType = "text []", IsArray = true)]
public string[] MenuIds { get; set; }

//数组函数：SqlSugarCore 5.1.4.158-preview15+
var x = Db.Queryable<UnitArrayLongtest1>()
    .Where(it => SqlFunc.PgsqlArrayContains(it.ids, 1))  //数组包含
    .ToList();
```

### 4. Geometry / PostGIS

Nuget 安装 Npgsql.NetTopologySuite（7.0以下版本）和 SqlSugarCore

用法1：
```csharp
SqlSugarClient Db = new SqlSugarClient(new ConnectionConfig()
{
    ConnectionString = "连接符字串",
    DbType = DbType.SqlServer,
    IsAutoCloseConnection = true
}, db => {
    NpgsqlConnection.GlobalTypeMapper.UseNetTopologySuite();
});

//实体
[SugarColumn(ColumnName = "geom")]
public Geometry geom { get; set; }
```

用法2：
```csharp
using (db.Ado.OpenAlways())
{
    var conn = (db.Ado.Connection as NpgsqlConnection);
    conn.TypeMapper.UseNetTopologySuite();
    var list = db.Queryable<xx>().ToList();
}
```

### 5. 时间异常处理

错误：`Cannot write DateTime with Kind=Unspecified to PostgreSQL type 'timestamp with time zone'`

ORM 默认配置如下：
```csharp
if (StaticConfig.AppContext_ConvertInfinityDateTime == false)
{
    AppContext.SetSwitch("Npgsql.EnableLegacyTimestampBehavior", true);
    AppContext.SetSwitch("Npgsql.DisableDateTimeInfinityConversions", true);
}

//如果不想用可以把
StaticConfig.AppContext_ConvertInfinityDateTime = true;  //这样就不走ORM默认
```

### 6. 架构的支持 Schema

连接字符串上加上 `searchpath=架构名`，可以支持多架构（5.0.9.1版本支持）

### 7. 自增配置

```csharp
MoreSettings = new ConnMoreSettings()
{
    //可以配置自增的方式（Identity需要新版本PG，默认是序列实现）
    PostgresIdentityStrategy = PostgresIdentityStrategy.Identity
}
```

### 8. 自定义修改列方法

AOP 修改 SQL 示例：
```csharp
db.Aop.OnExecutingChangeSql = (s, p) =>
{
    if (s.StartsWith("alter table ") && s.Contains("  type ") && !s.Contains(" USING "))
    {
        var type = s.Split("  type ").Last();
        var columnName = s.Split(" ALTER COLUMN ").Last().Split(" ").First();
        s = $"{s} USING {columnName}::{type}";
    }
    return new KeyValuePair<string, SugarParameter[]>(s, p);
};
```

### 9. ILike 不区分大小写

```csharp
MoreSettings = new ConnMoreSettings()
{
    EnableILike = true
}
```

### 10. jsonb 列

用对象/集合类型 + `IsJson = true`，ORM 自动序列化/反序列化：

```csharp
[SugarColumn(IsJson = true)]
public WifiConfig? WifiInfo { get; set; }

[SugarColumn(IsJson = true)]
public List<CyclicConfigItem>? CyclicConfig { get; set; }
```

> ⚠️ **属性类型用对象，不要用 `string`。** `IsJson = true` 会调 `SerializeObject`：对象类型第一次序列化得到 JSON 文本 ✅；`string` 类型本身已是文本，再序列化多一层引号 ❌。且 PG 拒绝 `text → jsonb` 隐式转换。

### 11. 建表更换自增列的方式

默认 Serial 方式实现自增（兼容低版本 PGSQL），Identity 只支持较高版本 PGSQL：
```csharp
MoreSettings = new ConnMoreSettings()
{
    PostgresIdentityStrategy = PostgresIdentityStrategy.Identity
}
```
