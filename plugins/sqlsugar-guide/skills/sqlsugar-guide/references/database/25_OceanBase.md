# OceanBase

- **NuGet**: `SqlSugarCore`（MySQL 模式）或 `SqlSugar.OceanBaseForOracleCore`（Oracle 模式）

## DbType
- MySQL 模式: `DbType.OceanBase`
- Oracle 模式: `DbType.OceanBaseForOracle`

#### MySQL 模式

## 连接字符串
```csharp
server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosql;
```

## 初始化
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.OceanBase,
    ConnectionString = "server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosql;",
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        DisableNarvchar = true
    }
});
```

特殊服务器连续 2 次写入操作报错，连接字符串加 `Pooling=false`。

## Hints 配置
```csharp
db.Queryable<Order>().Hints("/*+ ... */").ToList();
```

#### Oracle 模式（ODBC）

## 连接字符串
```csharp
Driver={OceanBase ODBC 2.0 Driver};Server=172.19.9.9;Port=2883;Database=XIR_TRD;
User=XIR_TRD@Xpia2C6G#obtest:1650773680;Password=aaAA11%%;Option=3;
```

## 初始化
```csharp
//程序启动时加上
InstanceFactory.CustomAssemblies = new System.Reflection.Assembly[] {
    typeof(OceanBaseForOracleProvider).Assembly
};

SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.OceanBaseForOracle,
    ConnectionString = "Driver={OceanBase ODBC 2.0 Driver};Server=172.19.9.9;Port=2883;Database=XIR_TRD;User=XIR_TRD@Xpia2C6G#obtest:1650773680;Password=aaAA11%%;Option=3;",
    IsAutoCloseConnection = true
});
```

## 注意事项
- ODBC 无法配置 DbType 为 varchar2，数据库字段使用 varchar
