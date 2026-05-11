# TDSQL

- **NuGet**: `SqlSugarCore`

#### MySQL 模式

- **DbType**: `DbType.TDSQL`

## 初始化
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    ConnectionString = Config.ConnectionString,
    DbType = DbType.TDSQL,
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        DisableNarvchar = true
    }
});
```

#### PGSQL 模式（ODBC 连接）

- **DbType**: `DbType.TDSQLForPGODBC`

## 初始化
```csharp
//程序启动时注册DLL
InstanceFactory.CustomAssemblies = new System.Reflection.Assembly[] {
    typeof(SqlSugar.TDSQLForPGODBC.TDSQLForPGODBCAdapter).Assembly
};

SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    ConnectionString = Config.ConnectionString,
    DbType = DbType.TDSQLForPGODBC,
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        PgSqlIsAutoToLower = false,
        PgSqlIsAutoToLowerCodeFirst = false
    }
});
```
