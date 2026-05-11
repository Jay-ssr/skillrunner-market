# Aspire 集成

## 必要包

- Aspire.Npgsql
- SqlSugarCoreNoDrive

## 配置使用 Aspire.Npgsql 自带的数据库追踪

```csharp
// Program.cs
builder.AddNpgsqlDataSource("platformAdminConnStr");

// ServiceExtensions 中统一调用中心化扩展
services.AddSqlSugarRedisCache();
services.AddPostgreSqlSugarDbContext<IDeleted>(
    configuration,
    "platformAdminConnStr",
    x => x.IsDeleted == false);
```

### 为什么需要预先注入 Ado.Connection

`PostgreSQLProvider.Connection` 的 getter 在首次访问时懒加载：

```csharp
// SqlSugar 源码 - PostgreSQLProvider.cs
public override IDbConnection Connection
{
    get
    {
        if (base._DbConnection == null)
        {
            // 直接从连接字符串 new，绕过了 NpgsqlDataSource
            base._DbConnection = new NpgsqlConnection(
                base.Context.CurrentConnectionConfig.ConnectionString);
        }
        return base._DbConnection;
    }
}
```

如果不预先注入，SqlSugar 会自行 `new NpgsqlConnection(connectionString)`，导致：

- **Aspire.Npgsql 的 OpenTelemetry 连接追踪丢失**
- **NpgsqlDataSource 的连接指标（池大小、等待时间）不生效**
- **连接复用失效**

因此在构造回调里执行 `sqlSugar.Ado.Connection ??= connectionFactory()`：

- 有 `NpgsqlDataSource` 时 → `dataSource.CreateConnection()`，复用 Aspire 的追踪链路
- 无 `NpgsqlDataSource` 时 → `new NpgsqlConnection(connectionString)`，降级到普通建连

详见 `SqlSugarServiceCollectionExtensions.cs`。
