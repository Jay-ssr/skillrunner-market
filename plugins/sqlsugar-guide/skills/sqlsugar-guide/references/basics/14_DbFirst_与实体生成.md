# DbFirst 与实体生成

## 生成实体

```csharp
// .NET 6 以下
db.DbFirst.IsCreateAttribute().CreateClassFile("c:\\Demo\\1", "Models");

// .NET 6 及以上：string 加 ?
db.DbFirst.IsCreateAttribute().StringNullable().CreateClassFile("c:\\Demo\\1", "Models");
```

详细文档：https://www.donet5.com/Home/Doc?typeId=1207

## .NET 6+ 非空 string 编译警告

不要使用 `required`，可改为下面写法：

```csharp
public class Order
{
    [SqlSugar.SugarColumn(IsPrimaryKey = true, IsIdentity = true)]
    public int Id { get; set; }
    public string Name { get; set; } = null!;
}
```

## 快捷生成实体

所有数据库都支持。只适合常规场景；个性化要求高时，改用自定义模板、Razor 模版或自定义生成器。

### 生成到指定目录

```csharp
// .NET 6 以下
db.DbFirst.IsCreateAttribute().CreateClassFile("c:\\Demo\\1", "Models");

// .NET 6 及以上：string 加 ?
db.DbFirst.IsCreateAttribute().StringNullable().CreateClassFile("c:\\Demo\\1", "Models");

// 参数1：路径
// 参数2：命名空间
// IsCreateAttribute：生成 SqlSugar 特性
```

### 格式化文件名

```csharp
db.DbFirst.FormatFileName(x => x.ToLower()).CreateClassFile("c\\");
```

### .NET 7 字符串是否加 `?`

```csharp
db.DbFirst.StringNullable().CreateClassFile("c\\"); // 强制可空 string 加 ?
```

### 按条件筛选生成

```csharp
db.DbFirst.Where("Student").CreateClassFile("c:\\Demo\\2", "Models");
db.DbFirst.Where(it => it.ToLower().StartsWith("view")).CreateClassFile("c:\\Demo\\3", "Models");
db.DbFirst.Where(it => it.ToLower().StartsWith("view")).CreateClassFile("c:\\Demo\\4", "Models");
```

### 生成带 SqlSugar 特性的实体

```csharp
db.DbFirst.IsCreateAttribute().CreateClassFile("c:\\Demo\\5", "Models");
```

### 生成带默认值的实体

```csharp
db.DbFirst.IsCreateDefaultValue().CreateClassFile("c:\\Demo\\6", "Demo.Models");

// 5.1.4.108-preview12+ 支持替换字符串
db.DbFirst.Where("Student")
    .CreatedReplaceClassString(it => it.Replace("xxx", "yyy")) // 也可用 Regex.Replace
    .CreateClassFile("c:\\Demo\\2", "Models");
```

## 自定义模板格式化

通过 `SettingPropertyTemplate` 等重载，可以更细粒度地控制生成结果。

```csharp
db.DbFirst
    // 类
    .SettingClassTemplate(old => { return old; /* 修改 old 值后返回 */ })
    // 构造函数
    .SettingConstructorTemplate(old => { return old; /* 修改 old 值后返回 */ })
    .SettingNamespaceTemplate(old => {
        return old + "\r\nusing SqlSugar;"; // 追加 SqlSugar 引用
    })
    // 属性备注
    .SettingPropertyDescriptionTemplate(old => { return old; /* 修改 old 值后返回 */ })
    // 属性：新重载，可完全自定义
    .SettingPropertyTemplate((columns, temp, type) => {
        var columnattribute = "\r\n           [SugarColumn({0})]";
        List<string> attributes = new List<string>();
        if (columns.IsPrimarykey)
            attributes.Add("IsPrimaryKey=true");
        if (columns.IsIdentity)
            attributes.Add("IsIdentity=true");
        if (attributes.Count == 0)
        {
            columnattribute = "";
        }
        return temp.Replace("{PropertyType}", type)
            .Replace("{PropertyName}", columns.DbColumnName)
            .Replace("{SugarColumn}", string.Format(columnattribute, string.Join(",", attributes)));
    })
    .CreateClassFile("c:\\Demo\\7");
```

该能力可能与 `IsCreateAttribute` 冲突，避免同时使用。

## 格式化类名和属性名

新功能：`5.1.4.115`

```csharp
db.DbFirst
    .IsCreateAttribute() // 生成 SqlSugar 自带特性
    .FormatFileName(it => "File_" + it) // 格式化文件名
    .FormatClassName(it => "Class_" + it) // 格式化类名
    .FormatPropertyName(it => "Property_" + it) // 格式化属性名
    .CreateClassFile("c:\\Demo\\4", "Models");

// 每种格式化只能设置一次
// 正确
FormatFileName(it => it.Replace(" ", "").Replace("-", "_"))
// 错误
.FormatFileName(it => it.Replace(" ", ""))
.FormatFileName(it => it.Replace("-", "_"))
```

## 替换生成后的 `ClassString`

该方式性能损耗最大，优先使用其他格式化能力。

```csharp
// 5.1.4.108-preview12+ 支持替换字符串
db.DbFirst.Where("Student")
    .CreatedReplaceClassString(it => it.Replace("xxx", "yyy").Replace("zzz", "ssss")) // 也可用 Regex.Replace
    .CreateClassFile("c:\\Demo\\2", "Models");
```

## 添加租户特性

```csharp
db.DbFirst.Where("order").SettingClassDescriptionTemplate(it => {
    return it + "\r\n    [Tenant(\"" + db.CurrentConnectionConfig.ConfigId + "\")]";
}).CreateClassFile("c:\\Demo\\1", "Models");
```

## 生成 `string?`（.NET 7+）

```csharp
db.DbFirst.StringNullable().CreateClassFile("c\\"); // 强制可空 string 加 ?
```
