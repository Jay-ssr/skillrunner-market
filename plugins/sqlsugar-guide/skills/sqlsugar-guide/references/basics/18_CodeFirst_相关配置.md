# CodeFirst 相关配置

以下配置用于控制 Code First 行为。

| 配置项 | 作用 | 常见场景 |
|------|------|----------|
| `EnableCodeFirstUpdatePrecision` | 允许更新 `decimal` / `double` 精度 | 结构变更时需要同步精度 |
| `SqlServerCodeFirstNvarchar` | SQL Server 建表时字符串默认使用 `NVARCHAR` | 需要贴近 SQL Server 习惯 |
| `PgSqlIsAutoToLowerCodeFirst` | PostgreSQL 建表时是否自动转小写 | 想保留驼峰表名或字段名 |
| `SqliteCodeFirstEnableDefaultValue` | SQLite Code First 启用默认值 | 需要用特性声明默认值 |
| `SqliteCodeFirstEnableDescription` | SQLite Code First 启用备注 | 需要同步列备注 |
| `SqliteCodeFirstEnableDropColumn` | SQLite Code First 允许删列 | 较新版本、.NET Core 场景 |

典型配置示例：

```csharp
var db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.SqlServer,
    ConnectionString = "...",
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        EnableCodeFirstUpdatePrecision = true,
        SqlServerCodeFirstNvarchar = true,
        PgSqlIsAutoToLowerCodeFirst = false,
        SqliteCodeFirstEnableDefaultValue = true,
        SqliteCodeFirstEnableDescription = true,
        SqliteCodeFirstEnableDropColumn = true
    }
});
```
