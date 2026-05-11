# SkillRunner Market

High-quality Claude Code skill plugins maintained by [Jay-ssr](https://github.com/Jay-ssr).

## Plugins

| Plugin | Description | Category |
|--------|-------------|----------|
| [sqlsugar-guide](./plugins/sqlsugar-guide) | SqlSugar ORM complete reference knowledge base (130+ topics) | knowledge-base |

## Install

```bash
# 1. Add marketplace
claude plugin marketplace add Jay-ssr/skillrunner-market

# 2. Install plugin
claude plugin install sqlsugar-guide@skillrunner-market
```

Or in Claude Code interactive mode:

```
/plugin marketplace add Jay-ssr/skillrunner-market
/plugin install sqlsugar-guide@skillrunner-market
```

## SqlSugar Guide Plugin

A comprehensive SqlSugar ORM knowledge base that auto-activates when you ask about SqlSugar, ORM, or database operations in C#/.NET projects.

### Coverage (130+ topic files)

- **basics/** (20 files) - Installation, SqlSugarClient/Scope, connection config, entity modeling, ValueObject, discriminator, DbFirst, CodeFirst
- **query/** (22 files) - Where, Select, pagination, join, subquery, navigation query, SqlFunc, ThenMapper, dynamic expression
- **write/** (12 files) - Insert, update, delete, Storageable, bulk write, GridSave, snowflake ID, navigation CRUD
- **dynamic/** (10 files) - Entityless CRUD, raw SQL/Ado, DynamicBuilder, Json2Sql, string expression
- **runtime/** (27 files) - Transaction, lock, async, cache, split table, multi-tenant, AOP, query filter, SugarRetry
- **database/** (37 files) - 30+ database vendors (MySQL, SqlServer, PostgreSQL, Oracle, SQLite, DM, KingbaseES, ClickHouse, etc.)
- **architecture/** (5 files) - Core class hierarchy, method call chains, hidden APIs, interface system (29 interfaces), deprecated API migration guide

### After Installation

The skill auto-activates when you mention SqlSugar, ORM, or database operations. No manual invocation needed.

## License

MIT
