# Sql中间管道

**功能说明**: SQL 中间管道替换 ADO.NET 执行 SQL 的所有代码，例如通过 Http 操作数据库等。

## 对比

| 特性 | AOP | 中间管道 | 扩展库 |
|------|-----|----------|--------|
| 能力 | 处理 SQL、参数、回调 | 替换 ADO 底层全部操作 | 根据规范创建 SQL 支持 DbType 切换 |
| SQL 参数 | 可修改 | 固定，按 DbType 处理 | 按规范生成 |

**使用案例1：将 SQL 传给其他服务器执行**

```csharp
db.CurrentConnectionConfig.SqlMiddle = new SqlMiddle
{
    IsSqlMiddle = true,
    ExecuteCommand = (sql, pars) => {
        return Httpxx.GetJavaServerExecuteCommand(sql, pars);
    },
    GetScalar = (sql, pars) => { ... },
    GetDataSetAll = (sql, pars) => { ... },
    GetDataReader = (sql, pars) => { ... }
};

var dt = db.Insertable(obj).ExecuteCommand();
```

**使用案例2：解析 SQL 去支持其他数据库**

```csharp
db.CurrentConnectionConfig.SqlMiddle = new SqlMiddle
{
    IsSqlMiddle = true,
    ExecuteCommand = (sql, pars) => {
        return GetRedisExecuteCommand(sql, pars);  //根据SQL解析成Redis返回结果
    },
    GetScalar = (sql, pars) => {
        return GetRedisScalar(sql, pars);
    },
    GetDataSetAll = (sql, pars) => {
        return GetRedisDataTable(sql, pars);
    },
    GetDataReader = (sql, pars) => {
        var json = jsonRedisDataReader(sql, pars);
        DataTable dataTable = JsonConvert.DeserializeObject<DataTable>(json);
        return dataTable.CreateDataReader();
    }
};
```
