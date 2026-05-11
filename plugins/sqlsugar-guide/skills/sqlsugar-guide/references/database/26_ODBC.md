# ODBC

## NuGet
- .NET Core: `SqlSugar.OdbcCore` + `SqlSugarCore`
- .NET Framework: `SqlSugar.Odbc` + `SqlSugar`

- **DbType**: `DbType.Odbc`

## 连接字符串
```csharp
Driver={驱动名};....
```

## 初始化
```csharp
//程序启动时加入
InstanceFactory.CustomAssemblies = new System.Reflection.Assembly[] {
    typeof(OdbcProvider).Assembly
};

SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.Odbc,
    ConnectionString = Config.ConnectionString,
    IsAutoCloseConnection = true
});
```

## 注意事项
1. 不支持 CodeFirst
2. DbFirst 兼容性未知
3. 普通 CRUD 兼容性未知
4. 原生脚本兼容完美

## 设置转译符号
```csharp
OdbcConfig.SqlTranslationLeft = "\"";
OdbcConfig.SqlTranslationRight = "\"";
```

## 设置分页模式
1. Limit 分页（MySQL 等）：默认
2. OFFSET ROWS FETCH 分页：
```csharp
MoreSettings = new ConnMoreSettings()
{
    DatabaseModel = DbType.SqlServer  //强制SQLSERVER分页
}
```
3. RowNumber 分页：
```csharp
OdbcConfig.IsCompatibleWithOldDatabaseVersion = true;
MoreSettings = new ConnMoreSettings()
{
    DatabaseModel = DbType.SqlServer
}
```
