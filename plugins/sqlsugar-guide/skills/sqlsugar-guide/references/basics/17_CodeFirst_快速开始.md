# CodeFirst 快速开始

```csharp
public class CodeFirstTable1
{
    [SugarColumn(IsIdentity = true, IsPrimaryKey = true)]
    public int Id { get; set; }
    public string Name { get; set; }
    [SugarColumn(ColumnDataType = "Nvarchar(255)")]
    public string Text { get; set; }
    [SugarColumn(IsNullable = true)]
    public DateTime CreateTime { get; set; }
}

// 建表
db.CodeFirst.SetStringDefaultLength(200).InitTables(typeof(CodeFirstTable1));
```

## 要点

- `InitTables(typeof(...))`：按实体创建表
- `SetStringDefaultLength(200)`：统一设置未显式指定长度的字符串默认长度
- `IsIdentity` + `IsPrimaryKey`：标识自增主键
- `IsNullable = true`：允许空值
- `ColumnDataType`：仅在需要指定数据库列类型时使用
