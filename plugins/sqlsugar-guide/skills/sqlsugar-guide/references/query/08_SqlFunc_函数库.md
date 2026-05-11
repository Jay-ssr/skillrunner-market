# SqlFunc 函数库

### C# 函数支持

```csharp
.ToString, .Contains, .Length, .ToLower, .ToUpper, .ToSubstring
.Equals, .HasValue, .Replace, .EndsWith, .StartsWith, .Trim
.HasValue, .Value, .AddDays, .AddHours, Convert.ToInt32 等
三元 ??, 时间.DayOfWeek, 时间.Date, 时间.Day 等
```

### 逻辑函数

```csharp
// IIF 三元判断
SqlFunc.IIF(it.Id == 1, 1, 2)  // it.id==1 ? 1 : 2

// Case When
SqlFunc.IF(st.Id > 1)
    .Return(st.Id)
    .ElseIF(st.Id == 1)
    .Return(st.SchoolId)
    .End(st.Id)

// IsNull
SqlFunc.IsNull(it.Id, 0)  // null 则为 0
```

### 时间函数

```csharp
// 时间格式化 (5.0.51+)
it.CreateTime.ToString("yyyy-MM-dd")

// 数据库时间
SqlFunc.GetDate()

// 是否同一天
SqlFunc.DateIsSame(it.CreateTime, DateTime.Now)

// 日期部分比较
SqlFunc.DateIsSame(DateTime date1, DateTime date2, DateType dataType)
// DateType: Year, Month, Day, Hour, Minute, Second, Millisecond

// 日期加
SqlFunc.DateAdd(DateTime date, int addValue, DateType dataType)
SqlFunc.DateAdd(DateTime date, int addValue)  // 加 N 天

// 获取部分
SqlFunc.DateValue(DateTime.Now, DateType.Day)

// DateDiff
SqlFunc.DateDiff(DateType.Day, DateTime.Now, DateTime.Now.AddDays(1))

// 周几
SqlFunc.DateValue(DateTime.Now, DateType.Weekday)

// 一年中第几周
SqlFunc.WeekOfYear

// Unix 时间戳 (5.1.4.194-preview35+)
SqlFunc.UNIX_TIMESTAMP(it.Date)
```

### 聚合函数

```csharp
SqlFunc.AggregateSum(it.num)           // 求和
SqlFunc.AggregateSumNoNull(it.num)      // 求和（过滤 null）
SqlFunc.AggregateAvg(it.num)           // 平均值
SqlFunc.AggregateAvgNoNull(it.num)     // 平均值（过滤 null）
SqlFunc.AggregateMin(it.num)           // 最小
SqlFunc.AggregateMax(it.num)           // 最大
SqlFunc.AggregateCount(it.num)         // 统计数量
SqlFunc.AggregateDistinctCount(it.num) // 去重统计
```

### 格式转换

```csharp
// 四舍五入 (5.0.4)
SqlFunc.Round(it.Price, 2)

// 绝对值 (5.0.4)
SqlFunc.Abs(it.Price)

// 字符串截取
SqlFunc.Substring(object value, int index, int length)

// 替换
SqlFunc.Replace(object value, string oldChar, string newChar)

// 大小写
SqlFunc.ToLower(object thisValue)
SqlFunc.ToUpper(object thisValue)

// 去空格
SqlFunc.Trim(object thisValue)

// 类型转换
SqlFunc.ToInt32(object value)
SqlFunc.ToInt64(object value)
SqlFunc.ToDate(object value)
SqlFunc.ToString(object value)
SqlFunc.ToDecimal(object value)
SqlFunc.ToGuid(object value)
SqlFunc.ToDouble(object value)
SqlFunc.ToBool(object value)

// 指定位置替换
SqlFunc.Stuff(string sourceString, int start, int length, string AddString)
```

### Bool 返回函数

```csharp
// NULL 判断
SqlFunc.IsNullOrEmpty(object thisValue)
SqlFunc.HasValue(object thisValue)

// 按位运算
SqlFunc.BitwiseAnd
SqlFunc.BitwiseInclusiveOR

// 判断大于 0 且不为 NULL
SqlFunc.HasNumber(object thisValue)

// 范围判断
SqlFunc.Between(object value, object start, object end)
```

### 模糊查询

```csharp
// Contains LIKE %@p%
SqlFunc.Contains(string thisValue, string parameterValue)
it.Name.Contains("a")

// StartsWith LIKE @p%
SqlFunc.StartsWith(object thisValue, string parameterValue)

// EndsWith LIKE %@p
SqlFunc.EndsWith(object thisValue, string parameterValue)

// 处理通配符
var likeValue = db.Utilities.EscapeLikeValue("a%a");
```

### IN 操作

```csharp
// 非参数化 IN（数量无上限）
SqlFunc.ContainsArray(object[] thisValue, string parameterValie)
SqlFunc.ContainsArrayUseSqlParameters(object[] thisValue, string parameterValie)

// Contains 数组
.Where(it => 数组变量.Contains(it.Id));  // IN
.Where(it => !Array.Contains(it.Id));     // NOT IN

// 字符串类型
NameList.Contains(it.Name, true)  // true: nvarchar, false: varchar

// 多字段 IN (5.1.4.67-preview04)
Where(it => list.Any(s => s.Id == it.Id && s.Name == it.Name))

// 逗号分割
SqlFunc.SplitIn("1,2,3,4", "5")  // 不存在返回 false
```

### 开窗函数

```csharp
// 需要数据库支持
SqlFunc.RowCount()           // count(1) over()
SqlFunc.RowMax(it.num)       // max(num) over()
SqlFunc.RowMin(it.num)       // min(num) over()
SqlFunc.RowAvg(it.num)       // avg(num) over()
SqlFunc.RowNumber(it.Id)      // row_number() over(order by Id)
SqlFunc.RowNumber(it.Id, it.Name)  // partition by name order by Id
SqlFunc.Rank                  // rank() over()

// 倒序
SqlFunc.RowNumber(SqlFunc.Desc(it.Id))

// 多字段
SqlFunc.RowNumber($"{it.Id} asc, {it.Name} desc", $"{it.Name}, {it.Id}")
```

### JSON 函数

```csharp
// 需要 [SugarColumn(IsJson = true)]
public List<Order> JsonObj { get; set; }

// 模糊查询
SqlFunc.JsonLike(it.JsonObj, "a")

// 获取字段
SqlFunc.JsonField(it.JsonObj, "id")
SqlFunc.JsonField(it.JsonObj, "obj", "id")

// 获取索引
SqlFunc.JsonIndex(it.JsonArray, 0)

// 数组是否存在
SqlFunc.JsonArrayAny(it.Json, "a")
SqlFunc.JsonListObjectAny(it.json, "Name", "a")

// 数组长度
SqlFunc.JsonArrayLength(it.Json)

// 是否有字段名
SqlFunc.JsonContainsFieldName("{id:1}", "id")

// PostgreSQL 特有
SqlFunc.JsonParse
```

### 其他函数

```csharp
// 字符串拼接
SqlFunc.MergeString(...)

// 随机数
SqlFunc.GetRandom()

// 字符串长度
SqlFunc.Length(object value)

// 字符位置
SqlFunc.CharIndex

// 补全
SqlFunc.PadLeft(...)

// 向上/下取整
SqlFunc.Floor(..)
SqlFunc.Ceil(..)

// 随机数（同上）
```

---
