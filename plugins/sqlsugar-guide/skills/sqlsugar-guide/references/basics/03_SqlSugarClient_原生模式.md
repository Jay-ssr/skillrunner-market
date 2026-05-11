# SqlSugarClient 原生模式

## 完整用例

`SqlSugarClient` 每次请求应 `new` 新对象，`db` 禁止跨上下文复用。IOC 使用 Scope 或瞬时注入。

```csharp
using SqlSugar;

// 创建数据库对象（用法类似 EF / Dapper，通过 new 保证线程安全）
SqlSugarClient Db = new SqlSugarClient(new ConnectionConfig()
{
    ConnectionString = "datasource=demo.db",
    DbType = DbType.Sqlite,
    IsAutoCloseConnection = true
},
db => {
    db.Aop.OnLogExecuting = (sql, pars) =>
    {
        // 获取原生 SQL，推荐 5.1.4.63
        Console.WriteLine(UtilMethods.GetNativeSql(sql, pars));

        // 获取无参数化 SQL，对性能有影响；特别是大 SQL / 多参数场景，仅建议调试使用
        // Console.WriteLine(UtilMethods.GetSqlString(DbType.SqlServer, sql, pars))
    };

    // 多租户场景需分别配置
    // db.GetConnection(i).Aop
});

// 建库
Db.DbMaintenance.CreateDatabase(); // 达梦和 Oracle 不支持建库
// 建表请看 Code First 专题
Db.CodeFirst.InitTables<Student>(); // 所有库都支持

// 查询
var list = Db.Queryable<Student>().ToList();

// 插入
Db.Insertable(new Student() { SchoolId = 1, Name = "jack" }).ExecuteCommand();

// 更新
Db.Updateable(new Student() { Id = 1, SchoolId = 2, Name = "jack2" }).ExecuteCommand();

// 删除
Db.Deleteable<Student>().Where(it => it.Id == 1).ExecuteCommand();

// 实体应与数据库结构保持一致
public class Student
{
    // 自增列需要 IsIdentity
    // 主键需要 IsPrimaryKey
    [SugarColumn(IsPrimaryKey = true, IsIdentity = true)]
    public int Id { get; set; }
    public int? SchoolId { get; set; }
    public string? Name { get; set; }
}
```

## 原生模式使用 IOC

Scope 场景使用 `SqlSugarClient`。

```csharp
// 注册上下文：AOP 中可获取 IOC 对象；如框架已提供可省略
services.AddHttpContextAccessor();

// 注册 SqlSugar，使用 AddScoped
services.AddScoped<ISqlSugarClient>(s =>
{
    SqlSugarClient sqlSugar = new SqlSugarClient(new ConnectionConfig()
    {
        DbType = SqlSugar.DbType.Sqlite,
        ConnectionString = "DataSource=sqlsugar-dev.db",
        IsAutoCloseConnection = true,
    },
    db =>
    {
        // 每次上下文都会执行

        // 获取 IOC 对象，不要求同一上下文
        // var log = s.GetService<Log>()

        // 获取 IOC 对象，要求同一上下文
        // var appServive = s.GetService<IHttpContextAccessor>();
        // var log = appServive?.HttpContext?.RequestServices.GetService<Log>();

        db.Aop.OnLogExecuting = (sql, pars) =>
        {
        };
    });
    return sqlSugar;
});

// 用接口接收
public class MyService(ISqlSugarClient db)
```

## 线程与偶发

详情：https://www.donet5.com/Home/Doc?typeId=1224
