# ValueObject 值对象

## 版本要求

`SqlSugarCore 5.1.4.141` 及以上版本。

## 功能说明

表是一维结构，实体可以用二维对象来表达。

| 方式 | 表结构 | 特点 |
|------|--------|------|
| 一对一 | 2 张表 | 导航关联 |
| 值对象 | 1 张表 | 部分字段封装成对象 |
| JSON 存类 | 1 个字段存多个字段 | 数据库只有 1 列 |
| 值对象 vs JSON | 数据库仍是多个字段 | 实体接收为 1 个对象 |

## 完整用例

### 表结构

值对象场景下，表结构仍是一维展开的单表，例如 `CustomerId`、`Name`、`Street`、`City`、`ZipCode` 都在同一张表中。

### 代码

```csharp
// 插入
db.Insertable(new UnitCustomeradfafas()
{
    CustomerId = 1,
    Name = "name",
    Address = new UnitAddressadfafa()
    {
        City = "city",
        Street = "street",
        ZipCode = "zipCode"
    }
}).ExecuteCommand();

// 更新
db.Updateable(new UnitCustomeradfafas()
{
    CustomerId = 1,
    Name = "name2",
    Address = new UnitAddressadfafa()
    {
        City = "city2",
        Street = "street2",
        ZipCode = "zipCode2"
    }
}).ExecuteCommand();

// 查询
var list = db.Queryable<UnitCustomeradfafas>().ToList();

// 条件查询
var list2 = db.Queryable<UnitCustomeradfafas>()
    .Where(it => it.Address.City == "city2")
    .Select(it => new {
        Street = it.Address.Street
    }).ToList();

public class UnitAddressadfafa
{
    // 支持 SugarColumn 设置别名
    public string Street { get; set; }
    public string City { get; set; }
    public string ZipCode { get; set; }
}

public class UnitCustomeradfafas
{
    [SqlSugar.SugarColumn(IsPrimaryKey = true)]
    public int CustomerId { get; set; }
    public string Name { get; set; }
    [SqlSugar.SugarColumn(IsOwnsOne = true)]
    public UnitAddressadfafa Address { get; set; }
}
```

虽然实体是二维结构，但对应表仍是一维单表。

## 技巧

### 切换 DTO

如果只希望查询结果是二维结构，而插入和更新仍使用一维实体，可以切换到 DTO：

```csharp
var list3 = db.Queryable<普通类>()
    .Select<Dto>().ToList();

public class Dto
{
    public int CustomerId { get; set; }
    public string Name { get; set; }
    [SqlSugar.SugarColumn(IsOwnsOne = true)] // 标识值对象
    public Address Address { get; set; }
}
```
