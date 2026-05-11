# Where 条件构建

## 常见写法

### 表达式查询

```csharp
var list = db.Queryable<Student>().Where(it => it.Id == id).ToList();

var list2 = db.Queryable<Student>()
    .WhereIF(id > 0, it => it.Id == id)
    .WhereIF(name != null, it => it.name == "a")
    .ToList();

var list3 = db.Queryable<Student>()
    .Where(it => it.Id == id || it.Name.Contains("jack"))
    .ToList();
// && = AND，|| = OR
```

### SQL 字符串查询

```csharp
var list = db.Queryable<Student>()
    .Where("id=@id", new { id = 1 })
    .ToList();

var list2 = db.Queryable<Student>()
    .Where("id=@id OR name LIKE '%'+@name+'%'", new { id = 1, name = "jack" })
    .ToList();
```

## 动态条件

### `ConditionalModel` / JSON

```csharp
[
    {"FieldName": "id", "ConditionalType": "0", "FieldValue": "1"},
    {"FieldName": "name", "ConditionalType": "0", "FieldValue": "jack"}
]

var conModels = db.Utilities.JsonToConditionalModels(json);
var student = db.Queryable<Student>().Where(conModels).ToList();

var cs = new List<IConditionalModel>
{
    new ConditionalModel
    {
        FieldName = "id",
        ConditionalType = ConditionalType.Equal,
        FieldValue = "1"
    },
    new ConditionalModel
    {
        FieldName = "name",
        ConditionalType = ConditionalType.Equal,
        FieldValue = "jack"
    }
};
```

### `Expressionable`

```csharp
var exp = Expressionable.Create<Student>()
    .And(it => it.Id == 1)
    .Or(it => it.Name.Contains("jack"))
    .ToExpression();

var list = db.Queryable<Student>().Where(exp).ToList();

var exp2 = Expressionable.Create<Student>();
exp2.OrIF(条件, it => it.Id == 1);
exp2.Or(it => it.Name.Contains("jack"));
var list2 = db.Queryable<Student>().Where(exp2.ToExpression()).ToList();
```

> `ToExpression()` 不能省略。

## 链式条件

```csharp
var query = db.Queryable<Student>().Where(it => it.Id == 1);

if (条件)
    query.Where(it => it.Name == "jack");

if (条件)
    query.Where(it => it.Id == 1);

int count = query.Clone().Count();
var list = query.Clone().ToList();
```

> 同一个 `query` 要复用到多个地方时，先 `Clone()`。

## 根据对象构建条件

### `WhereClass`

```csharp
var getAll = db.Queryable<Order>()
    .WhereClass(new Order() { Name = "a" }, ignoreDefaultValue: true)
    .ToList();

var getAll2 = db.Queryable<Order>()
    .WhereClass(List<Order>, ignoreDefaultValue: true)
    .ToList();

var byPk = db.Queryable<Order>()
    .WhereClassByPrimaryKey(new Order() { Id = 1 })
    .ToList();

var byPkList = db.Queryable<Order>()
    .WhereClassByPrimaryKey(List<Order>)
    .ToList();
```

`ignoreDefaultValue: true` 时，默认值不会参与条件，例如 `Id = 0` 不会进 `WHERE`。

### 字典查询

```csharp
var getAll = db.Queryable<Order>()
    .WhereColumns(new List<Dictionary<string, object>>)
    .ToList();
```
