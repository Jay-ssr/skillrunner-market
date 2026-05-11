# SqlSugar 已废弃 API 与迁移指南

> 汇总源码中所有 `[Obsolete]` 标记，帮助升级时快速定位和替换。

## 废弃 API 一览

### 1. Saveable → Storageable

**影响范围**: `ISqlSugarClient`, `SqlSugarClient`, `SqlSugarScope`

| 废弃 API | 替代方案 | 说明 |
|----------|----------|------|
| `db.Saveable<T>(List<T> list)` | `db.Storageable(list)` | 批量保存 |
| `db.Saveable<T>(T data)` | `db.Storageable(data)` | 单条保存 |
| `ISaveable<T>` 接口 | `IStorageable<T>` | 差异化存储，功能更强大 |

**迁移步骤**:
```csharp
// 旧写法
db.Saveable(list).ExecuteCommand();

// 新写法 — 功能对等
db.Storageable(list).ExecuteCommand();

// 新写法 — 充分利用 Storageable 能力
var result = db.Storageable(list)
    .WhereColumns(x => x.Id)       // 指定匹配列
    .ToStorage();
result.AsInsertable.ExecuteCommand();
result.AsUpdateable.ExecuteCommand();
```

**为什么迁移**: Storageable 支持 SplitInsert/Update/Delete/Ignore/Error 五种拆分、事务锁、分表、BulkCopy 等高级特性，功能远超 Saveable。

---

### 2. SqlFunc.CharIndex → SqlFunc.CharIndexNew

**影响范围**: `SqlFunc` 静态类

| 废弃 API | 替代方案 | 说明 |
|----------|----------|------|
| `SqlFunc.CharIndex(string, string)` | `SqlFunc.CharIndexNew(...)` | 参数顺序在不同数据库下不一致 |

**迁移步骤**:
```csharp
// 旧写法 — 参数顺序在不同数据库下含义不同
var q = db.Queryable<Order>()
    .Where(o => SqlFunc.CharIndex("abc", o.Name) > 0);

// 新写法 — 参数语义明确
var q = db.Queryable<Order>()
    .Where(o => SqlFunc.CharIndexNew(o.Name, "abc") > 0);
```

---

### 3. FiledNameSql → FieldNameSql

**影响范围**: `UtilMethods` 静态类

| 废弃 API | 替代方案 | 说明 |
|----------|----------|------|
| `UtilMethods.FiledNameSql()` | `UtilMethods.FieldNameSql()` | 旧名称拼写错误（Filed → Field） |

---

### 4. InitKeyType.SystemTable → 实体配置

**影响范围**: `InitKeyType` 枚举

| 废弃值 | 替代方案 | 说明 |
|--------|----------|------|
| `InitKeyType.SystemTable = 0` | 在实体类上使用 `[SugarTable]` 配置 | 不再推荐使用 SystemTable 模式 |

**迁移步骤**:
```csharp
// 旧写法 — 系统表模式
db.CodeFirst.InitTables(typeof(Order));  // InitKeyType.SystemTable

// 新写法 — 显式实体配置
[SugarTable("t_order")]
public class Order
{
    [SugarColumn(IsPrimaryKey = true, IsIdentity = true)]
    public int Id { get; set; }
}
```

---

### 5. SubInsertable.ExecuteReturnPrimaryKey → ExecuteCommand

**影响范围**: `ISubInsertable<T>`, `SubInserable` 实现

| 废弃 API | 替代方案 | 说明 |
|----------|----------|------|
| `ExecuteReturnPrimaryKey()` | `ExecuteCommand()` | 子表插入不再返回主键 |
| `ExecuteReturnPrimaryKeyAsync()` | `ExecuteCommandAsync()` | 异步版本 |

---

### 6. SelectFieldModel.FiledName → FieldName

**影响范围**: `SelectFieldModel` 类（Json2Sql 场景）

| 废弃属性 | 替代方案 | 说明 |
|----------|----------|------|
| `SelectFieldModel.FiledName` | `SelectFieldModel.FieldName` | 拼写错误修正 |

---

### 7. RewritableMethods → SqlSugarClient.Utilities

**影响范围**: `SqlSugarProvider`

| 废弃属性 | 替代方案 | 说明 |
|----------|----------|------|
| `SqlSugarProvider.RewritableMethods` | `SqlSugarClient.Utilities` | 通过客户端的 Utilities 属性访问 |

---

## 版本升级检查清单

升级 SqlSugar 时，按以下步骤排查：

1. **全局搜索** `Saveable` → 替换为 `Storageable`
2. **全局搜索** `SqlFunc.CharIndex(` → 确认参数顺序，替换为 `CharIndexNew`
3. **全局搜索** `FiledNameSql` → 替换为 `FieldNameSql`
4. **全局搜索** `InitKeyType.SystemTable` → 使用实体配置替代
5. **全局搜索** `ExecuteReturnPrimaryKey` → 替换为 `ExecuteCommand`
6. **全局搜索** `.FiledName` → 确认是否为 `SelectFieldModel`，替换为 `.FieldName`

## 注意事项

- 所有 `[Obsolete]` 标记在当前版本仍然可用，但未来版本可能移除
- 建议在编译时启用 `CS0618`（Obsolete 警告）作为升级提醒
- `Saveable` 是功能差距最大的废弃项，建议优先迁移
