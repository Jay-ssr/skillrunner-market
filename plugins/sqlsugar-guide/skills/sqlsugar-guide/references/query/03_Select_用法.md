# Select 用法

### 返回单字段

```csharp
List<int> listInt = db.Queryable<Student>().Select(it => it.Id).ToList();  // 返回一个字段
List<string> names = db.Queryable<Student>().Select(it => it.Name).ToList();

// 字符串方式
List<int> listInt = db.Queryable<Student>().Select<int>("id").ToList();
```

### 返回多字段

```csharp
// 返回匿名对象
DataTable dt = db.Queryable<Student>()
    .Select(it => new { id = it.Id, name = it.Name })
    .ToDataTable();

// 返回动态对象
List<dynamic> list = db.Queryable<Student>()
    .Select(it => (dynamic) new { id = it.Id, name = it.Name })
    .ToList();

// 返回指定类
List<Class1> list = db.Queryable<Student>()
    .Select(it => new Class1 { id = it.Id, name = it.Name })
    .ToList();

// 字符串方式
List<Order> list = db.Queryable<Student>()
    .Select<ViewModel>("id as id, name as name")
    .ToList();
```

### Dto 映射

```csharp
// 返回匿名集合 - 支持跨程序集
List<dynamic> dynamicList = db.Queryable<Student>()
    .Select(it => (dynamic) new { id = it.id })
    .ToList();

// 返回匿名集合 - 不能跨程序集
var dynamic = db.Queryable<Student>()
    .Select(it => new { id = it.id })
    .ToList();

// 返回类集合
List<Student> list = db.Queryable<Student>()
    .Select(it => new Student { id = it.id })
    .ToList();

// 自动返回 Dto 集合 (5.1.3.2)
var listDto = db.Queryable<Student>().Select<StudentDto>().ToList();

// 自动返回 Dto + 手动指定字段 (5.1.3.35)
var listDto = db.Queryable<Student>()
    .Select(it => new StudentDto()
    {
        Count = 100  // 手动指定
    }, true)  // true 表示开启自动映射
    .ToList();
```

### 多表返回 Dto

```csharp
// 手动 Dto
var newClass = db.Queryable<Student, School, DataTestInfo>(
    (st, sc, di) => new JoinQueryInfos(
        JoinType.Left, st.SchoolId == sc.Id,
        JoinType.Left, st.Name == di.String
    ))
    .Select((st, sc, di) => new ClassName { name = st.Name, scid = sc.Id })
    .ToList();

// 实体自动映射 1 (5.1.3.35) - 语法最美
var list4 = db.Queryable<SchoolA>()
    .LeftJoin<StudentA>((x, y) => x.SchoolId == y.SchoolId)
    .Select((x, y) => new UnitView01()
    {
        Name = x.SchoolName,
        Count = 100
    }, true)  // true 表示其余字段自动映射
    .ToList();
// SQL: SELECT [x].[ID] AS [id], [x].[Time] AS [Time], [x].[SchoolName] AS [Name], 100 AS [Count]
// FROM [SchoolA] x Left JOIN StudentA y ON ...

// 实体自动映射 2 - 通过 x.* 方式
var oneClass = db.Queryable<Order>()
    .LeftJoin<OrderItem>((o, i) => o.Id == i.OrderId)
    .LeftJoin<Custom>((o, i, c) => o.CustomId == c.Id)
    .Where(o => o.Id > 1)
    .Select((o, i, c) => new ViewOrder
    {
        Id = o.Id.SelectAll(),        // 等于 o.*
        CustomName = c.Name            // 等于 [c].[Name] AS [CustomName]
    }).ToList();
// SQL: SELECT o.*, [c].[Name] AS [CustomName] FROM ...

// 实体自动映射 3 - 通过约束实现
public class ViewOrder
{
    public string Name { get; set; }        // 主表规则【字段名】
    public string CustomName { get; set; }   // 从表规则【class+字段名】
    public string OrderItemPrice { get; set; } // 从表规则【class+字段名】
}

var viewModel = db.Queryable<Order>()
    .LeftJoin<OrderItem>((o, i) => o.Id == i.OrderId)
    .LeftJoin<Custom>((o, i, c) => o.CustomId == c.Id)
    .Select<ViewOrder>().ToList();
```

### Select 位置

```csharp
// 正常情况：应该在最后面
db.Queryable<Student>().Where(..).OrderBy(..).Select(..).ToList()

// 特殊情况：如果 Select 不是最后一个位置，需要加 MergeTable
Select(...).MergeTable().Where
```

### Select 之后字段处理

```csharp
// 方式 1 (5.1.4.113-preview+)
// 定义处理方法
public class UnitTool
{
    public static string GetName(string name)
    {
        return "name" + 111;
    }
}

// 获取 methodInfo
var methodInfo = typeof(UnitTool).GetMethod("GetName");
var list8 = db.Queryable<Order>()
    .Select(it => new
    {
        n = it.Name,
        name = SqlFunc.OnlyInSelectConvertToString(it.Name, methodInfo)
    }).ToList();

// 方式 2 - Mapper
// 实体类
var list = db.Queryable<Order>()
    .Select(it => new Order { Id = it.Id, Name = it.Name })
    .Mapper(it => {  // 只能写在 Select 后面
        it.Name = it.Id + it.Name;  // 相当于 ToList 循环赋值
    }).ToList();

// 匿名对象
var list = db.Queryable<Order>()
    .Select(it => (dynamic) new { Id = it.Id, Name = it.Name })
    .Mapper(it => {
        it.Name = it.Id + it.Name;
    }).ToList();
// 注意：(dynamic) 不要漏了
```

### 动态 Select

```csharp
// 方式 1：多库兼容
var selector = new List<SelectModel>() {
    new SelectModel() { FiledName = "id", AsName = "id2" },
    new SelectModel() { FiledName = "id" },
    new SelectModel() { FieldName = "{string}:a", AsName = "Name" }  // 常量
};
var list = db.Queryable<Order>().Select(selector).ToList();

// 方式 2：直接写 SQL
var list = db.Queryable<Order>().Select("ID AS id1, id as id").ToList();

// 方式 3: 动态表达式
StaticConfig.DynamicExpressionParserType = typeof(DynamicExpressionParser);
var list = db.Queryable<Order>()
    .Select("it", $"it=>new(it.Id as Id, it.Name)", typeof(Order))
    .ToList();
```

### 别名用法

```csharp
Select(it => new { id1 = it.id, name2 = it.name })
// SQL: SELECT id AS id1, name AS name2
```

---
