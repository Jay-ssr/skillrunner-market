# GBase

- **NuGet**: `SqlSugar.GBaseCore` + `SqlSugarCore`

- **DbType**: `DbType.GBase`

## 连接字符串
```csharp
//新版本
Host=localhost;Service=19088;Server=gbase01;Database=testdb;Protocol=onsoctcp;
Uid=gbasedbt;Pwd=GBase123;Db_locale=zh_CN.utf8;Client_locale=zh_CN.utf8

//老版本
Driver={GBase ODBC DRIVER (64-Bit)};Host=localhost;Service=19088;Server=gbase01;Database=testdb;
Protocol=onsoctcp;Uid=gbasedbt;Pwd=GBase123;Db_locale=zh_CN.utf8;Client_locale=zh_CN.utf8
```

## 初始化
```csharp
//程序启动时只执行一次
InstanceFactory.CustomAssemblies = new System.Reflection.Assembly[] {
    typeof(GBaseProvider).Assembly
};

SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    ConnectionString = "Host=localhost;Service=19088;Server=gbase01;Database=testdb;Protocol=onsoctcp;Uid=gbasedbt;Pwd=GBase123;Db_locale=zh_CN.utf8;Client_locale=zh_CN.utf8",
    DbType = DbType.GBase,
    IsAutoCloseConnection = true
});
```

## 注意事项
1. 大数据插入方式：`db.Insertable(updateObjs).PageSize(50).ExecuteCommand();`
2. Long 类型注意：升级最新版本
3. 时间类型：使用 `DATETIME YEAR TO FRACTION(5)` 类型
4. `Transaction not available`：建库语句加 `with log` 启用事务
