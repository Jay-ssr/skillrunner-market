# Vastbase

- **NuGet**: `SqlSugarCore`（需要升级到 5.1.4.111-preview06 及以上）

- **DbType**: `DbType.Vastbase`

## 连接字符串
```csharp
PORT=5432;DATABASE=SqlSugar4xTest;HOST=localhost;PASSWORD=haosql;USER ID=postgres;No Reset On Close=true
//架构（schema）
字符串加上 searchpath=架构
```

## 初始化
```csharp
var db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = SqlSugar.DbType.Vastbase,
    ConnectionString = Config.ConnectionString,
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        DisableNarvchar = true  //有些版本需要禁用Nvarchar
    }
});
```

## 配置 Vastbase 支持 .NET
在 .NET 中修改加密方式：
1. `password_encryption_type=1` 同时支持 sha256 和 md5 多重验证
2. 重启服务
3. 重新修改密码

## 数据类型和 C# 对照表
| Vastbase | C# |
|----------|-----|
| numeric | decimal |
| int2 | short |
| int1 | byte |
| int4 | int |
| int8 | long |
| boolean | boolean |
| float4 | float |
| float8 | double |
| TEXT | 大文本 |
| json | C#可序列化的对象（特性要加 IsJson=true） |
| timestamp | DateTime |
| date | DateTime |
| time | TimeSpan |
| uuid | Guid |

## 禁用自动转小写
```csharp
MoreSettings = new ConnMoreSettings()
{
    PgSqlIsAutoToLower = false,
    PgSqlIsAutoToLowerCodeFirst = false
}
```
