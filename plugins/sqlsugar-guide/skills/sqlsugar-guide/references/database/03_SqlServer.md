# SqlServer

- **NuGet**: `SqlSugarCore`（.NET Core）/ `SqlSugar`（.NET Framework）

- **DbType**: `DbType.SqlServer`

## 连接字符串
```csharp
//SqlSugarCore 5.1.4.169上以上版本
server=.;uid=sa;pwd=haosql;database=SQLSUGAR4XTEST;Encrypt=True;TrustServerCertificate=True;
```

## 初始化
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.SqlServer,
    ConnectionString = "server=.;uid=sa;pwd=haosql;database=SQLSUGAR4XTEST;Encrypt=True;TrustServerCertificate=True;",
    IsAutoCloseConnection = true
});
```

## 说明
- 部分版本要加上 `Encrypt=True;TrustServerCertificate=True;`
- .NET 10 出现全球化错误需要配置 `<InvariantGlobalization>false</InvariantGlobalization>`

## 注意事项

### 1. With(NoLock) 语法

SqlServer 默认事务禁止脏读：
```csharp
//单个设置 无锁
db.Queryable<Order>().With(SqlWith.NoLock).ToList();

//全局设置
var db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = SqlSugar.DbType.SqlServer,
    ConnectionString = Config.ConnectionString,
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        IsWithNoLockQuery = true  //全局设置NoLock
    }
});

//全局设置禁用自动Nolock，只针对当前查询禁用
db.Queryable<Order>().With(SqlWith.Null).ToList();
```

### 2. With Index 用法
```csharp
db.Queryable<Order>().With("with(indexname, NoLock)").ToList();
```

### 3. Linux CentOS 注意事项

ORM 默认字符串以 nvarchar 参数传到数据库，特殊情况会影响索引：
```csharp
//1. 通过特性指定类型
[SugarColumn(SqlParameterDbType = System.Data.DbType.AnsiString)]
public string name { get; set; }

//2. 全局操作
MoreSettings = new ConnMoreSettings()
{
    DisableNvarchar = true  //将参数全部转成varchar模式
}

//3. 指定当方法
db.CurrentConnectionConfig.MoreSettings = new MoreSettings() { DisableNvarchar = true };

//4. 指定具体代码
Where(it => it.Name == SqlFunc.ToVarchar("张"));

//5. 原生SQL用法
var name = new SugarParameter("@name", "haha", System.Data.DbType.AnsiString);
```

### 4. Geometry 类型

实体定义：
```csharp
public class UnitGe
{
    [SugarColumn(ColumnDataType = "geometry")]
    public string geometry1 { get; set; }
}

//代码
db.Insertable(new UnitGe() { geometry1 = "POINT (20 180)" }).ExecuteCommand();
var gelist = db.Queryable<UnitGe>().Select(it => new { geometry1 = it.geometry1.ToString() }).ToList();
```

### 5. 存储过程 DataTable 参数（表值参数）
```csharp
var s = new SugarParameter("@p", dt);
s.TypeName = "dtTableName";  //等同于原生SQL
```

### 6. SqlServer 2000

SqlServer 2000 使用 .NET Framework 版本 SqlSugar，低版本需设置 `InitKey = InitKey.Attribute`

### 7. CodeFirst 默认使用 Nvarchar
```csharp
MoreSettings = new ConnMoreSettings()
{
    SqlServerCodeFirstNvarchar = true  //建表字符串默认Nvarchar
}
```

### 8. 指定索引
```csharp
db.Queryable<T>().With("WITH(INDEX(index_name))").ToList();
```

### 9. Schema 用法
```csharp
var db = new SqlSugarClient(new ConnectionConfig()
{
    IsAutoCloseConnection = true,
    DbType = DbType.SqlServer,
    ConnectionString = Connection,
    ConfigureExternalServices = new ConfigureExternalServices()
    {
        EntityNameService = (type, entity) =>
        {
            entity.DbTableName = $"myschema1.{entity.DbTableName}";
        }
    }
});
```

### 10. 连接不上库解决方案

#### 10.1 Linux 下连接 SqlServer 2008 提示 TLS/SSL 握手错误：
- 安装 SqlServer TLS 1.2 补丁包

10.2 `The negotiated TLS 1.0 is an insecure protocol` 错误：
- 安装 TLS 1.2 补丁包或升级 SqlServer

10.3 `certificate chain was issued by an authority that is not trusted` 错误：
- 方案1：在客户端安装目标 SQL Server 的 TLS/SSL 证书
- 方案2（不太安全）：连接字符串设置 `TrustServerCertificate=true`
- 方案3（不安全）：配置 TLS/SSL 设置

10.4 `Only the invariant globalization` 错误：
```xml
<!-- 改启动文件的 .csproj -->
<InvariantGlobalization>false</InvariantGlobalization>
```
