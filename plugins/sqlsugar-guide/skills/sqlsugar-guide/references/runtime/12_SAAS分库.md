# SAAS分库

## SAAS分库的优势

SAAS 分库成熟稳定，应优先选用。

**原理**：基于业务维度划分数据，适合多账户系统。

| 功能点 | SAAS分库 | 聚合并发分库 |
|--------|----------|-------------|
| 性能 | 100 | 90 |
| 稳定性和成熟 | 100 | 80（.NET中更低） |
| Sql语法兼容性 | 100 | 60 |
| 隔离数据之间的关联 | | |
| 不使用Sqlsugar | 40 | |
| 使用Sqlsugar | 80 | |
| | 100 | |
| 用户数据安全 | 100（用户可以独享库，互不干扰） | 80 |

SAAS 分库优势明显，唯一注意点：避免隔离数据之间存在交集。

## 数据库设计

### 基础信息库

存储组织架构、权限、字典、用户等公共信息。因基础信息库是共享的，可通过读写分离或二级缓存优化性能。

### 业务库

我们要进行的分库都基于业务库进行分库，例如 A集团使用 A01库，B集团使用B01库，也可以多个小集团使用一个数据库：

- **业务库1**：集团A VIP用户独享一个库
- **业务库2**：集团B, 集团F
- **业务库3**：集团C, 集团D, 集团E...........集团Z（小客户多人共享一个库）

**性能无瓶颈、可扩展**：合理分库后性能无瓶颈，数据库可部署到不同服务器。

## 表设计

### 数据库配置表

主键、数据库连接信息、集团ID（基础信息库）

### 用户表

主键、用户名、密码、集团ID（基础信息库）

### 禁止自增列

考虑到租户迁移，自增列禁止使用，可以用GUID和雪花ID。

## 代码编写

### 单例模式IOC实现

```csharp
// 注册上下文：AOP里面可以获取IOC对象，如果有现成框架比如Furion可以不写这一行
services.AddHttpContextAccessor();

// 注册SqlSugar
services.AddSingleton<ISqlSugarClient>(s =>
{
    SqlSugarScope sqlSugar = new SqlSugarScope(new ConnectionConfig()
    {
        DbType = SqlSugar.DbType.SqlServer,
        ConnectionString = "DataSource=sqlsugar-dev.db",
        IsAutoCloseConnection = true,
        ConfigId = "default"
    },
    db =>
    {
        // 单例参数配置，所有上下文生效
        db.Aop.OnLogExecuting = (sql, pars) =>
        {
            // 获取IOC作用域对象
            var appServive = s.GetService<IHttpContextAccessor>();
            var obj = appServive?.HttpContext?.RequestServices.GetService<Log>();
            Console.WriteLine("AOP" + obj.GetHashCode());
        };
    });
    return sqlSugar;
});
```

```csharp
// 创建DbManager
/// <summary>
/// 数据库管理
/// </summary>
public class DbManger
{
    /// <summary>
    /// 获取业务库对象（用IOC这块代码不能写到IOC里面）
    /// </summary>
    public static ISqlSugarClient BizDb
    {
        get
        {
            UserInfo user = GetUserInfo();  // 根据Token获取用户信息和连接字符串信息
            var configId = user.OrgId.ToString();  // 集团ID（也可以叫租户ID）

            if (!Db.IsAnyConnection(configId))  // 用非默认ConfigId进行测试
            {
                // 添加业务库只在当前上下文有效（原理：SqlSugarScope模式入门文档去看）
                Db.AddConnection(new ConnectionConfig() {
                    ConfigId = configId,
                    ConnectionString = "DataSource=" + user.Connection,
                    DbType = DbType.SqlServer,
                    IsAutoCloseConnection = true
                });
            }

            // 原理说明：IsAnyConnection、AddConnection和GetConnection都是Scope周期不同请求不会有影响
            var result = Db.GetConnection(configId);
            // 可以给业务库result设置AOP和过滤器
            return result;
        }
    }

    /// <summary>
    /// 获取基础信息库对象（用IOC这块代码不能写到IOC里面）
    /// </summary>
    public static ISqlSugarClient MasterDb
    {
        get
        {
            // 如果是跨服务器分库，也需要动态配置的，因为库的IP会变
            // 参考业务库用法
            return Db.GetConnection("default");
        }
    }

    /// <summary>
    /// 通过IOC获取注入的Db对象
    /// </summary>
    public static SqlSugarScope Db
    {
        // 如果用Furion就是App.GetService<ISqlSugarClient>();
        // 如果项目里还没有 IOC，先把运行时依赖注入配置好
        var ihttp = 你存储的Services.BuildServiceProvider().GetService<IHttpContextAccessor>();
        var obj = ihttp?.HttpContext?.RequestServices.GetService<ISqlSugarClient>();
        return (SqlSugarScope)obj;
    }

    /// <summary>
    /// 获取用户数据库连接字符串信息
    /// </summary>
    private static UserInfo GetUserInfo()
    {
        // 读数据库配置一般会有缓存，不然浪费性能
    }
}
```

**使用DbManager和事务**：

```csharp
// 使用用例，继承后直接使用（老版本var myBizDb=BizDB要写在事务外面声明）
public class OrderManger : DbManger
{
    public void Test()
    {
        try
        {
            Db.BeginTran();  // 用Db管理MasterDb和BizDb事务，支持跨库事务
            MasterDb.Insertable(xxx).ExecuteCommand();  // 操作基础信息库
            BizDb.Insertable(xxx).ExecuteCommand();  // 操作业务库
            Db.CommitTran();  // 统一事务
        }
        catch (System.Exception ex)
        {
            Db.RollbackTran();
            throw ex;
        }
    }
}
```

### 非单例模式IOC实现

```csharp
services.AddHttpContextAccessor();

// 注册SqlSugarClient，这里你可以用Options或其他方式配置
services.AddScoped<ISqlSugarClient>(sp =>
{
    return new SqlSugarClient(new ConnectionConfig
    {
        // 配置你的默认连接
        ConnectionString = "默认连接字符串",
        DbType = DbType.MySql,
        IsAutoCloseConnection = true
    });
});

// 注册ChildDbContext是一个自个新建的类
services.AddScoped<ChildDbContext>(sp =>
{
    var httpContextAccessor = sp.GetRequiredService<IHttpContextAccessor>();
    var sqlSugar = httpContextAccessor.HttpContext?.RequestServices.GetService<ISqlSugarClient>();
    var configId = "你的configId";  // 可以从请求上下文或配置中动态获取

    if (!sqlSugar.IsAnyConnection(configId))
    {
        sqlSugar.AddConnection(new ConnectionConfig
        {
            ConfigId = configId,
            ConnectionString = "你的连接字符串",
            DbType = DbType.MySql,
            IsAutoCloseConnection = true
        });
    }

    var childDb = sqlSugar.GetConnection(configId);
    // childDb设置过滤器或者AOP
    // childDb.Aop.xxx
    return new ChildDbContext
    {
        Context = childDb
    };
});

// IOC获取默认db和处理子db事务
ISqlSugarClient db;
db.AsTenant().BeginTran();  // 多库事务，一般是接口ISqlSugarClient使用

// 获取具体业务db用
ChildDbContext.Context
```

## 跨库查询

### 跨库同服务器

跨库查询要用BizDb进行查询，因为BizDb是多变的，而MasterDb是固定的：

```csharp
var List = BizDb.Queryable<Order>()
    .LeftJoin<Custom>((o, cus) => o.CustomId == cus.Id, "SQLSUGAR4XTEST.dbo.Custom")
    // .LeftJoin<YYY> ... 可以多个
    .Where(o => o.Id == 1)
    .Select((o, cus) => new ViewOrder { Id = o.Id, CustomName = cus.Name })
    .ToList();
```

生成的Sql：

```sql
SELECT [o].[Id] AS [Id], [cus].[Name] AS [CustomName]
FROM [Order] o
Left JOIN [SQLSUGAR4XTEST].[dbo].[Custom] cus
ON ([o].[CustomId] = [cus].[Id])
WHERE ([o].[Id] = @Id0)
```

> 注意：上面的例子是SqlServer跨库查询的用法，不同的数据库跨库查询用法不一样。

### 跨库不同服务器

因为基础信息库是固定的，所以可以把基础信息库同步到不同的服务器上，通过读写分离的方式，那么每个业务服务器都会有一个基础信息库了，然后再通过6.5.1的方式实现。

## 业务表创建

可以通过CodeFirst创建业务库和表。

## 表过滤

一个业务库中的表对应多个集团，那我们设计表的时候肯定有个OrgId或者租户ID进行数据表区别：

```csharp
public static ISqlSugarClient BizDb
{
    get
    {
        UserInfo user = GetUserInfo();  // 获取用户数据库连接字符串信息
        var configId = user.OrgId.ToString();  // 集团ID（也可以叫租户ID）

        if (!Db.IsAnyConnection(configId))
        {
            Db.AddConnection(new ConnectionConfig() {
                ConfigId = configId,
                ConnectionString = "DataSource=" + user.Connection,
                DbType = DbType.SqlServer,
                IsAutoCloseConnection = true
            });
        }

        var result = Db.GetConnection(configId);
        // 接口过滤器（继承接口的类都有效）请升级5.1.3.47
        db.QueryFilter.AddTableFilter<IDeleted>(it => it.OrgId == user.OrgId);
        return result;
    }
}
```

## 高安全性日志

通过差异日志拿到数据更变记录：

```csharp
db.Aop.OnDiffLogEvent = it =>
{
    // 操作前记录：包含字段描述、列名、值、表名、表描述
    var editBeforeData = it.BeforeData;
    // 操作后记录：包含字段描述、列名、值、表名、表描述
    var editAfterData = it.AfterData;
    var sql = it.Sql;
    var parameter = it.Parameters;
    var data = it.BusinessData;  // 这边会显示你传进来的对象
    var time = it.Time;
    var diffType = it.DiffType;  // enum insert、update and delete
};

db.Insertable(new Student() { Name = "beforeName" })
    .EnableDiffLogEvent()  // 注意需要加上启用日志
    .ExecuteReturnIdentity();
```
