# Oracle

- **NuGet**: `SqlSugarCore`

- **DbType**: `DbType.Oracle`

## 连接字符串
```csharp
//写法1
Data Source=localhost/orcl;User ID=system;Password=haha

//写法2
Data Source=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=150.158.57.125)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=ORCL)));User Id=xx;Password=xx;Pooling='true';Max Pool Size=150
```

## 初始化
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.Oracle,
    ConnectionString = "Data Source=localhost/orcl;User ID=system;Password=haha",
    IsAutoCloseConnection = true
});
```

## 表模式说明
默认为大写表模式。

不转大写模式：
```csharp
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.Oracle,
    ConnectionString = "Data Source=localhost/orcl;User ID=system;Password=haha",
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        IsAutoToUpper = false  //禁用自动转成大写表
    }
});
```

## 注意事项

### 1. 自增 - 序列实现自增

Oracle 12C 以下不支持自增列，使用序列实现：
```sql
-- 数据库创建序列
create sequence SEQ_ID
minvalue 1
maxvalue 99999999
start with 1
increment by 1
nocache
order;
```

```csharp
//实体类
public class OrderItem
{
    [SqlSugar.SugarColumn(IsPrimaryKey = true, OracleSequenceName = "SEQ_ID")]
    public int ItemId { get; set; }
    public int OrderId { get; set; }
    public decimal? Price { get; set; }
    [SqlSugar.SugarColumn(IsNullable = true)]
    public DateTime? CreateTime { get; set; }
}

//插入返回自增列只支持单条返回自增
var id = db.Insertable(insertObj).ExecuteReturnIdentity();
```

### 2. 自增 - 原生自增列（12C+）

```csharp
SqlSugarClient db2 = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.Oracle,
    ConnectionString = Config.ConnectionString3,
    IsAutoCloseConnection = true,
    MoreSettings = new ConnMoreSettings()
    {
        EnableOracleIdentity = true  //启用Oracle自增列，需要12C以上版本
    }
});

//实体配置 IsIdentity=true, IsPrimaryKey=true
```

### 3. 游标参数

```csharp
var p = new SugarParameter("@name", "");  //有的传null报错，可以试试 null 和 ""
p.IsRefCursor = true;  //游标
p.Direction = System.Data.ParameterDirection.Output;  //如果是output还需要加这行
```

### 4. Oracle 时间存储带毫秒

升级较新版本即可。

### 5. Oracle 批量更新异常缓慢时

1. 批量更新比循环更新还慢时，升级版本解决
2. 使用 BulkCopy

### 6. 查询非常慢

#### 6.1 时间字段引起的慢：
```csharp
//可以设置参数类型，保证数据库一样就能走索引
[SugarColumn(SqlParameterDbType = System.Data.DbType.Date)]
public DateTime Time { get; set; }

//Ado.net
var p = new SugarParameter("@p", DateTime.Now, System.Data.DbType.Date);
```

#### 6.2 字符串引起的慢：
```csharp
[SugarColumn(ColumnDataType = "nvarchar2", SqlParameterDbType = typeof(Nvarchar2PropertyConvert))]
public string Name { get; set; }
```

### 7. 存储过程 Clob

```csharp
var p1 = new SugarParameter("@p", 1);
p1.IsClob = true;
```

### 8. 存储过程 Blob

传 null 会报错 `oracle value does not fall within the expected range`。
```csharp
var p1 = new SugarParameter(":p", byte[]);  //使用 new byte[]{} 替代 null
```

常见错误 `ORA-01461`，解决方案：
1. 升级 11G 补丁
2. 升级 SqlSugar 5.1.4.131-preview05+ 版本
```csharp
var p1 = new SugarParameter(":p", byte[]) { CustomDbType = OracleDbType.Blob };
```

### 9. 索引破坏

错误：`ORA-26028: 索引 XXX 在最初处于无法使用的状态`

解决方案：
```sql
alter table TESTFAST11 drop primary key
-- 找出重复数据删掉
-- 在重新创建索引
```

### 10. ORA-01745 无效的主机/绑定变量名

可能表和字段名是关键字：
```csharp
[SugarColumn(SqlParameterDbType = typeof(CommonPropertyConvert))]
public DateTime DcValue { get; set; }
```

### 11. Oracle11 超出长度上限

5.1.4.140：Oracle 11 主键名字和参数名字超过30报错
```csharp
MoreSettings = new ConnMoreSettings()
{
    MaxParameterNameLength = 30  //设置最大长度
}
```

### 12. Db.Fastest 乱码

部分用户出现大数据导入乱码，单独安装 `Oracle.ManagedDataAccess.Core 3.21.50`

### 13. US7ASCII 乱码问题

推荐改字符集（ODB.NET Oracle 不支持）：

| 方案 | 说明 |
|------|------|
| ODBC 连接 | 性能差 |
| OceanBase Oracle 模式 | - |
| 改字符集 | 推荐 |
| OLEDB 实现 | - |
| OracleUS7ASCLL 专用版本 | 只支持 .NET Framework |

### 14. 特殊字符乱码

新版本使用 Nvarchar2：
```csharp
[SugarColumn(SqlParameterDbType = typeof(Nvarchar2PropertyConvert))]
public string Name { get; set; }
```

老版本使用 Nvarchar2：
```csharp
db.Aop.OnExecutingChangeSql = (sql, pars) =>
{
    if (pars != null)
    {
        foreach (var item in pars)
        {
            item.IsNvarchar2 = true;
        }
    };
    return new KeyValuePair<string, SugarParameter[]>(sql, pars);
};
```

### 15. NClob 类型

```csharp
[SugarColumn(SqlParameterDbType = typeof(NClobPropertyConvert))]
public string Name { get; set; }
```

### 16. 数组参数

```csharp
var p = new SugarParameter("@p", value);
p.IsArray = true;
```

### 17. 超出 C# 精度的类型处理

例如 `FLOAT(64)` Oracle 驱动不支持：
```csharp
public class UnitBigNumber
{
    [SqlSugar.SugarColumn(IsPrimaryKey = true)]
    public int Id { get; set; }

    //QuerySql在单表没有select的查询中生效
    [SqlSugar.SugarColumn(QuerySql = "CAST( Num AS varchar(100))", ColumnDataType = "FLOAT(64)")]
    public string Num { get; set; }
}

var list = db.Queryable<UnitBigNumber>().Select(it => new UnitBigNumber
{
    adfadfa = it.adfadfa.ToString()
}, true).ToList();  //true表示自动映射其他字段
```

### 18. Varchar2 用法

#### 18.1 局部设置：
```csharp
[SugarColumn(ColumnDataType = "nvarchar2", SqlParameterDbType = typeof(Nvarchar2PropertyConvert))]
public string Name { get; set; }

var p = new SugarParameter("@name", "a") { CustomDbType = OracleDbType.Varchar2 };
```

#### 18.2 全局配置：
```csharp
db.Aop.OnExecutingChangeSql = (sql, pars) =>
{
    foreach (var p in pars)
    {
        if (p.DbType == DbType.String || p.DbType == DbType.AnsiString)
            p.CustomDbType = OracleDbType.Varchar2;
    }
    return new KeyValuePair<string, SugarParameter[]>(sql, pars);
};
```
