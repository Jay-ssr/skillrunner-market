# Sql分页（SqlServer特殊处理）

## 适用范围

`db.SqlQueryable` 适用于简单查询；存储过程或复杂 SQL 使用 Ado / 存储过程方式。

## 查询分页

`OrderBy` 要写在外层。

```csharp
int total=0;
var list = db.SqlQueryable<Student>("select * from student")
    .Where(it=>it.Id==1) // 可以表达式
    .OrderBy("id asc") // 也可以SQL
    .ToPageList(1, 2,ref total);

// 使用in
var list2= db.SqlQueryable<Student>("select * from  student where id in (@ids)  ").AddParameters(new SugarParameter[] {
    new SugarParameter("@ids", new int[] { 1, 2 })
})
    .OrderBy("id asc")
    .ToPageList(1,2);
```

> 无需分页时，优先使用 `db.Ado.SqlQuery`，兼容性更完整：`var list = db.Ado.SqlQuery<Student>("复杂Sql或者存储过程");`

## 结合表达式

```csharp
var list= db.SqlQueryable<Student>("select * from student").Where(it=>it.Id==1).ToPageList(1, 2);
```

## 更多用法

```csharp
var list= db.SqlQueryable<Student>("select * from student").Where("id=@id",new { id=1}).ToPageList(1, 2);

// 如果多个参数 new { id=1 , name="xx"} 用逗号隔开
```

## 添加参数

```csharp
var list = db.SqlQueryable<Student>("select * from student  where id=@id").AddParameters(new { id=1}).ToPageList(1, 2, ref total);

// AddParameters 有很多重载
ISugarQueryable<T> AddParameters(object parameters);
ISugarQueryable<T> AddParameters(SugarParameter[] parameters);
ISugarQueryable<T> AddParameters(List<SugarParameter> parameters);
```

## 无实体操作

```csharp
var list= db.SqlQueryable<object>("select * from student").Where("id=@id",new {id=1 }).ToDataTablePage(1, 2);
```

## IN 参数

```csharp
var list= db.SqlQueryable<object>("select * from [order] where  id in(@ids)" )
    .AddParameters(new{ids= new int[] { 1,2,3}}) // 这儿id需要一个数组
    .ToList()
```

## 联表多对象映射

类似 Dapper `Query<T,T2>`，将一维结果映射为二维对象。

```csharp
var list=db.SqlQueryable<SQLVO>("select 1 as id,'jack' as name ").ToList();

public class SQLVO
{
    [SugarColumn(IsOwnsOne =true)]
    public ITEM1 ITEM1 { get; set; } // item1和item2不能有重复字段
    
    [SugarColumn(IsOwnsOne = true)]
    public ITEM2 ITEM2 { get; set; } // item1和item2不能有重复字段
}

public class ITEM1
{
    public int ID { get; set; }
}

public class ITEM2
{
    public string Name { get; set; }
}
```

## 存储过程和复杂 SQL

超出简单分页场景时，使用 Ado 或存储过程方式。
