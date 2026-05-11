---
name: sqlsugar-guide
user-invocable: false
description: |
  SqlSugar ORM 自动参考知识库。触发：涉及 SqlSugar/ORM/C#数据库操作的任何问题。
  覆盖：Client/Scope、实体建模(SugarColumn/Table/Navigate/ValueObject/鉴别器)、
  查询(Queryable/Where/Select/Join/Includes/导航/SqlFunc/ThenMapper/Expressionable 12表)、
  写入(Insertable/Updateable/Deleteable/Storageable/Fastest)、
  代码生成(DbFirst/CodeFirst)、运行时(事务/AOP/缓存/分表/多租户/读写分离/异步)、
  无实体CRUD/Ado/DynamicBuilder/Json2Sql、30+数据库适配、IOC/仓储/UOW、
  源码架构(类关系/调用链/隐藏API/接口体系/废弃API迁移)。
  读取 references/ 下最小相关文件集（避免上下文溢出）。
---

<!-- section:routing -->
## Topic → 目录映射

basics: 安装、入门、Client/Scope、连接配置/池、实体映射、ValueObject、鉴别器、EntityMaintenance、DbFirst、CodeFirst
query: Where、Select、分页、排序、分组/去重、报表、SqlFunc、联表、子查询、嵌套、导航、树型、并集、动态条件、ThenMapper
write: 插入、更新、删除、保存、批量、表保存、导入/验证、雪花ID、导航增删改
dynamic: 无实体CRUD、原生SQL/Ado、动态类、Json2Sql、字符串表达式、低代码
runtime: 事务、锁、异步、并发、多库、缓存、读写分离、分表、多租户、BI、分批优化、IOC、仓储、UOW、AOP、查询过滤器、SugarRetry
database: 各厂商特有行为、通用配置、类型映射、JSON/枚举处理、自定义转换
architecture: 源码架构、方法调用链、隐藏API、接口体系(29接口)、已废弃API迁移
<!-- /section:routing -->

<!-- section:lookup -->
## 关键词 → 目录速查

`CodeFirst|InitTables|MoreSettings` → basics/
`Queryable|Where|Select|Join|Includes|ThenMapper|Expressionable` → query/
`Insertable|Updateable|Deleteable|Storageable|Fastest` → write/
`Ado|SqlQuery|ExecuteCommand|DynamicBuilder|Json2Sql` → dynamic/
`BeginTran|UseTran|Tenant|SplitTable|Aop|QueryFilter|SugarRetry` → runtime/
`StaticConfig|SqlMiddle|InstanceFactory|SqlFuncExternal|ConnMoreSettings` → architecture/
`QueryableWithAttr|InsertableWithAttr|MasterQueryable` → runtime/ 或 architecture/04
<!-- /section:lookup -->

<!-- section:entries -->
## 高价值入口

### 架构选型
- Client vs Scope → `basics/02`（对比表）→ 按需加 `basics/03` 或 `04`
- 单例模式 → `runtime/17`（含错误写法）；IOC 示例 → `basics/04`
- IOC 注入 → `runtime/18`

### 实体建模
- 配置系列：`basics/07`(内置) → `08`(自定义) → `09`(无特性) → `10`(完整示例)
- ValueObject/鉴别器/实体管理：`basics/11`、`12`、`13`

### 代码生成
- CodeFirst → `basics/17` → 按需加 `18`(配置)、`19`(表结构)、`20`(差异)
- DbFirst → `basics/14` → 按需加 `15`(Razor)、`16`(自定义生成器)

### 查询
- 基础 → `query/01`；条件 → `02`；投影 → `03`；执行+ThenMapper → `09`
- 联表 → `12`；导航 → `15`；子查询 → `13`
- 函数 → `08_SqlFunc`；分页 → `04`；动态表达式(12表) → `22`

### 写入
- 插入 → `write/01`；更新 → `02`；删除 → `03`；批量 → `05`；保存 → `04`

### AOP/过滤器
- AOP 全事件+日志+差异 → `runtime/25`
- 查询过滤器 → `runtime/26`（≈`query/21`，读一个即可）

### 故障排查
- 偶发错误/线程安全 → `runtime/06`
- 常见报错 → `runtime/23`
- 版本要求 → `runtime/24`
- 数据库连接 → `database/XX_数据库名.md` 注意事项

### 数据库适配
- 厂商文件（如 `database/01_MySQL.md`）→ 按需加 `31_通用配置`、`32_类型映射`、`35_JSON`、`36_枚举`、`37_自定义转换`

### 源码架构
- 核心类关系+InstanceFactory → `architecture/01`
- 方法调用链(查询/插入/CodeFirst) → `architecture/02`
- 隐藏API(StaticConfig/SqlMiddle/AOP 12事件/ConnMoreSettings/SqlFuncExternal) → `architecture/03`
- 接口体系(29接口分组+设计模式) → `architecture/04`
- 废弃API迁移(7项+升级清单) → `architecture/05`
<!-- /section:entries -->

<!-- section:redundancy -->
## 已知冗余

- `runtime/17` ↔ `basics/04` — 单例读 `runtime/17`，IOC 读 `basics/04`
- `runtime/26` ≈ `query/21` — 查询过滤器内容相同，读 `runtime/26`
- `runtime/25` 末尾含过滤器概述，详情仍读 `runtime/26`
<!-- /section:redundancy -->

<!-- section:rules -->
## 回答规则

- 保留 SqlSugar 类型名原样（`SqlSugarScope`、`Queryable<T>`、`Storageable`）
- 代码示例用 C#，遵循官方命名风格
- 涉及特定数据库时标注引用的厂商文件
- 版本敏感功能标注最低版本（如 `5.1.4.63+`）
- 报错优先引导 → `runtime/06`(偶发错误) + `runtime/23`(常见问题)

> 每次回答只加载最小必要文件集，避免上下文溢出。
<!-- /section:rules -->
