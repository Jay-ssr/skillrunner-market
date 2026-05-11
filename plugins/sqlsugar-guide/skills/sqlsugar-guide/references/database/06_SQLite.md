# SQLite

- **NuGet**: `SqlSugarCore`

- **DbType**: `DbType.Sqlite`

## 连接字符串
```csharp
//相对路径 - 推荐（ORM建库功能说明：建议不要加目录，ORM没办法创建文件夹）
Data Source=SqlSugar4xTest.sqlite;

//完整路径（这种方式ORM可以创建文件夹）
Data Source=c:\database\SqlSugar4xTest.sqlite;
```

## 初始化
```csharp
var db = new SqlSugarClient(new ConnectionConfig()
{
    DbType = SqlSugar.DbType.Sqlite,
    ConnectionString = "Data Source=SqlSugar4xTest.sqlite",
    IsAutoCloseConnection = true
});
```

## 注意事项

### 1. SQLite 加密

NuGet 安装：
- `SQLitePCLRaw.bundle_e_sqlcipher 2.0.4`
- `SqlSugarCore`（.NET Core）
- `SqlSugarCore_NetCore2`（支持 .Framework 4.7.2+）

```csharp
string dbName = Path.Combine(Environment.CurrentDirectory, "SampleDB.db");
string connStr = new SqliteConnectionStringBuilder()
{
    DataSource = dbName,
    Mode = SqliteOpenMode.ReadWriteCreate,
    Password = "admin"
}.ToString();

//性能优化：加密 Open 较慢，使用 SqlSugarClient 单例 + 关闭自动释放
public static SqlSugarClient Db = new SqlSugarClient(new ConnectionConfig()
{
    ConnectionString = connStr,
    DbType = DbType.Sqlite,
    IsAutoCloseConnection = false  //关闭自动释放
});

//如果有并发需要加Lock
lock (XXX.Db)
{
    XXX.Db.Insertable(data).ExecuteCommand();
}
```

### 2. .NET Framework 安装

Nuget 安装 System.Data.SQLite 失败时：
1. Nuget 安装 SqlSugar 或引用最新的 SqlSugar.dll
2. 解压下载文件包
3. System.Data.SQLite.dll 引用到项目
4. x86 x64 两个文件夹扔到 bin 下面的 dll 同一目录

### 3. 并行写入

```csharp
var rand = new Random();
for (int i = 0; i < 10000; i++)
{
    Task.Run(async () =>
    {
        await Task.Delay(rand.Next(1, 10000));
        await db.CopyNew()  //CopyNew保证线程安全
            .Updateable<Order>()
            .SetColumns(o => o.Name == "2")
            .Where(o => o.Id == 1)
            .ExecuteCommandAsync();
    });
}
```

### 4. 解除文件占用

```csharp
//.NET CORE
Microsoft.Data.Sqlite.SqliteConnection.ClearAllPools();
Microsoft.Data.Sqlite.SqliteConnection.ClearPool((SqliteConnection)db.Ado.Connection);

//.NET Framework
SQLiteConnection.ClearAllPools();

//也可以使用自带的备份库方法
db.DbMaintenance.BackupDataBase(null, "sql2014test222.db");
```

### 5. CodeFirst 默认值

请升级：SqlSugarCore 5.1.4.108-preview23+
```csharp
MoreSettings = new ConnMoreSettings()
{
    SqliteCodeFirstEnableDefaultValue = true  //启用默认值
}
```

```csharp
public class CodeFirstUnitafa
{
    [SugarColumn(DefaultValue = "(strftime('%Y-%m-%d %H:%M:%S', 'now', 'localtime'))")]
    public DateTime Name { get; set; }

    [SugarColumn(DefaultValue = "1")]
    public string Name2 { get; set; }
}
```

### 6. CodeFirst 备注

请升级：SqlSugarCore 5.1.4.108-preview25+
```csharp
MoreSettings = new ConnMoreSettings()
{
    SqliteCodeFirstEnableDescription = true  //启用备注
}
```

### 7. CodeFirst 删除列

请升级：SqlSugarCore 5.1.4.118-preview04+
```csharp
MoreSettings = new ConnMoreSettings()
{
    SqliteCodeFirstEnableDropColumn = true  //只支持 .net core
}
```

### 8. Memory 用法

```csharp
using (var db = new SqlSugarClient(new ConnectionConfig()
{
    IsAutoCloseConnection = false,  //内存方式需要手动释放
    DbType = DbType.Sqlite,
    ConnectionString = "Data Source=:memory:"
}))
{
    db.CodeFirst.InitTables<UserInfo002>();  //建表
    db.Insertable(userInfo).ExecuteCommand();
}
```

### 9. 常见错误

9.1 `database is locked`
- 使用 `db.Dispose` 替代 `db.Close`

9.2 `The type initializer for 'Microsoft.Data.Sqlite.SqliteConnection' threw an exception`
- 独立安装 Microsoft.Data.Sqlite，统一最新版本
- 发布 Linux 找不到文件报错，使用自包含发布，避免裁剪

#### 9.3 查询不到时间
- 清空存在错误格式时间的表，用 ORM 重新插入
