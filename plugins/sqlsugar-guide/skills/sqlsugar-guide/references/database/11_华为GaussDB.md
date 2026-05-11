# 华为GaussDB

## NuGet
- Npgsql 方式：`SqlSugarCore`
- 原生方式：`SqlSugar.GaussDBNativeCore` + `SqlSugarCore`

## DbType
- Npgsql 方式：`DbType.GaussDB`
- 原生方式：`DbType.GaussDBNative`

#### Npgsql 方式连接

## 连接字符串
```csharp
PORT=5432;DATABASE=SqlSugar4xTest;HOST=localhost;PASSWORD=haosql;USER ID=postgres;No Reset On Close=true
```

## 初始化
```csharp
var db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = SqlSugar.DbType.GaussDB,
    ConnectionString = Config.ConnectionString,
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        DisableNarvchar = true
    }
});
```

## 配置 GaussDB 支持 .NET
在 .NET 中修改加密方式：
1. `password_encryption_type=1` 同时支持 sha256 和 md5 多重验证
2. 重启服务
3. 重新修改密码

## 架构用法（Schema）
```csharp
PORT=9210;searchpath=zhnf;DATABASE=db_test;HOST=192.168.1;PASSWORD=abc;USER ID=joe;No Reset On Close=true;
```

## 禁用自动转小写
```csharp
MoreSettings = new ConnMoreSettings()
{
    PgSqlIsAutoToLower = false,           //CRUD不自动转小写
    PgSqlIsAutoToLowerCodeFirst = false   //建表不自动转小写
}
```

## 数据类型和 C# 对照表
| GaussDB | C# |
|---------|-----|
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
| bytes/bytea | Byte[] |

#### 原生方式连接

## 初始化
```csharp
//程序启动时执行
InstanceFactory.CustomAssemblies = new System.Reflection.Assembly[] {
    typeof(SqlSugar.GaussDBCore.GaussDBDataAdapter).Assembly
};

var db = new SqlSugarClient(new ConnectionConfig()
{
    ConnectionString = "PORT=5432;DATABASE=SqlSugar5Demo;HOST=localhost;PASSWORD=postgres;USER ID=postgres",
    DbType = SqlSugar.DbType.GaussDBNative,
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        //GaussDb 配置 GaussDb，OpenGauss 配置 OpenGauss
        DatabaseModel = SqlSugar.DbType.OpenGauss
    }
});
```
