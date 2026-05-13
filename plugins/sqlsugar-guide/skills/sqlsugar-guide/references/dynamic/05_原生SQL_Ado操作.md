# 原生SQL（Ado操作）

原生 SQL / 存储过程入口。用于复杂 SQL、多结果集、存储过程等场景。

## 基本用法

```csharp
// 调用Sql
db.Ado.具体方法

// 调用存储过程
db.Ado.UseStoredProcedure().具体方法
```

## 常用方法

| 方法名 | 描述 | 返回值 |
|--------|------|--------|
| SqlQuery<T> | 查询所有，返回实体集合 | List<T> |
| SqlQuery<T,T2> | 返回 2 个结果集 / 多结果集 | Tuple<List<T>, List<T2>> |
| SqlQuerySingle | 查询第一条记录 | T |
| SqlQuery<dynamic> | 查询所有，返回匿名对象 | dynamic |
| GetDataTable | 查询所有 | DataTable |
| GetDataReader | 获取 DataReader，需手动释放 | DataReader |
| GetDataSetAll | 获取多个结果集 | DataSet |
| ExecuteCommand | 返回受影响行数，常用于增删改 | int |
| GetScalar | 获取首行首列 | object |
| GetString | 获取首行首列 | string |
| GetInt | 获取首行首列 | int |
| GetLong | 获取首行首列 | long |
| GetDouble | 获取首行首列 | Double |
| GetDecimal | 获取首行首列 | Decimal |
| GetDateTime | 获取首行首列 | DateTime |

## 参数与查询

SQL 关键字前用 `@` 实现多库参数化。

### 无参数

```csharp
var dt=db.Ado.GetDataTable(sql) // 上表中的方法均可使用
```

### 参数 1：匿名对象

```csharp
var dt=db.Ado.GetDataTable("select * from table where id=@id and name like @name",
    new{id=1,name="%"+jack+"%"});
```

### 参数 2：SugarParameter 列表

```csharp
var dt=db.Ado.GetDataTable("select * from table where id=@id and name like @name",
    new List<SugarParameter>(){
        new SugarParameter("@id",1),
        new SugarParameter("@name","%"+jack+"%") // 执行sql语句
    });
```

### 参数 3：先生成再调整

```csharp
var pars =db.Ado.GetParameters(new{p=1,p2=p});
pars[1].DbType=System.Data.DbType.Date;
var dt=db.Ado.GetDataTable(sql,pars)
```

### 原生 SQL 查实体

```csharp
List<ClassA> t=db.Ado.SqlQuery<ClassA>(sql);
// 比 db.SqlQueryable 兼容性更强，支持复杂 SQL 和存储过程；无自带分页
```

### 原生 SQL 查匿名对象

```csharp
List<dynamic> t=db.Ado.SqlQuery<dynamic>(sql);
```

### 增删改

```csharp
db.Ado.ExecuteCommand(sql);
```

> `db.Ado.xxx` 还有更多方法；查询常用 `GetDataTable`、`SqlQuery`，增删改常用 `ExecuteCommand`。

## 存储过程

### 简单用法

```csharp
var dt = db.Ado.UseStoredProcedure().GetDataTable("sp_school",new{name="张三",age=0});
```

### 带 output 参数

```csharp
var nameP= new SugarParameter("@name", "张三");
var ageP= new SugarParameter("@age", null, true); // output 参数
var dt = db.Ado.UseStoredProcedure().GetDataTable("sp_school",nameP,ageP);
var list = db.Ado.UseStoredProcedure().SqlQuery<Class1>("sp_school",nameP,ageP);

// ageP.Value 获取 output 值
```

> SqlServer DataTable 参数、Oracle 游标参数、Blob/Clob 类型，按各数据库驱动分别处理。

### ReturnValue

```csharp
var nameP=new SugarParameter("@name", "张三", typeof(string),ParameterDirection.ReturnValue);
```

### 用 `GetParameters` 简化参数处理

```csharp
SugarParameter [] pars =db.Ado.GetParameters(new{p=1,p2=p});
pars[1].Direction=ParameterDirection.Output;
```

## 特殊参数与批处理

### IN 参数

```csharp
var dt = db.Ado.SqlQuery<Order>(
    "select * from [order] where  id in(@ids)",
    new { ids = new int[] { 1,2,3 } }); // 是个数组不是字符串

// new SugarParamter("@ids",int[] { 1,2,3})

// select * from [order] where  id in('1','2','3')
```

### SqlServer 带 GO 的脚本

```csharp
db.Ado.ExecuteCommandWithGo(sql) // go语句是独立一行就支持
```

### 查询 2 个结果集

等同于 Dapper `QueryMultiple`。

```csharp
var views=db.SqlQuery<T,T2>("select * from t; select * from t2"); // 多实体
var t1list=views.Item1;
var t2list=views.Item2;
```

### 集合参数批量操作

与 Dapper 两层集合参数思路一致，SqlSugar 可不直接写 SQL。

```csharp
// 删除
List<Dictionary<string,object>> list= new List<Dictionary<string,object>>;
list.Add(字典); // 条件列
db.Deleteable<object>().AS("[Order]").WhereColumns(list).ExecuteCommand();

// 插入
List<Dictionary<string,object>> list= new List<Dictionary<string,object>>;
list.Add(字典) // 插入所有要的所有列
db.Insertable(list).AS("student").ExecuteCommand();

// 更新
List<Dictionary<string,object>> list= new List<Dictionary<string,object>>;
list.Add(字典) // 更新和条件所需要的所有列
var t66 = db.Updateable(list).AS("student").WhereColumns("id").ExecuteCommand();
```

### 特殊 SQL 批量（5.0.6.3）

```csharp
using (db.Ado.OpenAlways()) { // 开启长连接
    foreach(var pars in List<parameter[]>)
    {
        db.Ado.ExecuteCommand(sql,paras);
    }
}
```

## 其他说明

### 表值参数、游标参数

- SqlServer：查表值参数
- Oracle：查表游标参数

### Dapper 用户迁移

从 Dapper 迁移时，按上述参数和映射思路平移。

### DataSet 转 `List<T>`

`SqlQuery<T,T2,T3..>` 最多只能有 7 个结果集；超过 7 个时可改用 `db.Ado.GetDataSetAll`。

```csharp
var datables=ds.Tables.Cast<DataTable>().ToList();
foreach (var item in tables)
{
    var list = db.Utilities.DataTableToList<Order>(item);
}
```

### 类作为参数

默认仅支持匿名对象。普通类需自行封装：普通类会把所有属性都转成参数。

```csharp
var pars=new{id=1;name="a"}; // 匿名类直接支持

var obj = new Order(){ .... }; // 类对象
SugarParameter[]  pars=obj.GetType().GetProperties()
    .Select(it => new SugarParameter(it.Name, it.GetValue(obj))).ToArray();
```
