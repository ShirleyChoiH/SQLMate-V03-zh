# SQL 概念参考

每个概念的精炼定义、语法骨架、最小示例与最常见的坑。按需加载对应条目；不要把全部条目都堆在面向用户的回复里。

---

## GROUP BY

把在所列列上取值相同的行合并为一组输出行，聚合函数针对每一组分别计算——而不是对全表计算。

**语法骨架**
```sql
SELECT group_col, AGG(expr) AS alias
FROM t
GROUP BY group_col;
```

**最小示例**
```sql
-- 每个班级的平均分
SELECT class, AVG(score) AS avg_score
FROM scores
GROUP BY class;
```

**常见坑** —— `SELECT` 中只能出现 (a) `GROUP BY` 中列出的列，(b) 聚合函数。非聚合列未参与 `GROUP BY` 在严格模式下（PostgreSQL、BigQuery、Snowflake）会报错；在 MySQL 关闭 `ONLY_FULL_GROUP_BY` 时则会返回不确定的结果。

---

## DISTINCT

从结果集中移除重复行。`COUNT(DISTINCT col)` 统计非 NULL 的独立值数量；`COUNT(*)` 统计行数（含重复与 NULL）。

**语法骨架**
```sql
SELECT DISTINCT col1, col2 FROM t;
SELECT COUNT(DISTINCT col) FROM t;
```

**最小示例**
```sql
-- 有多少个不同的用户浏览过
SELECT COUNT(DISTINCT user_id) AS unique_users
FROM page_views;
```

**常见坑** —— `DISTINCT` 作用于**整个** select 列表，不是单列，除非包在聚合里：`SELECT DISTINCT a, b` ≠ `SELECT DISTINCT a, DISTINCT b`。

---

## JOIN

通过谓词把两张表的行合并起来。

| 类型 | 保留哪些行 |
| --- | --- |
| `INNER` | 谓词在**两侧**都匹配上的 |
| `LEFT` | 左侧全部行；右侧未匹配的列为 `NULL` |
| `RIGHT` | 右侧全部行；左侧未匹配的列为 `NULL` |
| `FULL` | 两侧全部行；缺失一侧补 `NULL` |
| `CROSS` | 笛卡尔积（全组合） |
| `LATERAL` | 右侧可逐行引用左侧的列 |

**语法骨架**
```sql
SELECT a.col, b.col
FROM table_a a
LEFT JOIN table_b b ON a.id = b.a_id;
```

**最小示例**
```sql
-- 每个用户及其订单金额（没下单的用户也保留）
SELECT u.name, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

**常见坑** —— `LEFT JOIN` 之后再在 `WHERE` 里过滤右表列，会悄悄退化成 `INNER JOIN`。要保留左表未匹配的，就把右表的过滤放在 `ON` 而不是 `WHERE`。

---

## EXISTS 与 IN

二者都用于成员判断。在子查询场景下优先 `EXISTS`，因为它能短路，并且对内外列解耦。

**语法骨架**
```sql
SELECT u.id FROM users u
WHERE EXISTS (SELECT 1 FROM page_views pv WHERE pv.user_id = u.id);
```

**最小示例**
```sql
-- 找出有浏览记录的用户
SELECT u.name
FROM users u
WHERE EXISTS (
  SELECT 1 FROM page_views pv
  WHERE pv.user_id = u.id
);
```

**常见坑** —— `IN` 配子查询在带 `NULL` 时行为易出 bug：`NOT IN (subquery with NULL)` 会返回**零**行，而不是"不在列表中的行"。

---

## CTE（WITH）

一种在单条语句内生效的命名子查询。提升可读性，并允许同一中间结果被引用多次。

**语法骨架**
```sql
WITH cte_name AS (
  SELECT ... FROM ...
)
SELECT ... FROM cte_name;
```

**最小示例**
```sql
WITH daily AS (
  SELECT user_id, DATE_TRUNC('day', ts) AS day
  FROM page_views
)
SELECT day, COUNT(DISTINCT user_id) AS users
FROM daily
GROUP BY day;
```

**常见坑** —— 在较老的 PostgreSQL / SQL Server 中，CTE 是"优化屏障"（materialize 一次）。PostgreSQL 12+ 与 BigQuery 默认会内联 CTE。同一语句中不要重复使用同一个 CTE 名。

---

## 窗口函数

在**不折叠行**的前提下，对行做聚合（或排序、导航）。结果行数与输入相同，会新增一列。

**语法骨架**
```sql
SELECT
  col1,
  FUNC() OVER (
    PARTITION BY partition_col
    ORDER BY order_col
    ROWS BETWEEN frame_start AND frame_end
  ) AS alias
FROM t;
```

**最小示例**
```sql
-- 每个用户的第几次浏览（按时间排序）
SELECT
  user_id,
  ts,
  ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY ts) AS nth_view
FROM page_views;
```

常用窗口函数：`ROW_NUMBER`、`RANK`、`DENSE_RANK`、`LAG`、`LEAD`、`SUM/AVG/COUNT ... OVER (...)`、`FIRST_VALUE`、`NTILE`。

**常见坑** —— `PARTITION BY` **不是** `GROUP BY`。它不减少行数，只是按组重置窗口起点。

---

## CASE WHEN

行内条件判断。两种形式：简单形式（每条分支对同一表达式取值）和搜索形式（每条分支是布尔条件）。

**语法骨架**
```sql
SELECT
  CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE default_result
  END AS alias
FROM t;
```

**最小示例**
```sql
-- 按分数分等级
SELECT
  name,
  CASE
    WHEN score >= 90 THEN 'A'
    WHEN score >= 80 THEN 'B'
    WHEN score >= 70 THEN 'C'
    ELSE 'F'
  END AS grade
FROM scores;
```

用于分桶、自定义排序、条件聚合（`SUM(CASE WHEN is_new THEN 1 ELSE 0 END)`）。

---

## COALESCE / NULLIF

- `COALESCE(a, b, c, ...)` —— 返回第一个非 NULL 参数。即标准的"默认值"写法。
- `NULLIF(a, b)` —— 若 `a = b` 返回 `NULL`，否则返回 `a`。在做除法避免 `/0` 时很顺手：`amount / NULLIF(quantity, 0)`。

**语法骨架**
```sql
SELECT COALESCE(nickname, real_name, 'anonymous') AS display_name
FROM users;

SELECT amount / NULLIF(quantity, 0) AS unit_price
FROM orders;
```

**常见坑** —— `COALESCE` 和 `NULLIF` 常与方言专属的 `ISNULL` / `NVL` 混淆。跨方言请用 `COALESCE`，再在方言片段中替换。

---

## 子查询：标量与相关

- **标量** —— 返回单个值。用在 `SELECT` 或 `WHERE col = (SELECT ...)` 中。
- **相关** —— 引用外部查询；对外层每一行重新求值。在大表上常常是性能陷阱。

**最小示例**
```sql
-- 标量子查询：每个用户的订单数
SELECT
  u.name,
  (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count
FROM users u;
```

**常见坑** —— `SELECT col = (SELECT ...)` 在子查询返回多于一行的结果时，会返回 `NULL` 而**不会报错**。

---

## 集合运算

`UNION`（去重）、`UNION ALL`（不去重，更快）、`INTERSECT`、`EXCEPT` / `MINUS`。

**语法骨架**
```sql
SELECT col FROM table_a
UNION [ALL | INTERSECT | EXCEPT]
SELECT col FROM table_b;
```

**常见坑** —— 两侧列数相同、类型兼容。`INTERSECT` / `EXCEPT` 会静默去重——想保留重复要加 `ALL`。

---

## 日期 / 时间函数

| 目标 | PostgreSQL | MySQL | SQL Server | BigQuery |
| --- | --- | --- | --- | --- |
| 截断到天 | `DATE_TRUNC('day', ts)` | `DATE_FORMAT(ts, '%Y-%m-%d')` | `CAST(ts AS DATE)` | `DATE_TRUNC(ts, DAY)` |
| 日期差（天） | `ts1 - ts2` | `DATEDIFF(ts1, ts2)` | `DATEDIFF(day,...)` | `DATE_DIFF(DATE ts1, ts2)` |
| 加 n 天 | `ts + INTERVAL '1 day'` | `DATE_ADD(ts, INTERVAL 1 DAY)` | `DATEADD(day, 1, ts)` | `DATE_ADD(ts, INTERVAL 1 DAY)` |
| 当前时间 | `NOW()` | `NOW()` | `GETDATE()` | `CURRENT_TIMESTAMP()` |

---

## 字符串聚合

把多行拼接为单一单元格。

- PostgreSQL / SQL Server / Snowflake：`STRING_AGG(expr, ', ' ORDER BY ...)`
- MySQL：`GROUP_CONCAT(expr SEPARATOR ', ')`
- BigQuery：`STRING_AGG(expr, ', ')`

**常见坑** —— `STRING_AGG` 内的 `DISTINCT` 在 PostgreSQL 中支持，但并非所有方言都支持；不确定时可先在 CTE 中去重。

---

## HAVING

对 `GROUP BY` 产生的分组进行过滤。与 `WHERE` 的核心区别：`WHERE` 在分组前过滤行，`HAVING` 在分组后过滤组。

**语法骨架**
```sql
SELECT group_col, AGG(expr) AS alias
FROM t
GROUP BY group_col
HAVING AGG(expr) > threshold;
```

**最小示例**
```sql
-- 找出平均分超过 85 的班级
SELECT class, AVG(score) AS avg_score
FROM scores
GROUP BY class
HAVING AVG(score) > 85;
```

**常见坑** —— `HAVING` 中只能引用 `GROUP BY` 的列和聚合函数，不能引用未分组的非聚合列。`WHERE` 中不能用聚合函数。

---

## LIMIT / OFFSET

限制返回行数，或跳过前几行。常用于分页。

**语法骨架**
```sql
SELECT ... FROM t
ORDER BY col
LIMIT n OFFSET m;
```

**常见坑** —— 不带 `ORDER BY` 的 `LIMIT` 结果不确定（每次执行可能不同）。深分页（大 OFFSET）性能差，推荐用 keyset 分页（`WHERE id > last_id ORDER BY id LIMIT n`）。
