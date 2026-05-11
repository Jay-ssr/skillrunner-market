# Access

## NuGet
- .NET Framework: `SqlSugar` + `SqlSugar.Access`
- .NET Core: `SqlSugarCore` + `SqlSugar.AccessCore`

- **DbType**: `DbType.Access`

## 连接字符串
```csharp
Provider=Microsoft.ACE.OleDB.15.0;Data Source=" + GetCurrentProjectPath + "\\test.accdb
```

## 注意事项
- 新项目推荐 Sqlite
- 不同 Office 版本连接字符串有差异
- .NET CORE 下发布到 X86 文件夹
- 使用 X86 编译（与 Office 安装版本一致），避免 Any CPU

## 联表分页
Access 联表分页存在 OrderBy 时不支持：
1. 使用内存分页 `.ToList().Skip(1).Take()`
2. 去掉 OrderBy
3. 使用子查询

## 三表以上 JOIN
Access 默认只支持 2 表联表，通过嵌套查询实现：
```csharp
var getAll = db.Queryable<Order>()
    .LeftJoin<Order>((x, y) => x.Id == y.Id)
    .MergeTable()  //通过mergetable合并成一个表在进行JOIN
    .LeftJoin<Order>((x, y) => x.Id == y.Id)
    .ToList();
```
