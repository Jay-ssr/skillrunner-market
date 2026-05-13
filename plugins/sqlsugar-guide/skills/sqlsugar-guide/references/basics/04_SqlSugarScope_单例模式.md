# SqlSugarScope 单例模式

> 单例模式的错误写法汇总详见 `runtime/17_单例模式.md`，本文件侧重 IOC 注册和基础用法。

## 完整用例

`SqlSugarScope` 应使用单例。配置分两类：
- **全局生效**：在 `new SqlSugarScope(config, db => { db.xxx = yyyy; })` 中设置，对所有上下文生效
- **当前上下文生效**：直接在当前上下文里设置 `db.xxx = yyy;`，只影响当前上下文

```csharp
using SqlSugar;

// 建库
SqlSugarHelper.Db.DbMaintenance.CreateDatabase(); // 达梦和 Oracle 不支持建库

// 建表请看 Code First 专题
SqlSugarHelper.Db.CodeFirst.InitTables<Student>(); // 所有库都支持

// 查询
var list = SqlSugarHelper.Db.Queryable<Student>().ToList();

// 插入
SqlSugarHelper.Db.Insertable(new Student() { SchoolId = 1, Name = "jack" }).ExecuteCommand();

// 更新
SqlSugarHelper.Db.Updateable(new Student() { Id = 1, SchoolId = 2, Name = "jack2" }).ExecuteCommand();

// 删除
SqlSugarHelper.Db.Deleteable<Student>().Where(it => it.Id == 1).ExecuteCommand();

public class SqlSugarHelper // 不能是泛型类
{
    // 多库说明：
    // - 固定多库：可传 new SqlSugarScope(List<ConnectionConfig>, db => {})，参考多租户文档
    // - 非固定多库：改用多租户专题里的动态切库方案
    public static SqlSugarScope Db = new SqlSugarScope(new ConnectionConfig()
    {
        ConnectionString = "datasource=demo.db",
        DbType = DbType.Sqlite,
        IsAutoCloseConnection = true // 不设为 true 需手动 close
    },
    db => {
        // 全局生效配置点；AOP 和程序启动配置通常放这里
        db.Aop.OnLogExecuting = (sql, pars) =>
        {
            // 获取原生 SQL，推荐 5.1.4.63
            Console.WriteLine(UtilMethods.GetNativeSql(sql, pars));

            // 获取无参数化 SQL，对性能有影响；特别是大 SQL / 多参数场景，仅建议调试使用
            // Console.WriteLine(UtilMethods.GetSqlString(DbType.SqlServer, sql, pars))
        };

        // 其他全局配置可继续写// db.Ado.IsDisableMasterSlaveSeparation = true;

        // 多租户场景需分别配置
        // db.GetConnection(i).Aop
    });
}

// 实体应与数据库结构保持一致
public class Student
{
    [SugarColumn(IsPrimaryKey = true, IsIdentity = true)]
    public int Id { get; set; }
    public int? SchoolId { get; set; }
    public string? Name { get; set; }
}
```

### 验证单例是否成功

```csharp
SqlSugarHelper.Db.HasCode(); // 服务启动后若 HasCode 一直相同，说明单例生效
// IOC 场景按官方 DEMO 注册通常不需要额外验证
```

## 单例模式使用 IOC

单例场景使用 `AddSingleton` 注册 `SqlSugarScope`。

```csharp
// 注册上下文：AOP 中可获取 IOC 对象；如框架已提供可省略
services.AddHttpContextAccessor();

// 注册 SqlSugar
services.AddSingleton<ISqlSugarClient>(s =>
{
    SqlSugarScope sqlSugar = new SqlSugarScope(new ConnectionConfig()
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
