---
name: sqlsugar-guide
user-invocable: false
description: |
  SqlSugar ORM 参考知识库，系统自动参考。
  触发：涉及 SqlSugar 的操作/问题/报错/迁移，或 C#/.NET 数据库相关话题。
  按需读取 references/ 下最小相关文件集，禁止全目录加载。
---

# SqlSugar 知识库

> 系统自动参考，用户不可调用。

## 定位流程

1. 按主题目录定位：basics、query、write、dynamic、runtime、database
2. 按「高价值入口」精确读取 1-2 个文件
3. 跨主题时才读多个文件，优先高价值入口

## Topic Routing

| 目录 | 主题 |
|------|------|
| `basics/` | 安装、SqlSugarClient/Scope、连接配置/池、实体映射、ValueObject、鉴别器、EntityMaintenance、DbFirst、CodeFirst、表结构管理 |
| `query/` | Where、Select、分页、排序、分组/去重、报表、SqlFunc、联表、子查询、导航查询、树型、并集、动态条件、跨库 |
| `write/` | 插入、更新、删除、保存、批量写入、导入/验证、雪花ID、导航增删改 |
| `dynamic/` | 无实体 CRUD、Ado 原生 SQL、动态类、Json2Sql、字符串表达式 |
| `runtime/` | 事务、锁、异步、并发、多库、缓存、读写分离、分表、多租户、IOC、仓储、UOW、AOP、查询过滤器、SQL注入防护、Aspire |
| `database/` | 厂商特有行为、通用配置、类型映射、JSON/枚举处理、自定义转换 |

## 高价值入口

### 架构选型
- SqlSugarClient vs SqlSugarScope → `basics/02` → 按需加 `03` 或 `04`
- 单例模式 → `runtime/17`（含错误写法）；IOC 示例可加 `basics/04`
- IOC 注入 → `runtime/18`

### 实体建模
- 实体配置：`basics/07`(内置) → `08`(自定义) → `09`(无特性) → `10`(完整示例)
- ValueObject/鉴别器/实体管理：`basics/11`、`12`、`13`

### 代码生成
- CodeFirst → `basics/17` → 按需加 `18`(配置)、`19`(表结构)、`20`(数据库差异)
- DbFirst → `basics/14` → 按需加 `15`(Razor)、`16`(自定义生成器)

### 查询
- 基础 `01`、条件 `02`、投影 `03`、分页 `04`、函数 `08`
- 联表 `12`、导航 `15`、子查询 `13`

### 写入
- 插入 `01`、更新 `02`、删除 `03`、批量 `05`、保存 `04`

### 故障排查
- 偶发错误/线程安全 → `runtime/06`
- 常见报错 → `runtime/23`；版本要求 → `runtime/24`
- 数据库连接报错 → 对应 `database/XX` 的"注意事项"

### AOP/过滤器
- AOP+日志拦截 → `runtime/25`
- 查询过滤器 → `runtime/26`（与 `query/21` 重复，读一个即可）

## 冗余文件（只读一个）
- `runtime/17` ↔ `basics/04` — 单例读 `runtime/17`，IOC 示例读 `basics/04`
- `runtime/26` ≈ `query/21` — 优先 `runtime/26`
- `runtime/25` 末尾含过滤器概述，详细仍读 `runtime/26`

## API 模式速查

- `CodeFirst|InitTables|MoreSettings` → basics/
- `Queryable|Where|Select|Join|Subqueryable|Includes` → query/
- `Insertable|Updateable|Deleteable|Storageable|Fastest` → write/
- `Ado|SqlQuery|ExecuteCommand|DynamicBuilder` → dynamic/
- `BeginTran|UseTran|Tenant|SplitTable|Aop|QueryFilter` → runtime/

## 回答规则

- 保留 SqlSugar 类型名/API 名原样，不翻译不缩写
- 代码示例用 C#，遵循官方命名风格
- 依赖数据库引擎时标注引用的数据库文件
- 跨主题基于最小必要文件作答
- 信息不足时说明，不编造 API
- 版本敏感功能标注最低版本（如 `5.1.4.63+`）
- 报错优先引导至 `runtime/06` + `runtime/23`
