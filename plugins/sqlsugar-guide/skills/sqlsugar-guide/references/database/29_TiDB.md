# TiDB

- **NuGet**: `SqlSugarCore`

- **DbType**: `DbType.Tidb`

## 连接字符串
```csharp
server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosql;
```

## 初始化
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.Tidb,
    ConnectionString = "server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosql;",
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        DisableNarvchar = true
    }
});
```

## Hints 配置
```csharp
db.Queryable<Order>().Hints("/*+ ... */").ToList();
```
