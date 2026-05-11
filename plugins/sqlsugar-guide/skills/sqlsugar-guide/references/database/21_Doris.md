# Doris

- **NuGet**: `SqlSugarCore`

- **DbType**: `DbType.Doris`

## 连接字符串
```csharp
server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosql;Pooling=false
```

## 初始化
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.Doris,
    ConnectionString = "server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosq;Pooling=false",
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        DisableNarvchar = true
    }
});
```

## 负载均衡
```csharp
server=fe1,fe2,fe3;port=9030;database=MP;user=root;password='hjq@123';Pooling=true;LoadBalance=RoundRobin;
```

## 常见错误

`Unsupported command`，连接字符串加上 `Pooling=false`
