---
name: dbhub
description: 用于通过 DBHub MCP 服务器查询数据库的指南。当需要探索数据库结构、检查表结构或通过 DBHub 的 MCP 工具（search_objects、execute_sql）执行 SQL 查询时，应使用此技能。当涉及任何数据库查询任务、Schema 探索、数据获取或通过 MCP 执行 SQL 时都会触发 —— 即使用户只是说“查看数据库”或“帮我找一些数据”。该技能确保你遵循正确的“先探索后查询”流程，而不是猜测表结构。
---

# DBHub 数据库查询指南

在通过 DBHub 的 MCP 服务器操作数据库时，始终遵循 **先探索（explore）再查询（query）** 的模式。
在不了解 Schema 的情况下直接编写 SQL 是最常见的错误 —— 这会导致查询失败、浪费 token，并让用户产生挫败感。

---

# 可用工具

DBHub 提供两个 MCP 工具：

| 工具               | 作用                               |
| ---------------- | -------------------------------- |
| `search_objects` | 探索数据库结构 —— schema、表、列、索引、存储过程、函数 |
| `execute_sql`    | 在数据库上执行 SQL 语句                   |

如果配置了多个数据库，DBHub 会为每个数据源注册独立的工具（例如 `search_objects_prod_pg`、`execute_sql_staging_mysql`）。
通过调用相应名称的工具来选择目标数据库。

---

# 先探索后查询（Explore-Then-Query）工作流

每个数据库任务都应遵循以下流程。
核心思想是：每一步都会缩小范围，从而避免加载不必要的信息并浪费 token。

---

## 步骤 1：发现有哪些 Schema

```
search_objects(object_type="schema", detail_level="names")
```

这一步可以帮助你了解数据库整体结构。
大多数数据库都有一个主 schema（例如 PostgreSQL 的 `public`，SQL Server 的 `dbo`），以及一些可以忽略的系统 schema。

---

## 步骤 2：找到相关表

当知道 schema 后，列出其中的表：

```
search_objects(object_type="table", schema="public", detail_level="names")
```

如果需要查找特定对象，可以使用 pattern：

```
search_objects(object_type="table", schema="public", pattern="%user%", detail_level="names")
```

`pattern` 参数使用 SQL LIKE 语法：

* `%` 匹配任意字符
* `_` 匹配单个字符

如果需要更多上下文（例如行数、列数或表注释）来识别正确的表，可以使用：

```
detail_level="summary"
```

---

## 步骤 3：检查表结构

在编写任何查询之前，先了解表的列结构：

```
search_objects(object_type="column", schema="public", table="users", detail_level="full")
```

返回信息包括：

* 列名
* 数据类型
* 是否允许 NULL
* 默认值

这些信息足以编写正确的 SQL。

如果需要了解查询性能或 join 方式，还可以查看索引：

```
search_objects(object_type="index", schema="public", table="users", detail_level="full")
```

---

## 步骤 4：编写并执行查询

当你已经确认表名和列名后，就可以编写精确的 SQL：

```
execute_sql(sql="SELECT id, email, created_at FROM public.users WHERE created_at > '2024-01-01' ORDER BY created_at DESC")
```

---

# 渐进式信息披露：选择正确的 detail_level

`detail_level` 参数控制 `search_objects` 返回的信息量。
应从最小信息开始，并在需要时逐步深入 —— 这样可以保持响应快速并减少 token 使用。

| 级别        | 返回内容                | 使用场景            |
| --------- | ------------------- | --------------- |
| `names`   | 仅对象名称               | 浏览、寻找目标表        |
| `summary` | 名称 + 元数据（行数、列数、注释）  | 在类似表之间选择、了解数据规模 |
| `full`    | 完整结构（列类型、索引、存储过程定义） | 编写查询前、理解关系结构    |

经验规则：

* 广泛探索 → `names`
* 缩小范围 → `summary`
* 查询前分析 → `full`

---

# 多数据库环境

当 DBHub 配置了多个数据库源时，会为每个源注册独立工具。
工具名称遵循：

```
{tool}_{source_id}
```

例如：

```
# 查询生产 PostgreSQL 数据库
search_objects_prod_pg(object_type="table", schema="public", detail_level="names")
execute_sql_prod_pg(sql="SELECT count(*) FROM orders")

# 查询测试 MySQL 数据库
search_objects_staging_mysql(object_type="table", detail_level="names")
execute_sql_staging_mysql(sql="SELECT count(*) FROM orders")
```

如果只有单个数据库，工具名称就是：

```
search_objects
execute_sql
```

当用户提到具体数据库或环境时，应调用对应的数据源工具。

---

# 搜索特定对象

`search_objects` 支持针对不同对象类型的搜索：

```
# 查找名称包含 "order" 的所有表
search_objects(object_type="table", pattern="%order%", detail_level="names")

# 查找所有名为 "email" 的列
search_objects(object_type="column", pattern="email", detail_level="names")

# 查找匹配 pattern 的存储过程
search_objects(object_type="procedure", schema="public", pattern="%report%", detail_level="summary")

# 查找函数
search_objects(object_type="function", schema="public", detail_level="names")
```

---

# 常见使用模式

## “数据库里有什么数据？”

1. 列出 schemas
2. 使用 `summary` 列出表
3. 选择相关表
4. 使用 `full` 查看结构

---

## “从数据库获取 X 数据”

1. 搜索与 X 相关的表
2. 检查列结构
3. 编写精准的 SELECT 查询

---

## “这些表之间有什么关系？”

1. 对两个表使用 `full` 查看结构
2. 通过列和索引识别外键或 join 字段

---

## “执行这条 SQL”

如果用户提供了完整 SQL，可以直接执行。
如果执行失败（例如列或表错误），应回到探索流程，而不是猜测修复。

---

# 错误恢复

当查询失败时：

* **未知表或列**
  使用 `search_objects` 查找正确名称，而不是猜测

* **Schema 错误**
  先列出所有 schema —— 表可能位于不同 schema

* **权限错误**
  数据库可能是只读模式，检查是否仅允许 SELECT

* **多语句执行**
  `execute_sql` 支持通过 `;` 分隔多个 SQL 语句

---

# 不要这样做

* **不要猜测表或列名**
  始终先使用 `search_objects` 验证

* **不要一次性加载整个 schema**
  使用渐进式探索：先 `names`，再针对性 `full`

* **不要在多数据库环境中使用错误工具**
  如果用户指定数据库，应使用对应的数据源工具（例如 `execute_sql_prod_pg`）

* **不要盲目重试失败查询**
  如果 SQL 失败，应先分析 schema，再重新编写查询。
