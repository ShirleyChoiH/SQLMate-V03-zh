# SQL 优化模式

当用户说"优化这条 SQL"、查询看起来开销很大，或"注意事项"需要一条实际的性能说明时，加载本参考。每个模式都给出**反例**、**修正**、以及**生效原因**。

本 skill 的首要原则是**简约省钱**：用最少的代码和最小的数据扫描量完成需求。以下模式帮助在查询看起来昂贵时找到更省的替代方案。

---

## 1. 可走索引的谓词（Sargable）

如果数据库能够使用被过滤列上的索引，那么该谓词就是 *sargable* 的。

**反例**
```sql
WHERE UPPER(email) = 'USER@EXAMPLE.COM'
WHERE DATE(created_at) = '2026-01-15'
WHERE YEAR(created_at) = 2026
```

**修正**
```sql
WHERE email = 'user@example.com'           -- 存储规范大小写，或在 UPPER 上加函数索引
WHERE created_at >= '2026-01-15' AND created_at < '2026-01-16'
WHERE created_at >= '2026-01-01' AND created_at < '2027-01-01'
```

**原因** —— 给列套一层函数会强制全表扫描；索引无法被使用。

---

## 2. 子查询优先 EXISTS 而非 IN

**反例**
```sql
SELECT u.* FROM users u
WHERE u.id IN (SELECT user_id FROM page_views WHERE ts >= NOW() - INTERVAL '30 days');
```

**修正**
```sql
SELECT u.* FROM users u
WHERE EXISTS (
  SELECT 1 FROM page_views pv
  WHERE pv.user_id = u.id AND pv.ts >= NOW() - INTERVAL '30 days'
);
```

**原因** —— `IN` 会物化整个子查询，再把外层每一行与之比较；`EXISTS` 在外层每行的首个匹配处即可短路。

---

## 3. 避免 SELECT *

**反例** —— `SELECT *` 会返回用不到的列，阻断"覆盖索引"策略，并在表结构变化时打乱代码。

**修正** —— 显式写出需要的列。在列存数仓（BigQuery、Snowflake）上，扫描量可能下降 5-100 倍。

---

## 4. 尽早过滤、晚做 JOIN

把 `WHERE` 过滤尽量靠近数据源。现代优化器通常会自动做这一步，但 CTE + 下游 `WHERE` 在老版本规划器中可能误导优化顺序。

**模式**
```sql
WITH recent AS (
  SELECT user_id, page_name, ts
  FROM page_views
  WHERE ts >= NOW() - INTERVAL '30 days'   -- 先过滤
)
SELECT r.page_name, COUNT(DISTINCT r.user_id) AS users
FROM recent r
GROUP BY r.page_name;
```

---

## 5. 用窗口函数去重，代替自连接

**反例**
```sql
SELECT * FROM events e1
JOIN events e2 ON e1.user_id = e2.user_id AND e1.ts < e2.ts;
```

**修正**
```sql
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY ts DESC) AS rn
  FROM events
) ranked WHERE rn = 1;
```

`ROW_NUMBER() = 1` 一遍扫描即可并行；自连接让输入规模呈平方级膨胀。

---

## 6. 为热点查询建覆盖索引

覆盖索引同时包含查询所需的过滤列与 `SELECT` 列，这样数据库就完全不必回表。

```sql
CREATE INDEX idx_pv_user_recent
  ON page_views (user_id, ts DESC)
  INCLUDE (page_name);     -- SQL Server / PostgreSQL 11+
```

---

## 7. 独立计数：HyperLogLog / 近似 / 精确

| 需求 | 最划算的实现 |
| --- | --- |
| 精确计数 | `COUNT(DISTINCT col)` |
| 近似（±2%） | `APPROX_COUNT_DISTINCT(col)`（BigQuery / Snowflake） |
| HyperLogLog | PostgreSQL `hll` 扩展 / Snowflake `HLL_ACCUMULATE` |

`COUNT(DISTINCT)` 在大表上是最耗资源的操作之一；如果只要量级，可切换为近似计数。

---

## 8. JOIN 顺序在小引擎上仍然要紧

MySQL / PostgreSQL 上的规划器通常会重新排序 JOIN。在极大表上 JOIN 时，借助 CTE（PostgreSQL 12+ 默认内联）或 MySQL / Oracle 的 `/*+ LEADING */` 优化器提示能跑赢规划器。除非**实测**规划器错了，否则不要急着加 hint。

---

## 9. 窗口函数的 frame

未指定 `ROWS BETWEEN` 的窗口函数默认使用 `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`，一般够用。但在大分区上，缩小窗口能让函数开销显著降低：

```sql
SUM(amount) OVER (
  PARTITION BY user_id
  ORDER BY ts
  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS running_total
```

优先使用 `ROWS`（按行计数，快）而不是 `RANGE`（按值范围，会扫描同值的所有行）。

---

## 10. 批量化 DELETE / UPDATE

`DELETE FROM huge_table WHERE condition` 可能锁表数分钟。拆批执行：

```sql
DELETE FROM logs WHERE ts < '2025-01-01' LIMIT 10000;  -- MySQL
-- 或 PostgreSQL：
DELETE FROM logs WHERE ctid IN (
  SELECT ctid FROM logs WHERE ts < '2025-01-01' LIMIT 10000
);
```

---

## 在"注意事项"中应提示的基数与代价信号

- `SELECT COUNT(*) FROM huge_table` 不带 `WHERE`——全表扫描。
- 对宽表的每一列都 `DISTINCT`——每行做一次单行哈希。
- 大外表 `SELECT` 中嵌套相关子查询——每行重新求值。
- `LIKE '%foo'`（前导通配）——无法走索引。
- 同一索引列上多个 `OR` 连接的不等式——通常改写为 `UNION` 更划算。
- 用 `UNION`（不带 `ALL`）合并本来想用 `UNION ALL` 的结果——额外的排序/去重开销。

---

## 11. 分区裁剪（Partition Pruning）

BigQuery、Snowflake、Hive 等支持分区表的引擎上，分区列的过滤条件直接决定扫描哪些分区。不加分区过滤 = 扫全表 = 全额计费。

**反例**
```sql
-- 扫描整张分区表（非常贵）
SELECT user_id, page FROM page_views GROUP BY user_id, page;
```

**修正**
```sql
-- 只扫描 2026 年 9 月的分区
SELECT user_id, page
FROM page_views
WHERE date >= '2026-09-01' AND date < '2026-10-01'
GROUP BY user_id, page;
```

**原因** —— 分区裁剪让引擎跳过不匹配的物理分区文件，扫描量和费用可能降低几个数量级。

---

## 12. 只在真正需要另一张表的列时才 JOIN

JOIN 不是"显得专业"的工具——如果用户的问题只涉及一张表的数据，就用单表查询。每次多余的 JOIN 都可能让扫描行数成倍增长。

**反例**
```sql
-- 用户只问"订单表里有哪些金额大于 100 的订单"
SELECT o.id, o.amount, c.name
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id
WHERE o.amount > 100;
```

**修正**
```sql
-- 用户不需要客户名，只问订单本身
SELECT id, amount
FROM orders
WHERE amount > 100;
```

**原因** —— 少一次 JOIN = 少扫一张表的对应数据，查询更简单、更快、更省钱。只有在用户**明确需要**另一张表的列时才 JOIN。

---

## 13. 量级估算时用近似函数

精确 `COUNT(DISTINCT)` 在大表上是最耗资源的操作之一。如果业务只需要量级估算（如"大约有多少独立用户"），使用近似函数。

| 场景 | 推荐写法 | 误差 | 引擎 |
| --- | --- | --- | --- |
| 精确独立计数 | `COUNT(DISTINCT col)` | 0 | 所有 |
| 近似独立计数 | `APPROX_COUNT_DISTINCT(col)` | ±0.5-2% | BigQuery / Snowflake |
| HyperLogLog | `HLL_ACCUMULATE(col)` | ±1-3% | Snowflake / PostgreSQL (hll 扩展) |
| 近似分位数 | `APPROX_QUANTILES(col, n)` | 可控 | BigQuery |

**原因** —— 近似算法用固定大小的内存哈希（而非全量去重），在亿级数据上可以省掉几十 GB 的中间结果。

---

## 14. 简单需求用简单 SQL

如果需求简单，SQL 就应该简单。5 行能写完的查询不写成 20 行——多余的嵌套不会改变执行计划，但增加了维护成本和理解难度。

**反例**
```sql
-- 用户只问"这个表有多少行"
WITH counted AS (
  SELECT COUNT(*) AS total FROM page_views
)
SELECT total FROM counted;
```

**修正**
```sql
SELECT COUNT(*) FROM page_views;
```

**原因** —— 多余的 CTE 嵌套不会改变执行计划，但增加了维护成本和理解难度。只在查询**逻辑上**有多个阶段时才用 CTE。
