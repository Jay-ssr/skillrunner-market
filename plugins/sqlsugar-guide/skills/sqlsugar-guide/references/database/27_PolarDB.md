# PolarDB

- **NuGet**: `SqlSugarCore`

- **DbType**: `DbType.PolarDB`

## 连接字符串
```csharp
server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosql;Pooling=false
```

## 初始化
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.PolarDB,
    ConnectionString = "server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosql;",
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        DisableNarvchar = true
    }
});
```
