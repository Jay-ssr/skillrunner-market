# GoldenDB

## 说明

用法和 MySQL 一样，唯一区别是要禁用连接池。

- **DbType**: `DbType.MySql`

## 连接字符串
```csharp
server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosql;Pooling=false;
```

## 初始化
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.MySql,
    ConnectionString = "server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosql;Pooling=false;",
    IsAutoCloseConnection = true
});
```
