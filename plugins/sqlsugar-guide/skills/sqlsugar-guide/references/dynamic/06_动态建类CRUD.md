# 动态建类 CRUD

## 优点

- 多库场景支持 JSON、自定义类型、建表和 CRUD（纯字典方式无法做到）
- 支持实体能力：AOP、过滤器、导航等

> 相同表创建出的 `Type` 需缓存（内置缓存或自行缓存）。无限创建类型会导致内存泄露。

## 动态建类 + CRUD

```csharp
var typeBilder = db.DynamicBuilder().CreateClass("table1", new SugarTable(){});
// 可以循环添加列
typeBilder.CreateProperty("Id",typeof(int),new SugarColumn(){IsPrimaryKey=true,IsIdentity=true});
typeBilder.CreateProperty("Name", typeof(string), new SugarColumn() { });
typeBilder.WithCache(); // 缓存Key 表名+字段名称相加

// 创建类
var type = typeBilder.BuilderType();

// 创建表
db.CodeFirst.InitTables(type); // 建表属性API看迁移

// 根据字典转成类对象
var dic=new Dictionary<string,object>(){{ "Id", 1 },{ "Name", "jack" }};
var value= db.DynamicBuilder().CreateObjectByType(type,dic); // 可以是List<字典>
// 可以是List<Dictionary<string, object>()

db.InsertableByObject(value).ExecuteCommand();
db.UpdateableByObject(value).ExecuteCommand();
db.DeleteableByObject(value).ExecuteCommand();
db.StorageableByObject(value).ExecuteCommand(); // 插入或者更新

// 查询 带有类功能 5.1.4.84
// API和无实体查询类似
db.QueryableByObject(type).ToList();
```

## 动态建类 + 分表操作

```csharp
var type = db.DynamicBuilder().CreateClass("table1", new SugarTable()
{
},null,null,new SplitTableAttribute(SplitType.Day))
    .CreateProperty("Id",typeof(int),new SugarColumn(){IsPrimaryKey=true,IsIdentity = true })
    .CreateProperty("Time", typeof(DateTime), new SugarColumn() { },true) // true表示分表字符
    .WithCache() // 缓存起来根据表名和字段名组合的KEY
    .BuilderType();

// 创建表
db.CodeFirst.InitTables(type); // 建表属性API看迁移

db.InsertableByObject(value).SpiltTable().ExecuteCommand();
db.UpdateableByObject(value).SpiltTable().ExecuteCommand();
db.DeleteableByObject(value).SpiltTable().ExecuteCommand();

// 查询 需要高版本sqlsugar
db.QueryableByObject(type).SpiltTable().ToList();
```

## 复杂构造

### 继承基类或接口

表已存在且非 CodeFirst 建表时，去掉主键配置，避免约束导致建表失败。

```csharp
var type = db.DynamicBuilder().CreateClass("table12", new SugarTable(){}
    ,typeof(Order) // 继承Order类
)
    .CreateProperty("yyy", typeof(string), new SugarColumn() { })
    // .WithCache() 缓存起来根据表名和字段名组合的KEY
    .BuilderType();
db.CodeFirst.InitTables(type);
```

#### `CreateClass` 全参数

```csharp
public DynamicProperyBuilder CreateClass(
    string entityName,
    SugarTable table=null,
    Type baseType = null, // 基类
    Type[] interfaces = null, // 接口
    SplitTableAttribute splitTableAttribute=null)
```

### 属性是当前类（树形）

需要升级到：`5.1.4.125-preview12+`

```csharp
// 类为 Tree 时，Child 属性为 List<Tree>，使用固定类型代表自身集合
propertyType = typeof(DynamicOneselfTypeList); // List<当前类>

// Parent 属性为 Tree，使用固定类型代表当前类
propertyType = typeof(DynamicOneselfType); // 当前类
```

### 导航：A 里有 B，B 里也有 A

需要升级到：`SqlSugarCore 5.1.4.157-preview10+`

```csharp
var a = db.DynamicBuilder().CreateClass("a")
    .CreateProperty("aid", typeof(int))
    .CreateProperty("bid", typeof(int))
    .CreateProperty("bItem", typeof(SqlSugar.NestedObjectType),
        navigate: new Navigate(NavigateType.OneToOne,nameof(bid)));

var b = db.DynamicBuilder().CreateClass("b")
    .CreateProperty("bid", typeof(int))
    .CreateProperty("aList", typeof(SqlSugar.NestedObjectTypeList), 
        navigate: new Navigate(NavigateType.OneToMany,nameof(A.bid)))
    .WithCache() // 存入缓存防止多次创建引起内存泄露
    .BuilderTypes(a);

var typea = b.Item1;
var typeb = b.Item2;

// nameof(bid) 等价 "bid"，nameof(A.bid) 等价 "bid"
// 直接填字符串即可
```

更多导航属性可参考 5.3。

## 动态类如何加索引

```csharp
Db.DbMaintenance.CreateIndex(....)
// isUnique 是区分普通索引还是唯一索引
bool CreateIndex(string tableName, string [] columnNames, bool isUnique=false);
bool CreateIndex(string tableName, string[] columnNames, string IndexName, bool isUnique = false);
```

## 导航

### 导航过滤（5.1.4.107-preview14）

配置 `Dynamic.Core` 时，在项目启动阶段统一注册。

```csharp
var list5=db.QueryableByObject(EntityType).Where("it", $"it=>it.Address.Id=={1}").ToList();
var list6=db.QueryableByObject(EntityType).Where("it", $"it=>it.Pers.Any(s=>s.AddressId==it.Id)").ToList();
```

### 导航填充

```csharp
var list=db.QueryableByObject(EntityType)
    .Includes("A1","A2") // A1下面有A2
    .Includes("B1")
    .ToList();
```

### 动态创建导航

需要升级到：`5.1.4.110-preview02+`

```csharp
var db = NewUnitTest.Db;
var typeBilder = db.DynamicBuilder().CreateClass("table1dfafa1231", new SugarTable() { });
// 可以循环添加列
typeBilder.CreateProperty("Id", typeof(int), new SugarColumn(){ IsPrimaryKey = true, IsIdentity = true });
typeBilder.CreateProperty("Name", typeof(string), new SugarColumn() { });

// 一对一
typeBilder.CreateProperty("OrderInfo",typeof(Order),
    navigate: new Navigate(NavigateType.OneToOne, "Id" )); // 和导航查询一配置法

// 一对多
typeBilder.CreateProperty("OrderInfos", typeof(List<Order>),
    navigate: new Navigate(NavigateType.OneToMany, nameof(Order.Id))); // 和导航查询一配置法

typeBilder.WithCache(); // 缓存Key 表名+字段名称相加

// 创建类
var type = typeBilder.BuilderType();

// 建表
db.CodeFirst.InitTables(type);

// 设置动态表达式
StaticConfig.DynamicExpressionParserType = typeof(DynamicExpressionParser);

var dic=  new Dictionary<string,object> { { "Name", "jack" } };
var insertObj = db.DynamicBuilder().CreateObjectByType(type, dic);
db.InsertableByObject(insertObj).ExecuteCommand();

var list=db.QueryableByObject(type)
    .Where("it", $"it=>it.OrderInfo.Id==1")
    .Where("it",$"it=>it.OrderInfos.Any()")
    .ToList();

var list2 = db.QueryableByObject(type)
    .Includes("OrderInfo")
    .Includes("OrderInfos")
    .ToList();
```

## 字符串表达式

动态建类场景也可直接使用字符串表达式：

```csharp
.Where("it", $"it=>it.OrderInfo.Id==1")
```

更多字符串表达式写法见对应章节。

## 清空缓存

如果类定义发生变化，可按创建时的 `className` 清理缓存。

```csharp
// 删除正常类
db.Utilities.RemoveCacheByLikeKey<Type>("className")

// 删除（3.3讲解的那种情况）
db.Utilities.RemoveCacheByLikeKey<Tuple<Type, Type>>("className");
```
