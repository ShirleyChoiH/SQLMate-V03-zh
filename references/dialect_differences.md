# SQL 方言差异

用于在不同方言之间挑选语法的速查表。挑出与用户指定方言对应的一行；未指定时，默认使用 PostgreSQL / ANSI SQL，并在"注意事项"中写一句可移植提示。

---

## SELECT TOP / LIMIT

| 方言 | 语法 |
| --- | --- |
| PostgreSQL、MySQL | `SELECT ... LIMIT n OFFSET m;` |
| SQLite | `SELECT ... LIMIT n OFFSET m;` |
| BigQuery | `SELECT ... LIMIT n;` |
| Snowflake | `SELECT ... LIMIT n;` |
| SQL Server | `SELECT TOP n ...`（旧式），或 `SELECT ... ORDER BY ... OFFSET m ROWS FETCH NEXT n ROWS ONLY;`（ANSI） |
| Oracle | `SELECT ... FETCH FIRST n ROWS ONLY;` |

---

## 字符串聚合

| 方言 | 函数 |
| --- | --- |
| PostgreSQL | `STRING_AGG(expr, ', ' ORDER BY col)` |
| SQL Server | `STRING_AGG(expr, ', ')`（2017+） |
| MySQL | `GROUP_CONCAT(expr SEPARATOR ', ')` |
| BigQuery | `STRING_AGG(expr, ', ' ORDER BY col)` |
| Snowflake | `LISTAGG(expr, ', ') WITHIN GROUP (ORDER BY col)` |
| Oracle | `LISTAGG(expr, ', ') WITHIN GROUP (ORDER BY col)` |

---

## 日期 / 时间函数

| 目标 | PostgreSQL | MySQL | SQL Server | BigQuery |
| --- | --- | --- | --- | --- |
| 当前时间 | `NOW()` / `CURRENT_TIMESTAMP` | `NOW()` | `GETDATE()` / `SYSDATETIME()` | `CURRENT_TIMESTAMP()` |
| 截断到天 | `DATE_TRUNC('day', ts)` | `DATE(ts)` / `DATE_FORMAT(ts, '%Y-%m-%d')` | `CAST(ts AS DATE)` / `FORMAT(ts, 'yyyy-MM-dd')` | `DATE_TRUNC(ts, DAY)` |
| 日期差（天） | `ts1::date - ts2::date` | `DATEDIFF(ts1, ts2)` | `DATEDIFF(day, ts2, ts1)` | `DATE_DIFF(ts1, ts2, DAY)` |
| 加 n 天 | `ts + INTERVAL '1 day'` | `DATE_ADD(ts, INTERVAL 1 DAY)` | `DATEADD(day, n, ts)` | `DATE_ADD(ts, INTERVAL n DAY)` |
| 取年份 / 月份 | `EXTRACT(YEAR FROM ts)` | `YEAR(ts)`、`MONTH(ts)` | `YEAR(ts)`、`MONTH(ts)` | `EXTRACT(YEAR FROM ts)` |

---

## 布尔值与真值

| 方言 | 布尔字面量 | `WHERE` 真值判断 |
| --- | --- | --- |
| PostgreSQL | `TRUE` / `FALSE` | 必须显式布尔表达式 |
| MySQL 8+ | `TRUE` / `FALSE` | 数字自动转布尔；`0` 为假 |
| SQL Server | 没有原生布尔 | 用 `bit`：`WHERE is_active = 1` |
| BigQuery | `TRUE` / `FALSE` | 必须显式布尔表达式 |
| SQLite | `0` / `1` | `0` 为假，其他均视为真 |

---

## 字符串匹配与正则

| 方言 | 不区分大小写 LIKE | 正则 |
| --- | --- | --- |
| PostgreSQL | `ILIKE` | `~` / `~*`（不区分大小写）/ `REGEXP_MATCHES` |
| MySQL | `LIKE` 默认不区分大小写（取决于 collation） | `REGEXP` / `RLIKE` |
| SQL Server | `LIKE` 默认不区分大小写（取决于 collation） | `LIKE` 配合 `%[...]%` 模式 |
| BigQuery | `LOWER(col) LIKE LOWER(pattern)` | `REGEXP_CONTAINS(col, pattern)` |
| SQLite | `LIKE` 默认对 ASCII 不区分大小写 | `REGEXP`（无内置，需扩展） |

---

## 标识符引号

| 方言 | 引号格式 | 说明 |
| --- | --- | --- |
| PostgreSQL | `"column"` | 大小写敏感 |
| MySQL | `` `column` `` | Linux 大小写敏感；macOS/Windows 不敏感 |
| SQL Server | `[column]` 或 `"column"` | 大小写不敏感 |
| BigQuery | `` `column` `` | 反引号 |
| ANSI 标准 | `"column"` | 推荐用于可移植性 |

---

## NULL 处理差异

- `CONCAT` 在 MySQL 与 BigQuery 中跳过 `NULL`，但 PostgreSQL 会因 `NULL` 返回 `NULL`。
- `NULLIF(a, 0)` 各方言通用；用作除法分母前先调它。
- `DISTINCT` 把所有 `NULL` 视为相等（只保留一个 `NULL`），各方言通用。
- 有序集聚合（如 `PERCENTILE_CONT`）在 PostgreSQL 与 SQL Server 中存在；MySQL 无直接等价，需在应用层计算。

---

## 分页

| 方言 | 推荐写法 |
| --- | --- |
| PostgreSQL | `LIMIT n OFFSET m`（深翻页慢——更推荐 keyset 分页） |
| MySQL | `LIMIT n OFFSET m` |
| SQL Server | `ORDER BY ... OFFSET m ROWS FETCH NEXT n ROWS ONLY` |
| BigQuery | `LIMIT n OFFSET m`；也支持基于游标的分页 `WHERE ts > last_ts` |
| Snowflake | `LIMIT n OFFSET m` |

---

## 当用户说"就普通 SQL 就行"

默认采用 PostgreSQL / ANSI SQL 语法，可在 PostgreSQL、SQLite、BigQuery、Snowflake 不改直接运行；在 MySQL / SQL Server 上只需微调。在"注意事项"中列出最可能需要的微调：

- MySQL：`STRING_AGG` → `GROUP_CONCAT`。
- SQL Server：`LIMIT n` → `SELECT TOP n`。
- SQL Server：`TRUE` / `FALSE` → `1` / `0`。
- MySQL / SQL Server：`EXTRACT(YEAR FROM ts)` → `YEAR(ts)`。
