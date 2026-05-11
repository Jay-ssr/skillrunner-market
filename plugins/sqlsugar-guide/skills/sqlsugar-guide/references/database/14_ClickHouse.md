# ClickHouse

- **NuGet**: `SqlSugar.ClickHouseCore` + `SqlSugarCore`

- **DbType**: `DbType.ClickHouse`

## 连接字符串
```csharp
host=localhost;port=8123;user=default;password=;database=default
```

## 初始化
```csharp
//程序启动时加入
InstanceFactory.CustomAssemblies = new System.Reflection.Assembly[] {
    typeof(ClickHouseProvider).Assembly
};

SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = DbType.ClickHouse,
    ConnectionString = "host=localhost;port=8123;user=default;password=;database=default",
    IsAutoCloseConnection = true
});
```

## 限制
1. 大小写要和数据库一模一样
2. 不支持事务
3. 只支持 Linux

## 大数据写入 BulkCopy
```csharp
//同步
db.Fastest<DC_Scene>().BulkCopy(lstData);

//高并发异步
await db.CopyNew().Fastest<DC_Scene>().BulkCopyAsync(lstData);
```

## 数组类型
```csharp
[SugarColumn(ColumnDataType = "Array(UInt64)", IsArray = true)]
public UInt64[] Text { get; set; }
```

## 建表自定义引擎
```csharp
[SqlSugar.ClickHouse.CKTable(@"engine = MergeTree PARTITION BY toYYYYMM(dt)
ORDER BY(toYYYYMM(dt))
SETTINGS index_granularity = 8192;")]
public class CKTest
{
    public string Id { get; set; }
    public DateTime dt { get; set; }
}
```

## 发布缺少 DLL
```csharp
//程序启动时加入
InstanceFactory.CustomAssemblies = new System.Reflection.Assembly[] {
    typeof(ClickHouseProvider).Assembly
};
```
