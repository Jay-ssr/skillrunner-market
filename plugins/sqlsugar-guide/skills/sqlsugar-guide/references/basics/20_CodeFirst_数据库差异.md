# CodeFirst 数据库差异

## MySQL

- 建库时可通过 `StaticConfig.CodeFirst_MySqlCollate` 指定字符集排序规则
- 建表时可通过 `StaticConfig.CodeFirst_MySqlTableEngine` 指定引擎或附带默认字符集
- MySQL 不支持把“创建表 / 删除表”放进事务，这个限制对 Code First 同样成立

```csharp
StaticConfig.CodeFirst_MySqlCollate = "utf8mb4_0900_ai_ci";
StaticConfig.CodeFirst_MySqlTableEngine = " InnoDB DEFAULT CHARSET=utf8mb4";
db.DbMaintenance.CreateDatabase();
```

## SqlServer

- 如需让 Code First 默认创建 `NVARCHAR`，开启 `SqlServerCodeFirstNvarchar`

```csharp
MoreSettings = new ConnMoreSettings()
{
    SqlServerCodeFirstNvarchar = true
};
```

## PostgreSQL

- PostgreSQL 默认倾向于小写表名和字段名
- 若要保留驼峰，需要同时关注查询和建表两个开关

```csharp
MoreSettings = new ConnMoreSettings()
{
    PgSqlIsAutoToLower = false,
    PgSqlIsAutoToLowerCodeFirst = false
};
```

## SQLite

- SQLite 的默认值、备注、删列都依赖额外开关
- SQLite 对删列和改列的支持弱于其他数据库，设计结构演进时应更保守

```csharp
MoreSettings = new ConnMoreSettings()
{
    SqliteCodeFirstEnableDefaultValue = true,
    SqliteCodeFirstEnableDescription = true,
    SqliteCodeFirstEnableDropColumn = true
};

public class CodeFirstUnitafa
{
    [SugarColumn(DefaultValue = "(strftime('%Y-%m-%d %H:%M:%S', 'now', 'localtime'))")]
    public DateTime Name { get; set; }
}
```
