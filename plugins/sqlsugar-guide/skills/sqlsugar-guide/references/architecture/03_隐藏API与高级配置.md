# SqlSugar 隐藏 API 与高级配置

> 源码中暴露但官方文档未详述的接口和配置项。

## StaticConfig — 全局隐藏选项

文件：`Infrastructure/StaticConfig.cs`

```csharp
// 每次创建 Provider 后回调 — 可注入 AOP、配置默认值
StaticConfig.CompleteDbFunc = provider => {
    provider.Aop.OnLogExecuting = (sql, pars) => Console.WriteLine(sql);
};

// 每次创建 Queryable/Insertable/Updateable/Deleteable 后回调
StaticConfig.CompleteQueryableFunc = queryable => { /* ... */ };
StaticConfig.CompleteInsertableFunc = insertable => { /* ... */ };
StaticConfig.CompleteUpdateableFunc = updateable => { /* ... */ };
StaticConfig.CompleteDeleteableFunc = deleteable => { /* ... */ };
```

| 属性 | 用途 |
|---|---|
| `CompleteDbFunc` | Provider 创建后回调，可统一注入 AOP |
| `CompleteQueryableFunc` | Queryable 创建后回调 |
| `CompleteInsertableFunc` | Insertable 创建后回调 |
| `CompleteUpdateableFunc` | Updateable 创建后回调 |
| `CompleteDeleteableFunc` | Deleteable 创建后回调 |
| `CustomSnowFlakeFunc` | 完全替换雪花 ID 生成器 |
| `CustomSnowFlakeTimeErrorFunc` | 处理雪花 ID 时钟漂移 |
| `CustomGuidFunc` | 替换 GUID 生成 |
| `CustomGuidByValueFunc` | 转换 GUID（如顺序 GUID） |
| `SplitTableGetTablesFunc` | 替换分表发现逻辑（避免查系统表） |
| `SplitTableCreateTableFunc` | 替换分表创建逻辑 |
| `CacheRemoveByLikeStringFunc` | 替换缓存清除策略（适配 Redis） |
| `BulkCopy_MySqlCsvPath` | MySQL 批量复制 CSV 临时路径 |
| `QueryOneToOneEnableDefaultValue` | OneToOne 查询启用默认值 |
| `EnableAllWhereIF` | WhereIF 条件为 false 时仍生成 SQL（用于预览） |
| `CodeFirst_MySqlCollate` | MySQL CodeFirst 排序规则 |
| `CodeFirst_MySqlTableEngine` | MySQL 表引擎（如 InnoDB） |
| `Encode` / `Decode` | 全局字符串加解密函数 |
| `DynamicExpressionParserType` | 自定义动态表达式解析器类型 |
| `EnableAot` | 启用 AOT 编译支持 |

## SqlMiddle — SQL 执行中间件

在 `ConnectionConfig` 中，`SqlMiddle` 提供完整的 SQL 拦截层：

```csharp
var db = new SqlSugarClient(new ConnectionConfig {
    // ...
    SqlMiddle = new SqlMiddle {
        GetScalar = (sql, pars) => { /* 完全替换标量查询 */ },
        ExecuteCommand = (sql, pars) => { /* 完全替换命令执行 */ },
        GetDataReader = (sql, pars) => { /* 完全替换 DataReader */ },
        GetDataSetAll = (sql, pars) => { /* 完全替换 DataSet */ },
        // + 异步版本
    }
});
```

用途：代理数据库、多库路由、审计、跨库中间件。

## AOP 完整事件列表

文件：`Entities/ConnectionConfig.cs`

| 事件 | 签名 | 用途 |
|---|---|---|
| `OnLogExecuting` | `Action<string, SugarParameter[]>` | SQL 执行前（查看 SQL） |
| `OnLogExecuted` | `Action<string, SugarParameter[]>` | SQL 执行后 |
| `OnExecutingChangeSql` | `Func<string, SugarParameter[], KeyValuePair<string, SugarParameter[]>>` | **执行前改写 SQL** |
| `OnDiffLogEvent` | `Action<DiffLogModel>` | 审计日志（增删改前后值） |
| `OnError` | `Action<SqlSugarException>` | 全局错误处理 |
| `DataExecuting` | `Action<object, DataFilterModel>` | 数据变更前拦截（实体级别） |
| `DataChangesExecuted` | `Action<object, DataFilterModel>` | 数据变更后 |
| `DataExecuted` | `Action<object, DataAfterModel>` | 实体物化完成后 |
| `CheckConnectionExecuting` | `Action<IDbConnection>` | 连接检查前 |
| `CheckConnectionExecuted` | `Action<IDbConnection, TimeSpan>` | 连接检查后（含耗时） |
| `OnGetDataReadering` | `Action<string, SugarParameter[]>` | DataReader 获取前 |
| `OnGetDataReadered` | `Action<string, SugarParameter[], TimeSpan>` | DataReader 获取后（含耗时） |

## ConfigureExternalServices — 服务替换

| 服务 | 默认值 | 可替换为 |
|---|---|---|
| `SerializeService` | Newtonsoft.Json | System.Text.Json 等 |
| `ReflectionInoCacheService` | 内存字典 | Redis / 分布式缓存 |
| `DataInfoCacheService` | null（无缓存） | Redis 查询结果缓存 |
| `RazorService` | 内置 Razor | DbFirst 自定义模板 |
| `SplitTableService` | DateSplitTableService | 自定义分表策略 |
| `SqlFuncServices` | 空列表 | 自定义 SQL 函数（见下方） |
| `EntityService` | null | 运行时覆盖列映射 |
| `EntityNameService` | null | 运行时覆盖表名 |
| `AppendDataReaderTypeMappings` | 空列表 | 自定义 CLR → DbType 映射 |

## SqlFuncExternal — 自定义 SQL 函数

在 lambda 表达式中注册自定义 SQL 函数：

```csharp
var db = new SqlSugarClient(new ConnectionConfig {
    ConfigureExternalServices = new ConfigureExternalServices {
        SqlFuncServices = new List<SqlFuncExternal> {
            new SqlFuncExternal {
                UniqueMethodName = "MyCustomFunc",
                MethodValue = (model, dbType, context) => {
                    return $"CUSTOM_FUNC({model.Args[0]})";
                }
            }
        }
    }
});

// 然后在代码中定义静态方法并在 lambda 中使用
```

## ConnMoreSettings — 隐藏连接选项

| 设置 | 用途 |
|---|---|
| `IsAutoRemoveDataCache` | CUD 后自动清除缓存 |
| `IsWithNoLockQuery` | SQL Server 加 WITH(NOLOCK) |
| `DisableWithNoLockWithTran` | 事务内禁用 NOLOCK |
| `DisableNvarchar` | 用 VARCHAR 替代 NVARCHAR |
| `PgSqlIsAutoToLower` | PostgreSQL 自动小写标识符 |
| `EnableILike` | PostgreSQL 用 ILIKE 替代 LIKE |
| `TableEnumIsString` | 枚举存为字符串 |
| `EnableCodeFirstUpdatePrecision` | CodeFirst 更新列精度 |
| `SqliteCodeFirstEnableDropColumn` | SQLite 启用 DROP COLUMN |
| `EnableOracleIdentity` | Oracle 12c+ 用 IDENTITY 替代序列 |
| `EnableJsonb` | PostgreSQL 用 JSONB 替代 JSON |
| `IsAutoUpdateQueryFilter` | Schema 变更自动更新过滤器 |
| `EnableModelFuncMappingColumn` | 函数列映射 |
| `DatabaseModel` | 覆盖数据库模型类型 |
| `DefaultCacheDurationInSeconds` | 全局默认缓存时长 |
| `DbMinDate` | 数据库最小有效日期 |
| `PostgresIdentityStrategy` | PostgreSQL PK 策略：Serial 或 Identity |

## QueryFilterProvider 高级特性

```csharp
// FilterJoinPosition 控制过滤器加在 JOIN ON 还是 WHERE
db.QueryFilter.AddTableFilter<User>(u => u.IsDeleted == false,
    FilterJoinPosition.On);  // LEFT JOIN 时保留 NULL 行

// 临时移除和恢复过滤器
var backup = db.QueryFilter.ClearAndBackup();
// ... 执行不带过滤器的查询 ...
db.QueryFilter.Restore(backup);

// 条件注册
db.QueryFilter.AddTableFilterIF(isAdmin == false, 
    u => u.TenantId == currentTenantId);
```
