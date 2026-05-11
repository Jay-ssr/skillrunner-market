# JSON 类型处理

### 基本用法

多表查询用到 ViewModel 时也需要 IsJson 特性。

即使数据库不支持 JSON 类型，SqlSugar 也能支持 JSON。

```csharp
public class UnitJsonTest
{
    [SqlSugar.SugarColumn(IsPrimaryKey = true, IsIdentity = true)]
    public int Id { get; set; }
    [SqlSugar.SugarColumn(IsJson = true)] // 必填
    public Order Order { get; set; } // 可以是数组或者JArray JObject
    public string Name { get; set; }
}

Db.Insertable(new UnitJsonTest()
{ Name = "json1", Order = new Order { Id = 1, Name = "order1" } }).ExecuteCommand();

var list = Db.Queryable<UnitJsonTest>().ToList();
```

### 动态Json

实体用 JObject 和 JArray 实现动态类型，支持任何可序列化的字典类型。

```csharp
[SqlSugar.SugarColumn(IsJson = true)]
public JObject Order { get; set; }

JObject o = JObject.FromObject(new { id = 1 });
JArray o2 = JArray.FromObject(new List<object> { new { id = 1 } });
// 只要能序列化都支持
```

5.1.4.68 版本支持 System.Text.Json 对象。

### JSON函数 5.1.2.8+

支持 JObject、JArray、实体、集合等类型。

```csharp
[SugarColumn(IsJson = true)] // 添加特性
public List<Order> JsonObj { get; set; }
```

| 函数名 | 说明 | 兼容 |
|--------|------|------|
| SqlFunc.JsonLike | 模糊查询，兼容所有库，性能一般适合小数据处理<br>`SqlFunc.JsonLike(it.JsonObj, "a")` 等于 Like '%a%' | 支持所有数据库 |
| SqlFunc.JsonField | 注意：大小写要一样<br>(1)查询Id的值{id:1}<br>`SqlFunc.JsonField(it.JsonObj, "id")` 返回1<br>(2)多层级查询，查询id的值{obj:{id:"a"}}<br>`SqlFunc.JsonField(it.JsonObj, "obj", "id")` 返回a | 支持 PostgreSQL<br>支持 SqlServer2017+<br>支持 MySql (只能字段)<br>支持 Oracle (只能字段)<br>Sqlite升级到：5.1.4.148 支持 Sqlite |
| SqlFunc.JsonIndex | 需要升级到:5.1.4.113+<br>获取json数组的索引对象<br>`SqlFunc.JsonIndex(it.JsonArray, 0)` 如果是['a','c']那么返回a | PostgreSQL<br>MySql<br>SqlServer2017 |
| SqlFunc.JsonParse | 转成JSON类型 | 支持PostgreSQL |
| SqlFunc.JsonContainsFieldName | 第一层是否存在id（如果多层级需要结合JsonField）<br>`SqlFunc.JsonField("{id:1}", "id")` true | 支持PostgreSQL |
| SqlFunc.JsonArrayAny | 需要升级到：5.1.3.36+<br>["a","b","c"]这种数组里面是否存在字符串a<br>注意：如果是数字可以1或者"1"都试一下<br>也就是[1,2] 参数用int类型<br>也就是["1","2"]参数用string类型<br>注意：如果是多个值可以结合【动态表达式】<br>https://www.donet5.com/Ask/9/26342 | 支持PostgreSQL<br>支持MySql<br>支持SqlServer(最新版本) |
| SqlFunc.JsonListObjectAny | 需要升级到：5.1.3.36+<br>[{"name":"a"},{"name":"b"}]集合中是否存在name=a的一项<br>注意：如果是数字可以1或者"1"2种类型参数都试一下<br>注意：如果是多个值可以结合【动态表达式】<br>https://www.donet5.com/Ask/9/26342 | 支持PostgreSQL<br>支持MySql<br>支持SqlServer(最新版本) |
| SqlFunc.JsonArrayLength | [1,2,3]获取数组长度 | 支持PostgreSQ<br>SqlSugar 5.1.4.115<br>支持MySql<br>支持SqlServer |

### 使用场景

保存用户附带信息，免额外建表。

### 自定义JSON序列化

使用自定义类型转换方式实现 JSON，兼容 `SqlFunc.Json*` 系列函数。
