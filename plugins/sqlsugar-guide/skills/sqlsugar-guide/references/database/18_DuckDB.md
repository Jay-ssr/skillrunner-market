# DuckDB

- **NuGet**: `SqlSugar.DuckDBCore` + `SqlSugarCore`

- **DbType**: `DbType.DuckDB`

## 连接字符串
```csharp
DataSource = train_services.db
```

## 初始化
```csharp
InstanceFactory.CustomAssemblies = new System.Reflection.Assembly[] {
    typeof(SqlSugar.DuckDB.DuckDBProvider).Assembly
};

var db = new SqlSugarClient(new ConnectionConfig()
{
    IsAutoCloseConnection = true,
    DbType = DbType.DuckDB,
    ConnectionString = "DataSource = train_services.db",
    LanguageType = LanguageType.Default
});
```
