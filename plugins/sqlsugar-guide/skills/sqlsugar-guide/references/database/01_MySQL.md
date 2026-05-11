# MySQL

- **NuGet**: `SqlSugarCore`（.NET Core）/ `SqlSugar`（.NET Framework）

- **DbType**: `DbType.MySql`

## 连接字符串
```csharp
server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosql;
//其他配置
Port=3305          //指定端口号，不指定默认3306
Charset=utf8mb4    //设置编码
AllowLoadLocalInfile=true  //启用bulkcopy
```

## 兼容数据库
`DbType.MySql` 除了支持 MySQL 外，还支持以下数据库：
- OceanBase（MySQL 模式）
- PolarDB
- TiDB
- MariaDB、Percona Server、Amazon Aurora
- Azure Database for MySQL
- Google Cloud SQL for MySQL
- kunDB
- TDSQL、GoldenDB、Doris

## 初始化
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.MySql,
    ConnectionString = "server=localhost;Database=SqlSugar4xTest;Uid=root;Pwd=haosql;",
    IsAutoCloseConnection = true
});
```

## 注意事项

### 1. 禁用 NVarchar

特殊服务器不支持 `N'xx'` 这种 Nvarchar 插入：
```csharp
MoreSettings = new ConnMoreSettings()
{
    DisableNarvchar = true  //禁用Nvarchar
}
```

### 2. Date/Time 错误处理

错误：`Unable to convert MySQL date/time value to System.DateTime`

解决方案（3选1）：
1. 将该字段的缺省值设置为 null，而不是 `0000-00-00` / `0000-00-00 00:00:00`
2. 在连接字符串中添加：`Convert Zero Datetime=True` 或 `Allow Zero Datetime=True`
3. 将该字段设置成字符串类型

### 3. 乱码或表情 emoji 处理

连接字符串配置：
```csharp
CharSet=utf8mb4;
```

ORM 推荐写法：
```csharp
//较新版本正常写就行
//老版本要换成下面写法
db.Insertable(List<实体>).UseParameter().ExecuteCommand()  //5.0.3.8-Preview及以上版本支持

//老版本bulkCopy
db.Fastest<Order>().SetCharacterSet("utf8mb4").BulkCopy(list1x);
```

数据库配置：
```sql
-- 修改database字符集
ALTER DATABASE 数据库名 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci;

-- 修改table字符集
ALTER TABLE 表名 CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

-- 修改column字符集
ALTER TABLE 表名 CHANGE 字段名 字段名 该字段原来的数据类型 CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

### 4. 插入时间带毫秒

版本要求：
- .NET Framework 4.5.2+
- MySql.Data 8.0 以上版本
- .NET Core 2.0+

用法：
```csharp
[SugarColumn(ColumnDataType = "DATETIME(3)")]
public DateTime CreateTime { get; set; }
```

驱动的 `DataReader.GetDateTime()` 不支持查出毫秒，需要处理：
```csharp
Select(it => new
{
    CreateTime = it.CreateTime.ToString()  //转成string就能查出带毫秒的时间
});
```

### 5. 用户自定义 SQL 变量

在连接字符串里面加上 `Allow User Variables=True`

### 6. TinyInt(1) 处理

非 bool 字段使用 `tinyint(1)` 会被驱动解析成 bool，导致计算错误。老项目禁用 tinyint 转 bool：
```
TreatTinyAsBoolean=false  //连接字符串加上
```

### 7. BulkCopy 常见错误

错误：`The used command is not allowed with this MySQL version` 或 `Loading local data is disabled`

解决方案：
1. 添加配置 `AllowLoadLocalInfile=true`
2. 添加配置仍报错，在 MySQL 执行：`SET GLOBAL local_infile=1`

### 8. 连接问题处理

错误：`The given key '23111' was not present in the dictionary`
1. 检查连接字符串是否正确
2. 如果是 MySQL 版本问题，独立安装 MySql.Data 8.0.29 或升级 SqlSugar 到最新版本

### 9. 忽略重复 Insert 用法
```csharp
db.Insertable(insertObj).MySqlIgnore().ExecuteCommand();  //5.1.4.59+
```

### 10. 表和实体大小写不一致

有些用户去表名分大小写，实体不是小写的可以统一转换：
```csharp
var db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = SqlSugar.DbType.MySql,
    ConnectionString = Config.ConnectionString,
    IsAutoCloseConnection = true,
    ConfigureExternalServices = new ConfigureExternalServices
    {
        EntityNameService = (type, entity) =>
        {
            entity.DbTableName = entity.DbTableName?.ToLower();  //转小写
        },
        EntityService = (type, column) =>
        {
            column.DbColumnName = column.DbColumnName?.ToLower();  //转小写
        }
    }
});
```

### 11. 建库添加字符集和表引擎

建库加字符集（需要字符串上配置 `charset=utf8mb4`）：
```csharp
StaticConfig.CodeFirst_MySqlCollate = "utf8mb4_0900_ai_ci";  //较高版本支持
db.DbMaintenance.CreateDatabase();  //创建库
```

表加引擎：
```csharp
StaticConfig.CodeFirst_MySqlTableEngine = " InnoDB ";
//或者
StaticConfig.CodeFirst_MySqlTableEngine = " InnoDB DEFAULT CHARSET=utf8mb4";
```

### 12. MySQL 性能优化

| 数据量 | 优化方式 |
|--------|----------|
| 30 万+ | 避免联表查 count，使用导航查询或 ThenMapper |
| 200 万+ | 单表 count 也会变慢，不返回 count 只查前 100 页 |

减少联表，使用导航查询 `Includes` 填充。

### 13. 事务注意点

1. MYSQL 不支持创建表和删除表处理事务，原生事务也一样
2. MyISAM 存储引擎不支持事务，需要改成 InnoDB

### 14. 禁用 Char(36) 转 GUID

驱动默认是将 char36 当作 GUID 来使用，可以通过连接字符串禁用：
```
OldGuids=True
```

### 15. Point 类型

```csharp
[SugarColumn(InsertSql = "ST_GeomFromText('{0}')", UpdateSql = "ST_GeomFromText('{0}')", QuerySql = "ST_AsText(coordinate)")]
public string coordinate { get; set; }

//插入和更新用实体方式更新
oldModel.coordinate = "point(22.22 33.33)";
db.Insertable(oldModel).ExecuteCommand();
db.Updateable(oldModel).ExecuteCommand();
```

### 16. 连接不上库的所有情况

先用原生 .NET 测试：
```csharp
new MySqlConnection("xxxx;charset=utf8;").Open();  //在没有编码情况下 ORM 会加上 charset=utf8;
```

| 编号 | 错误 | 方案 |
|------|------|------|
| 1 | `parametric information is abnormal`、`Packets out of order`、`Unsupported command(Com_RESET_CONNECTION)` | 连接字符串加 `Pooling=false` |
| 2 | `Connection open error. 调用 SSPI 失败`（.NET Framework） | 加 `SslMode=none`，`CharSet` 改为 utf8mb4；或换用 `DbType.MySqlConnector` + 安装 MySqlConnector DLL |
| 3 | 第一次连不上，Navicat 连接后又可以连上 | 参考官方论坛解决方案 |
| 4 | ORM 默认加 `;charset=utf8` 报错 | 换一个不报错的编码 |
| 5 | 给定的关键字不在字典中 | 检查连接字符串，.NET Framework 升级 mysql.data |
| 6 | SSL Authentication Error | 加 `SslMode=none;AllowPublicKeyRetrieval=True;` |
