---
name: sql-mate-3-zh
description: 本 skill 应用于 SQL 初学者或工作中需要使用 SQL 但不熟悉 SQL 的人。覆盖两种场景：Case 1 — 用户需要解决一个实际的数据问题（例如查找或统计某个数据），输出包含完整 SQL 代码、结果解读、注意事项、小课堂（逐行解读代码并配图解）；Case 2 — 用户想要弄懂某个 SQL 函数或概念（例如 GROUP BY、JOIN 等），输出包含函数解读、示例 SQL 及模拟结果、图解。典型触发包括"写一个 SQL 查询……"、"怎么统计每个页面的独立用户数"、"帮我解释这条 SQL"、"GROUP BY 是做什么的"、"JOIN 怎么用"、"SQL 入门"等。所有图解使用主题色 #008e89 与 #00337c。
agent_created: true
---

# SQL Mate 3.0

## 概览

面向 SQL 初学者与工作中需要用 SQL 但不熟悉 SQL 的人。本 skill 把用户的问题路由到两种场景之一，每种场景有固定的输出结构和教学风格。

- **Case 1（解决实际问题）**：用户有一个具体的数据需求——查找、统计、排序、筛选某个数据。输出四部分：SQL 代码 → 结果解读 → 注意事项 → 小课堂（逐行解读 + 图解）。
- **Case 2（理解某个函数）**：用户想弄懂一个 SQL 函数或概念。输出两部分：函数解读 → 示例 + 图解。

## 场景路由

收到用户请求后，先判断属于哪种场景：

| 信号 | 路由到 |
| --- | --- |
| 用户描述了一个数据需求："帮我查出……""统计每个……的……""找出销售额最高的……" | **Case 1** |
| 用户给出表结构 + 数据问题 | **Case 1** |
| 用户问"XXX 怎么用""XXX 是什么""XXX 和 YYY 有什么区别" | **Case 2** |
| 用户粘贴一条 SQL 问"这条在做什么" | **Case 1**（但 SQL 已给定，跳过"写 SQL"步骤，直接从解读开始） |
| 用户说"给我讲讲窗口函数""GROUP BY 怎么理解" | **Case 2** |

如果两种信号同时出现（例如"帮我写一个用到窗口函数的查询，顺便讲讲窗口函数"），优先按 Case 1 输出完整四部分，在小课堂部分对该函数做 Case 2 式的深入讲解。

---

## 设计令牌

所有可视化输出（SVG 图解、表格、代码块样式）严格遵循以下设计令牌：

| 令牌 | 值 | 用途 |
| --- | --- | --- |
| `--primary` | `#008e89` | 主色：标题栏、强调标签、图解主线条、表格表头 |
| `--secondary` | `#00337c` | 辅色：副标题、数据高亮、图解次要线条 |
| `--bg-card` | `#ffffff` | 卡片背景 |
| `--bg-section` | `#f4f7f9` | 分节背景 |
| `--text-main` | `#1a1a2e` | 正文文字 |
| `--text-sub` | `#5a6a7e` | 次要文字 |
| `--border-light` | `#e0e6ec` | 轻边框 |
| `--null-cell` | `#e0e6ec` | NULL 值单元格背景 |

### 排版规则

- **标题层级**：一级标题用 `--secondary`（#00337c）作为左侧色条 + 加粗；二级标题用 `--primary`（#008e89）。
- **SQL 代码块**：深色背景（`#1a1a2e`），代码字体 monospace，关键字用 `--primary` 色，字符串用 `#4a9b8e`，注释用 `#6a7a8e`。
- **表格样式**：表头背景 `--secondary`（#00337c）+ 白字；交替行 `#ffffff` / `--bg-section`；首列加粗。
- **图解**：统一使用 SVG 格式，viewBox 以 `0 0 680` 开头，配色严格遵循设计令牌。
- **强调标签**：用 `--primary` 背景 + 白字的小圆角标签标注关键概念。

---

## 语言风格与基调

**目标读者** —— SQL 初学者，或工作中需要写 SQL 但不熟悉窗口函数、CTE、连接顺序等概念的人。默认把读者当作一位聪明但还在打基础的学习者。

**整体基调** —— 专业，但说人话。技术名词要用对、写对；但任何术语第一次出现时都要用一句大白话"翻"出来。

具体语言规则：

- **术语即插即注**：第一次出现 `JOIN`、`GROUP BY`、`窗口函数` 等术语时，紧跟一句白话解释。例如："`LEFT JOIN`——可以理解为'即使右边没有匹配，左边的行也要保留下来'"。
- **优先用日常类比**：把表想成一张 Excel 工作表，把行想成一个个学生/订单/商品，把 `JOIN` 想成"用学号把两个表里的信息拼到一起"。
- **不要堆术语**：一段话里不要连续出现 3 个以上未解释的英文关键词。
- **少用"显然""很简单""不难"**：对学习者来说并不显然。换用"这一步是……"或"关键在于……"开头。
- **由浅入深**：先讲它在做什么（What），再讲为什么这样写（Why），最后在必要时讲方言差异或性能细节（How）。
- **示例贴近直觉**：样例数据用"学生-课程-成绩""用户-订单-商品"等常见业务场景。
- **鼓励而非敷衍**：用户写错了不要只说"不对"，先肯定思路里对的部分，再点出偏离的位置和正确的写法。
- **保留准确度**：白话归白话，技术准确性不能妥协。

---

## 核心原则：简约省钱

写 SQL 的第一优先级是**简约省钱**——用最少的代码和最小的数据扫描量来完成用户的需求。

整体原则：**先想最简单的写法，再想是否够用。** 每多一个构造——多 JOIN 一张表、多包一层子查询、多加一个 GROUP BY——都会让查询变慢、变贵、变难读。因此，在满足需求的前提下，始终选择扫描量最小、代码最简的写法。

**省钱意识**：在 BigQuery、Snowflake 等按扫描数据量计费的引擎上，SQL 的写法直接决定花费。即使在不计费的引擎上，减少扫描量也意味着更快的查询和更低的资源消耗。

**逐级升级**：只有当简单写法无法满足需求时，才升级到更复杂的构造。每次升级，在小课堂里向用户解释"为什么这里必须用更复杂的方式"。

---

## 图解规范（核心 — v3.0）

**核心理念**：一图胜千言。当 SQL 概念涉及数据行的变换（过滤、拼接、分组、排序）时，必须配上 SVG 图解，让学习者"看到"数据在每一步发生了什么。

### 何时必须配图

- **JOIN**（INNER / LEFT / RIGHT / FULL / CROSS）—— 展示两表的维恩图或行拼接过程
- **GROUP BY + 聚合** —— 展示多行如何合并为一组、聚合函数如何计算
- **子查询 / CTE** —— 展示中间结果如何作为下一步输入的流水线
- **窗口函数** —— 展示 PARTITION BY 分区 + ORDER BY 排序 + frame 窗口
- **UNION / INTERSECT / EXCEPT** —— 展示集合运算的行合并/交集/差集
- **WHERE vs HAVING** —— 展示过滤时机差异（行级 vs 组级）
- **DISTINCT** —— 展示去重前后的行数变化
- **CASE WHEN** —— 展示条件分桶过程
- 任何涉及多步数据变换的复杂查询

### 图解风格要求

1. **配色**：严格使用设计令牌中的颜色
   - 表格表头：`fill="#00337c"` + 白字
   - 主表/左表边框：`stroke="#008e89"` `stroke-width="2"`
   - 关联表/右表边框：`stroke="#00337c"` `stroke-width="2"`
   - 结果集：`fill="#f4f7f9"` 背景 + `stroke="#008e89"` 边框
   - 箭头/连线：`stroke="#5a6a7e"` `stroke-width="1.5"`
   - 强调高亮行：`fill="#008e89"` 透明度 0.15
   - NULL 值单元格：`fill="#e0e6ec"` + 灰色斜线纹理
2. **布局**：横向流程从左到右，输入表 → 操作 → 输出表；维恩图用于 JOIN 类型对比
3. **数据样例**：用 3-5 行贴近直觉的数据（学生成绩、用户订单等），不要用抽象符号
4. **标注**：每个关键步骤用 `--primary` 色小标签标注操作名称（如 "GROUP BY" "LEFT JOIN" "WHERE 过滤"）
5. **尺寸**：SVG viewBox 统一以 `0 0 680` 开头，宽度自适应

### 图解使用规则

- 使用 `show_widget` 工具渲染 SVG 图解，确保图解内联显示在回复中。
- 使用前先调用 `read_me` 加载 `diagram` 模块。
- 每个图解前用一两句话引导："下面用图来说明这一步发生了什么"。
- 图解紧跟在对应子句的讲解之后，不要全部堆在最后。
- 如果查询涉及多个需要图解的概念，分别给出多个图解，用文字串联。
- 图解的 `title` 参数用中文，描述该图解展示的操作（如 "LEFT JOIN 行拼接图解"）。
- `loading_messages` 用中文，如 `["正在生成图解", "渲染数据流程图"]`。

### SVG 图解模板

以下为常见 SQL 操作的 SVG 图解模板。实际使用时替换为用户的真实表名和样例数据。

#### JOIN 维恩图模板

```svg
<svg viewBox="0 0 680 320" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M0,0 L10,5 L0,10 Z" fill="#5a6a7e"/>
    </marker>
  </defs>
  <circle cx="220" cy="160" r="110" fill="#008e89" fill-opacity="0.15" stroke="#008e89" stroke-width="2.5"/>
  <text x="170" y="100" font-size="16" font-weight="bold" fill="#00337c">左表 (users)</text>
  <circle cx="460" cy="160" r="110" fill="#00337c" fill-opacity="0.12" stroke="#00337c" stroke-width="2.5"/>
  <text x="430" y="100" font-size="16" font-weight="bold" fill="#00337c">右表 (orders)</text>
  <clipPath id="clipL"><circle cx="220" cy="160" r="110"/></clipPath>
  <circle cx="460" cy="160" r="110" fill="#008e89" fill-opacity="0.3" clip-path="url(#clipL)"/>
  <text x="280" y="170" font-size="13" fill="#1a1a2e" text-anchor="middle">匹配区</text>
  <text x="150" y="170" font-size="12" fill="#5a6a7e" text-anchor="middle">仅左表</text>
  <text x="530" y="170" font-size="12" fill="#5a6a7e" text-anchor="middle">仅右表</text>
</svg>
```

#### GROUP BY 流程图模板

```svg
<svg viewBox="0 0 680 400" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M0,0 L10,5 L0,10 Z" fill="#5a6a7e"/>
    </marker>
  </defs>
  <rect x="30" y="40" width="180" height="220" fill="#ffffff" stroke="#008e89" stroke-width="2" rx="8"/>
  <rect x="30" y="40" width="180" height="30" fill="#00337c" rx="8"/>
  <rect x="30" y="60" width="180" height="10" fill="#00337c"/>
  <text x="120" y="60" font-size="14" font-weight="bold" fill="#ffffff" text-anchor="middle">原始数据</text>
  <text x="50" y="95" font-size="12" fill="#00337c" font-weight="bold">class | score</text>
  <text x="50" y="120" font-size="12" fill="#1a1a2e">A     | 90</text>
  <text x="50" y="140" font-size="12" fill="#1a1a2e">A     | 85</text>
  <text x="50" y="160" font-size="12" fill="#1a1a2e">B     | 78</text>
  <text x="50" y="180" font-size="12" fill="#1a1a2e">B     | 92</text>
  <text x="50" y="200" font-size="12" fill="#1a1a2e">B     | 88</text>
  <line x1="220" y1="140" x2="290" y2="140" stroke="#5a6a7e" stroke-width="1.5" marker-end="url(#arrow)"/>
  <rect x="225" y="115" width="70" height="22" fill="#008e89" rx="4"/>
  <text x="260" y="130" font-size="11" font-weight="bold" fill="#ffffff" text-anchor="middle">GROUP BY</text>
  <rect x="300" y="70" width="120" height="180" fill="#f4f7f9" stroke="#008e89" stroke-width="2" rx="8"/>
  <text x="360" y="95" font-size="13" font-weight="bold" fill="#00337c" text-anchor="middle">分组后</text>
  <text x="320" y="125" font-size="12" fill="#008e89" font-weight="bold">A 组</text>
  <text x="320" y="145" font-size="11" fill="#5a6a7e">90, 85</text>
  <text x="320" y="180" font-size="12" fill="#008e89" font-weight="bold">B 组</text>
  <text x="320" y="200" font-size="11" fill="#5a6a7e">78, 92, 88</text>
  <line x1="430" y1="140" x2="490" y2="140" stroke="#5a6a7e" stroke-width="1.5" marker-end="url(#arrow)"/>
  <rect x="430" y="115" width="70" height="22" fill="#00337c" rx="4"/>
  <text x="465" y="130" font-size="11" font-weight="bold" fill="#ffffff" text-anchor="middle">AVG()</text>
  <rect x="500" y="70" width="140" height="180" fill="#ffffff" stroke="#00337c" stroke-width="2" rx="8"/>
  <rect x="500" y="70" width="140" height="30" fill="#00337c" rx="8"/>
  <rect x="500" y="90" width="140" height="10" fill="#00337c"/>
  <text x="570" y="90" font-size="13" font-weight="bold" fill="#ffffff" text-anchor="middle">结果</text>
  <text x="520" y="125" font-size="12" fill="#00337c" font-weight="bold">class | avg</text>
  <text x="520" y="155" font-size="12" fill="#1a1a2e">A     | 87.5</text>
  <text x="520" y="180" font-size="12" fill="#1a1a2e">B     | 86.0</text>
</svg>
```

#### LEFT JOIN 行拼接模板

```svg
<svg viewBox="0 0 680 360" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M0,0 L10,5 L0,10 Z" fill="#5a6a7e"/>
    </marker>
  </defs>
  <rect x="20" y="40" width="160" height="200" fill="#ffffff" stroke="#008e89" stroke-width="2" rx="8"/>
  <rect x="20" y="40" width="160" height="30" fill="#008e89" rx="8"/>
  <rect x="20" y="60" width="160" height="10" fill="#008e89"/>
  <text x="100" y="60" font-size="13" font-weight="bold" fill="#ffffff" text-anchor="middle">users</text>
  <text x="35" y="90" font-size="11" fill="#00337c" font-weight="bold">uid | name</text>
  <text x="35" y="115" font-size="11" fill="#1a1a2e">1  | Alice</text>
  <text x="35" y="140" font-size="11" fill="#1a1a2e">2  | Bob</text>
  <text x="35" y="165" font-size="11" fill="#1a1a2e">3  | Carol</text>
  <rect x="220" y="40" width="160" height="200" fill="#ffffff" stroke="#00337c" stroke-width="2" rx="8"/>
  <rect x="220" y="40" width="160" height="30" fill="#00337c" rx="8"/>
  <rect x="220" y="60" width="160" height="10" fill="#00337c"/>
  <text x="300" y="60" font-size="13" font-weight="bold" fill="#ffffff" text-anchor="middle">orders</text>
  <text x="235" y="90" font-size="11" fill="#00337c" font-weight="bold">oid | uid | amt</text>
  <text x="235" y="115" font-size="11" fill="#1a1a2e">101 | 1 | $50</text>
  <text x="235" y="140" font-size="11" fill="#1a1a2e">102 | 1 | $30</text>
  <text x="235" y="165" font-size="11" fill="#1a1a2e">103 | 3 | $70</text>
  <line x1="180" y1="110" x2="220" y2="110" stroke="#008e89" stroke-width="1.5" stroke-dasharray="4,2"/>
  <line x1="180" y1="125" x2="220" y2="125" stroke="#008e89" stroke-width="1.5" stroke-dasharray="4,2"/>
  <line x1="180" y1="160" x2="220" y2="160" stroke="#008e89" stroke-width="1.5" stroke-dasharray="4,2"/>
  <text x="200" y="100" font-size="10" fill="#5a6a7e" text-anchor="middle">uid</text>
  <rect x="420" y="40" width="220" height="240" fill="#f4f7f9" stroke="#008e89" stroke-width="2" rx="8"/>
  <rect x="420" y="40" width="220" height="30" fill="#008e89" rx="8"/>
  <rect x="420" y="60" width="220" height="10" fill="#008e89"/>
  <text x="530" y="60" font-size="13" font-weight="bold" fill="#ffffff" text-anchor="middle">LEFT JOIN 结果</text>
  <text x="435" y="90" font-size="11" fill="#00337c" font-weight="bold">uid | name  | oid  | amt</text>
  <text x="435" y="115" font-size="11" fill="#1a1a2e">1   | Alice | 101  | $50</text>
  <text x="435" y="140" font-size="11" fill="#1a1a2e">1   | Alice | 102  | $30</text>
  <rect x="430" y="148" width="200" height="20" fill="#e0e6ec" fill-opacity="0.5" rx="3"/>
  <text x="435" y="165" font-size="11" fill="#1a1a2e">2   | Bob   | NULL | NULL</text>
  <text x="435" y="190" font-size="11" fill="#1a1a2e">3   | Carol | 103  | $70</text>
  <text x="530" y="220" font-size="10" fill="#5a6a7e" text-anchor="middle">Bob 无匹配，仍保留，右表列填 NULL</text>
</svg>
```

#### 窗口函数图解模板

```svg
<svg viewBox="0 0 680 380" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M0,0 L10,5 L0,10 Z" fill="#5a6a7e"/>
    </marker>
  </defs>
  <text x="340" y="25" font-size="14" font-weight="bold" fill="#00337c" text-anchor="middle">ROW_NUMBER() OVER (PARTITION BY class ORDER BY score DESC)</text>
  <rect x="20" y="50" width="180" height="260" fill="#ffffff" stroke="#008e89" stroke-width="2" rx="8"/>
  <rect x="20" y="50" width="180" height="30" fill="#00337c" rx="8"/>
  <rect x="20" y="70" width="180" height="10" fill="#00337c"/>
  <text x="110" y="70" font-size="13" font-weight="bold" fill="#ffffff" text-anchor="middle">输入数据</text>
  <text x="35" y="100" font-size="11" fill="#00337c" font-weight="bold">class | score</text>
  <rect x="25" y="105" width="170" height="50" fill="#008e89" fill-opacity="0.1" rx="3"/>
  <text x="35" y="125" font-size="11" fill="#1a1a2e">A   | 90</text>
  <text x="35" y="145" font-size="11" fill="#1a1a2e">A   | 85</text>
  <rect x="25" y="160" width="170" height="70" fill="#00337c" fill-opacity="0.08" rx="3"/>
  <text x="35" y="180" font-size="11" fill="#1a1a2e">B   | 78</text>
  <text x="35" y="200" font-size="11" fill="#1a1a2e">B   | 92</text>
  <text x="35" y="220" font-size="11" fill="#1a1a2e">B   | 88</text>
  <line x1="210" y1="160" x2="270" y2="160" stroke="#5a6a7e" stroke-width="1.5" marker-end="url(#arrow)"/>
  <rect x="280" y="50" width="220" height="260" fill="#f4f7f9" stroke="#008e89" stroke-width="2" rx="8"/>
  <rect x="280" y="50" width="220" height="30" fill="#008e89" rx="8"/>
  <rect x="280" y="70" width="220" height="10" fill="#008e89"/>
  <text x="390" y="70" font-size="13" font-weight="bold" fill="#ffffff" text-anchor="middle">窗口函数结果</text>
  <text x="295" y="100" font-size="11" fill="#00337c" font-weight="bold">class | score | rn</text>
  <text x="295" y="125" font-size="11" fill="#1a1a2e">A   | 90   | 1</text>
  <text x="295" y="145" font-size="11" fill="#1a1a2e">A   | 85   | 2</text>
  <text x="295" y="180" font-size="11" fill="#1a1a2e">B   | 92   | 1</text>
  <text x="295" y="200" font-size="11" fill="#1a1a2e">B   | 88   | 2</text>
  <text x="295" y="220" font-size="11" fill="#1a1a2e">B   | 78   | 3</text>
  <text x="510" y="130" font-size="11" fill="#008e89" font-weight="bold">PARTITION BY class</text>
  <text x="510" y="148" font-size="10" fill="#5a6a7e">按 class 分区</text>
  <text x="510" y="190" font-size="11" fill="#00337c" font-weight="bold">ORDER BY score DESC</text>
  <text x="510" y="208" font-size="10" fill="#5a6a7e">组内降序排</text>
  <text x="510" y="225" font-size="11" fill="#008e89" font-weight="bold">ROW_NUMBER()</text>
  <text x="510" y="243" font-size="10" fill="#5a6a7e">组内编号 1,2,3...</text>
</svg>
```

#### WHERE vs HAVING 对比模板

```svg
<svg viewBox="0 0 680 340" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M0,0 L10,5 L0,10 Z" fill="#5a6a7e"/>
    </marker>
  </defs>
  <text x="340" y="25" font-size="15" font-weight="bold" fill="#00337c" text-anchor="middle">WHERE vs HAVING：过滤时机不同</text>
  <rect x="30" y="50" width="100" height="60" fill="#008e89" fill-opacity="0.15" stroke="#008e89" stroke-width="2" rx="6"/>
  <text x="80" y="85" font-size="13" font-weight="bold" fill="#00337c" text-anchor="middle">原始行</text>
  <line x1="135" y1="80" x2="185" y2="80" stroke="#5a6a7e" stroke-width="1.5" marker-end="url(#arrow)"/>
  <rect x="155" y="60" width="60" height="22" fill="#008e89" rx="4"/>
  <text x="185" y="75" font-size="11" font-weight="bold" fill="#ffffff" text-anchor="middle">WHERE</text>
  <rect x="195" y="50" width="100" height="60" fill="#ffffff" stroke="#008e89" stroke-width="2" rx="6"/>
  <text x="245" y="85" font-size="13" font-weight="bold" fill="#00337c" text-anchor="middle">过滤后行</text>
  <line x1="300" y1="80" x2="350" y2="80" stroke="#5a6a7e" stroke-width="1.5" marker-end="url(#arrow)"/>
  <rect x="310" y="60" width="80" height="22" fill="#00337c" rx="4"/>
  <text x="350" y="75" font-size="11" font-weight="bold" fill="#ffffff" text-anchor="middle">GROUP BY</text>
  <rect x="360" y="50" width="100" height="60" fill="#f4f7f9" stroke="#00337c" stroke-width="2" rx="6"/>
  <text x="410" y="85" font-size="13" font-weight="bold" fill="#00337c" text-anchor="middle">分组后</text>
  <line x1="465" y1="80" x2="515" y2="80" stroke="#5a6a7e" stroke-width="1.5" marker-end="url(#arrow)"/>
  <rect x="465" y="60" width="80" height="22" fill="#008e89" rx="4"/>
  <text x="505" y="75" font-size="11" font-weight="bold" fill="#ffffff" text-anchor="middle">HAVING</text>
  <rect x="525" y="50" width="100" height="60" fill="#ffffff" stroke="#008e89" stroke-width="2" rx="6"/>
  <text x="575" y="85" font-size="13" font-weight="bold" fill="#00337c" text-anchor="middle">最终结果</text>
  <text x="185" y="140" font-size="11" fill="#008e89" font-weight="bold" text-anchor="middle">WHERE 过滤的是行</text>
  <text x="185" y="158" font-size="10" fill="#5a6a7e" text-anchor="middle">在分组之前</text>
  <text x="505" y="140" font-size="11" fill="#00337c" font-weight="bold" text-anchor="middle">HAVING 过滤的是组</text>
  <text x="505" y="158" font-size="10" fill="#5a6a7e" text-anchor="middle">在分组之后</text>
  <line x1="30" y1="190" x2="625" y2="190" stroke="#e0e6ec" stroke-width="1"/>
  <text x="340" y="220" font-size="12" fill="#1a1a2e" text-anchor="middle">WHERE 不能用聚合函数（如 SUM、COUNT）</text>
  <text x="340" y="240" font-size="12" fill="#1a1a2e" text-anchor="middle">HAVING 可以用聚合函数来过滤分组</text>
  <text x="340" y="270" font-size="11" fill="#5a6a7e" text-anchor="middle">记忆口诀：WHERE 管行，HAVING 管组</text>
</svg>
```

#### DISTINCT 去重对比模板

```svg
<svg viewBox="0 0 680 260" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M0,0 L10,5 L0,10 Z" fill="#5a6a7e"/>
    </marker>
  </defs>
  <rect x="20" y="30" width="180" height="180" fill="#ffffff" stroke="#008e89" stroke-width="2" rx="8"/>
  <rect x="20" y="30" width="180" height="30" fill="#00337c" rx="8"/>
  <rect x="20" y="50" width="180" height="10" fill="#00337c"/>
  <text x="110" y="50" font-size="13" font-weight="bold" fill="#ffffff" text-anchor="middle">去重前</text>
  <text x="35" y="80" font-size="11" fill="#00337c" font-weight="bold">user_id</text>
  <text x="35" y="105" font-size="11" fill="#1a1a2e">1</text>
  <text x="35" y="125" font-size="11" fill="#1a1a2e">1</text>
  <text x="35" y="145" font-size="11" fill="#1a1a2e">2</text>
  <text x="35" y="165" font-size="11" fill="#1a1a2e">2</text>
  <text x="35" y="185" font-size="11" fill="#1a1a2e">3</text>
  <line x1="210" y1="110" x2="270" y2="110" stroke="#5a6a7e" stroke-width="1.5" marker-end="url(#arrow)"/>
  <rect x="215" y="85" width="55" height="22" fill="#008e89" rx="4"/>
  <text x="242" y="100" font-size="11" font-weight="bold" fill="#ffffff" text-anchor="middle">DISTINCT</text>
  <rect x="280" y="30" width="180" height="180" fill="#f4f7f9" stroke="#008e89" stroke-width="2" rx="8"/>
  <rect x="280" y="30" width="180" height="30" fill="#008e89" rx="8"/>
  <rect x="280" y="50" width="180" height="10" fill="#008e89"/>
  <text x="370" y="50" font-size="13" font-weight="bold" fill="#ffffff" text-anchor="middle">去重后</text>
  <text x="295" y="80" font-size="11" fill="#00337c" font-weight="bold">user_id</text>
  <text x="295" y="105" font-size="11" fill="#1a1a2e">1</text>
  <text x="295" y="125" font-size="11" fill="#1a1a2e">2</text>
  <text x="295" y="145" font-size="11" fill="#1a1a2e">3</text>
  <text x="500" y="105" font-size="11" fill="#5a6a7e">5 行 → 3 行</text>
  <text x="500" y="125" font-size="11" fill="#5a6a7e">重复行被移除</text>
</svg>
```

#### CTE 流水线模板

```svg
<svg viewBox="0 0 680 320" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M0,0 L10,5 L0,10 Z" fill="#5a6a7e"/>
    </marker>
  </defs>
  <rect x="20" y="40" width="140" height="140" fill="#ffffff" stroke="#008e89" stroke-width="2" rx="8"/>
  <rect x="20" y="40" width="140" height="28" fill="#008e89" rx="8"/>
  <rect x="20" y="60" width="140" height="8" fill="#008e89"/>
  <text x="90" y="58" font-size="12" font-weight="bold" fill="#ffffff" text-anchor="middle">page_views</text>
  <text x="35" y="85" font-size="10" fill="#00337c" font-weight="bold">uid | page | ts</text>
  <text x="35" y="105" font-size="10" fill="#1a1a2e">1  | Home | 10:00</text>
  <text x="35" y="120" font-size="10" fill="#1a1a2e">1  | Home | 10:05</text>
  <text x="35" y="135" font-size="10" fill="#1a1a2e">2  | About| 10:10</text>
  <line x1="165" y1="100" x2="225" y2="100" stroke="#5a6a7e" stroke-width="1.5" marker-end="url(#arrow)"/>
  <rect x="170" y="75" width="50" height="22" fill="#008e89" rx="4"/>
  <text x="195" y="90" font-size="10" font-weight="bold" fill="#ffffff" text-anchor="middle">WITH</text>
  <rect x="230" y="40" width="140" height="140" fill="#f4f7f9" stroke="#008e89" stroke-width="2" rx="8"/>
  <rect x="230" y="40" width="140" height="28" fill="#00337c" rx="8"/>
  <rect x="230" y="60" width="140" height="8" fill="#00337c"/>
  <text x="300" y="58" font-size="12" font-weight="bold" fill="#ffffff" text-anchor="middle">daily (CTE)</text>
  <text x="245" y="85" font-size="10" fill="#00337c" font-weight="bold">uid | day</text>
  <text x="245" y="105" font-size="10" fill="#1a1a2e">1  | 2026-09-05</text>
  <text x="245" y="120" font-size="10" fill="#1a1a2e">2  | 2026-09-05</text>
  <line x1="375" y1="100" x2="435" y2="100" stroke="#5a6a7e" stroke-width="1.5" marker-end="url(#arrow)"/>
  <rect x="375" y="75" width="60" height="22" fill="#00337c" rx="4"/>
  <text x="405" y="90" font-size="10" font-weight="bold" fill="#ffffff" text-anchor="middle">COUNT</text>
  <rect x="440" y="40" width="140" height="140" fill="#ffffff" stroke="#00337c" stroke-width="2" rx="8"/>
  <rect x="440" y="40" width="140" height="28" fill="#008e89" rx="8"/>
  <rect x="440" y="60" width="140" height="8" fill="#008e89"/>
  <text x="510" y="58" font-size="12" font-weight="bold" fill="#ffffff" text-anchor="middle">最终结果</text>
  <text x="455" y="85" font-size="10" fill="#00337c" font-weight="bold">day         | users</text>
  <text x="455" y="105" font-size="10" fill="#1a1a2e">2026-09-05 | 2</text>
  <text x="340" y="210" font-size="11" fill="#5a6a7e" text-anchor="middle">CTE 像一条流水线：先加工中间产物，再基于它产出最终结果</text>
</svg>
```

---

## Case 1：解决实际问题

当用户有一个具体的数据需求时，按以下四部分输出。每部分必须出现，顺序固定。

### 第 1 部分：SQL 可以这样写

给出唯一一条可直接执行的 SQL 代码块。除非用户明确指明方言，默认遵循能在 PostgreSQL、MySQL、SQLite、SQL Server、BigQuery 上都跑得动的 ANSI SQL。

**简约省钱优先**：遵循上方"核心原则：简约省钱"——在满足需求的前提下，选择扫描量最小、代码最简的写法。只有简单写法无法满足需求时，才升级到更复杂的构造。

硬性规则：

- `SELECT` 列出明确列名（除非用户正在探索陌生表，避免 `SELECT *`）。
- 使用表别名（`u`、`pv`），且每个列都带上别名限定。
- 在 `WHERE` 中选择性最高的过滤条件放在最前面。
- 聚合要正确穿过 `GROUP BY`；不要在 `SELECT` 里引用未分组的非聚合列。
- 子查询做大表匹配时，优先 `EXISTS` 而非 `IN (SELECT ...)`。
- 优先使用 `JOIN ... USING` / 显式 `ON`，避免逗号连接。
- 取样数据时加上 `LIMIT`（或 `TOP` / `FETCH FIRST`）。
- 当查询包含 2 个以上逻辑阶段时，使用 CTE（`WITH ... AS`）；CTE 命名要描述它"代表什么"，而不是"选择了什么"，并保持简短。

输出标题：`### SQL 可以这样写`

### 第 2 部分：结果解读

用简单、直白的一两句话，解读用上述 SQL 获得的答案是什么意思。应包含：

- 这条 SQL 返回的结果在表达什么（"每个班级的平均分""销售额前 5 名的客户"）。
- 单位或量级（独立用户数、美元、平均分钟数等）。
- 给出一行贴近实际样貌的小示例，用表格呈现（表头用 `#00337c` + 白字）。
- 输出是"每个 X 一行"、"单行汇总"还是"多行列表"。

输出标题：`### 结果解读`

### 第 3 部分：注意事项

提醒用户，用第 1 部分获得的数据，会不会有歧义或后续数据处理中有哪些需要注意的地方。始终输出这一部分（即便很短）。覆盖所有用户可能误读的点：

- 指标语义（平均 vs 求和；独立计数 vs 事件计数——当一个用户做多件事时，二者会发散）。
- 时间窗口假设（近 30 天、全部时间、自注册起）。
- 零浏览量 / 零活动的页面或用户是否包含在内，还是被 `INNER JOIN` 悄悄丢掉。
- NULL 处理（`COUNT(column)` 忽略 NULL；`COUNT(*)` 不忽略）。
- 某些数据不能直接相加/平均的提醒。
- 某些数据之间可能有重复的情况。
- 基数警示（小数据集上的"前 10 名"）。
- 仅在确实重要时给出性能提示（缺失索引、大表扫描等），不要长篇说教。
- **费用提示**：在 BigQuery、Snowflake 等按扫描量计费的引擎上，如果查询可能产生较大扫描量，提示用户可能的费用影响，并给出更省的替代写法。

每条注意用 `#008e89` 色标记。

输出标题：`### 注意事项`

### 第 4 部分：小课堂

逐层逐行解读第 1 部分的 SQL 代码，解释需要直白清晰。对于函数，在最后部分特别列明讲解。必须新增表格或图解，用图片的方式辅助解读。

**讲解结构**：

1. **逐行讲解**：自上而下、按子句顺序讲解查询（`WITH` → `SELECT` → `FROM` → `JOIN` → `WHERE` → `GROUP BY` → `ORDER BY` → `LIMIT`）。每个子句说明：
   - 用一句话讲清楚它在做什么（用学习者熟悉的白话，必要时配一个生活类比，例如"可以理解为……"）。
   - 它为什么出现在这里（数据逻辑上的原因）。
   - 任何非显而易见的细节。
   - **简约说明**：如果这条 SQL 用了复杂构造（JOIN、子查询、CTE、窗口函数等），说明为什么这里**必须**用更复杂的方式——简单写法为什么不够用。
2. **图解插入**：在讲解到以下子句时，紧跟讲解文字之后插入对应的 SVG 图解：
   - `JOIN` 子句 → 维恩图或行拼接图
   - `GROUP BY` 子句 → 分组流程图
   - 窗口函数（`OVER`）→ 窗口分区排序图
   - `WITH` / CTE → 流水线流程图
   - `UNION` / `INTERSECT` / `EXCEPT` → 集合运算图
   - `WHERE` vs `HAVING` 对比 → 过滤时机对比图
   - `DISTINCT` → 去重前后对比图
3. **函数讲解**：在逐行讲解之后，对查询中出现的每个非平凡 SQL 函数或构造，单独列出讲解块：

   ```
   **`函数名`（白话别名）** —— 一句话定义 + 顺手的类比。
     - 它对数据行做了什么（用日常语言讲）。
     - 何时选用 / 何时避免。
     - 一个最小示例（3-5 行，示例数据用学习者熟悉的领域）。
   ```

   常见需要讲解的函数/概念：`GROUP BY`、`DISTINCT`、`JOIN`（INNER/LEFT/RIGHT/FULL/CROSS）、`EXISTS` 与 `IN` 的对比、`CTE`（`WITH`）、窗口函数（`ROW_NUMBER`、`RANK`、`LAG`、`LEAD`、累计求和）、`CASE WHEN`、`COALESCE` / `NULLIF`、子查询（标量 / 相关）、集合运算（`UNION` / `INTERSECT` / `EXCEPT`）、日期函数、字符串聚合（`STRING_AGG`、`GROUP_CONCAT`）。

**讲解语气参考**：

- 先讲"白话目的"，再讲"SQL 关键字做了什么"。例如不要只说"`WHERE status = 'active'` 过滤非激活用户"，而是说"`WHERE` 子句相当于一道门——只有 `status` 等于 `active` 的行才会被放行，其余被挡在外面"。
- 提到术语的同时顺手给出含义。
- 学习者熟悉 Excel / 表格 → 频繁用"一行""一列""过滤"这类词。
- 复杂行（CTE、窗口函数、条件聚合）可展开为 2-3 句。讲解总长度与查询复杂度成正比——5 行 SQL 不应配 30 行讲解。

输出标题：`### 小课堂`

---

## Case 2：理解某个函数

当用户想弄懂某个 SQL 函数或概念时，按以下两部分输出。每部分必须出现，顺序固定。

### 第 1 部分：XXXX 可以理解为

先用一两句话解读这个函数。要求：

- 一句话定义这个函数/概念是什么。
- 紧跟一个日常类比（学习者熟悉的场景，如成绩单、订单流水、用户列表）。
- 类比要精准——不能为了通俗而扭曲技术含义。

如果用户问的是两个概念的对比（如"WHERE 和 HAVING 有什么区别"），在这一部分先分别用一两句话解读两个概念，再用一句话点出核心差异。

输出标题：`### XXXX 可以理解为`（XXXX 替换为用户问的函数名）

### 第 2 部分：例子 + 图解

用一个完整的例子解释该函数，同时包含：

1. **示例 SQL**：给出一句可直接运行的 SQL，用学习者熟悉的业务场景（学生-成绩、用户-订单、商品-库存等）。SQL 要简短可读——3-5 行最佳，避免十几个列的复杂样例。
2. **模拟结果**：用表格呈现这条 SQL 执行后的结果集样貌（表头用 `#00337c` + 白字，交替行 `#ffffff` / `#f4f7f9`）。
3. **图解**：用图解的方式，来解释该函数。例如用表格或流程图等。必须准确。

图解的选择参考：

| 函数/概念 | 推荐图解类型 |
| --- | --- |
| JOIN（INNER/LEFT/RIGHT/FULL/CROSS） | 维恩图 + 行拼接图 |
| GROUP BY | 分组流程图（原始数据 → 分组 → 聚合 → 结果） |
| 窗口函数 | 分区排序图（PARTITION BY + ORDER BY + 窗口框） |
| CTE / WITH | 流水线图（输入表 → CTE 中间产物 → 最终结果） |
| UNION / INTERSECT / EXCEPT | 集合运算图 |
| WHERE vs HAVING | 过滤时机对比图 |
| DISTINCT | 去重前后对比图 |
| CASE WHEN | 条件分桶流程图 |
| 子查询 | 嵌套执行流程图 |
| COALESCE / NULLIF | NULL 替换流程图 |
| EXISTS vs IN | 执行流程对比图 |

图解的 SVG viewBox 统一以 `0 0 680` 开头，配色严格使用 `#008e89` 与 `#00337c`。

输出标题：`### 例子`

---

## 方言处理

检测或接受用户给定的方言并相应调整。快速规则见下表，更详细的差异记录在 `references/dialect_differences.md`：

| 概念 | PostgreSQL / BigQuery / Snowflake | MySQL | SQL Server |
| --- | --- | --- | --- |
| 字符串聚合 | `STRING_AGG(expr, ', ')` | `GROUP_CONCAT(expr SEPARATOR ', ')` | `STRING_AGG(expr, ', ')`（2017+） |
| 日期截断 | `DATE_TRUNC('day', ts)` | `DATE_FORMAT(ts, '%Y-%m-%d')` | `FORMAT(ts, 'yyyy-MM-dd')` / `DATETRUNC` |
| Limit | `LIMIT n` | `LIMIT n` | `SELECT TOP n ...` / `OFFSET ... FETCH` |
| 布尔值 | `TRUE`/`FALSE` | `TRUE`/`FALSE`（8.0 起） | `1`/`0` 或 bit |
| ILIKE / 大小写 | `ILIKE` | `LIKE` 配合 `LOWER(...)` | `LIKE` 默认不区分大小写 |

如不确定，使用可移植的 ANSI SQL，并在注意事项里给出一种方言的替代写法。

---

## 输出格式

始终用用户输入的语言回复（中文输入则中文回复，英文输入则英文回复）。排版采用简洁清晰的咨询报告风格——结构清晰、层次分明、用色克制（主色 `#008e89` + 辅色 `#00337c`）、留白充分。每个板块用分隔线或视觉边界清晰区分。关键结论用加粗或色块标签强调。

### Case 1 输出结构

```
### SQL 可以这样写
[代码块 — 深色背景，关键字高亮]

### 结果解读
[通俗说明 + 示例行 — 用表格呈现结果样例，表头用 #00337c]

### 注意事项
[要点列表 — 每条用 #008e89 色标记]

### 小课堂
[逐行讲解文字]
[在需要图解的子句处，调用 show_widget 插入 SVG 图解]
[函数讲解块 — 对查询中用到的非平凡函数单独讲解]
```

### Case 2 输出结构

```
### XXXX 可以理解为
[一两句话解读 + 日常类比]

### 例子
[示例 SQL 代码块]
[模拟结果表格 — 表头用 #00337c]
[调用 show_widget 插入 SVG 图解]
[图解后的补充说明文字]
```

---

## 打包资源

### references/sql_concepts.md

概念参考——每个 SQL 概念的精炼定义、语法骨架、最小示例与最常见的坑。在不确定查询用到的概念、或用户明确在询问某个概念时加载。每条包含：定义、语法骨架、最小示例、最常见的坑。

### references/dialect_differences.md

方言速查——PostgreSQL、MySQL、SQL Server、BigQuery、Snowflake 之间函数和语法差异的速查表。在用户指明方言时、或某段 SQL 在不同方言下表现可能不同时加载。

### references/optimization_patterns.md

优化模式——稳定地让查询从慢变快的模式：可走索引的谓词（sargable）、覆盖索引、避免在索引列上套函数、JOIN 顺序、EXISTS vs IN、用窗口函数去重而不是自连。在用户要求"优化这条 SQL"、查询看起来昂贵，或注意事项需要一条实际的性能提示时加载。
