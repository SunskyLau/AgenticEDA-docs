# Insight Taxonomy v3

本版目标：为 Agentic EDA 框架定义一套**原子化、单标签**的 insight 分类体系，用于 Coverage Map 的 taxonomy 轴、summarizer 的结构化输出、以及论文中的 insight 分类叙事。

旧版文档保留为：`Insight Taxonomy.v1.md`（多标签讨论）、`Insight Taxonomy.v2.md`（首版 predominant type）。

---

## 1. 学术来源与推导逻辑

### 1.1 设计策略

本 taxonomy 以 **DataShot 的 data fact taxonomy 为骨架**，最大程度保留其原子事实类型的定义与命名，仅针对 EDA 场景做最小化适配。同时引用 CoInsight 验证覆盖完整性、arXiv 2404.01644 验证 EDA 场景适用性。

这一策略的核心理由：DataShot 的 11 类 fact type 是目前数据叙事（data storytelling）领域引用最广的 fact-level 分类体系，与我们 summarizer 产出的 atomic insight 处于同一粒度（原子数据事实）。直接继承而非重新发明，既保证了学术可追溯性，也降低了 reviewer 的认知成本。

### 1.2 三条文献主线

**主线 A：DataShot — 数据事实原子类型（Data Fact Taxonomy）**

DataShot（Microsoft Research, IEEE TVCG 2020）从大规模 infographic 语料中实证推导出 11 类可组合的数据事实类型：

```
Value / Aggregation / Proportion / Rank / Difference / Trend /
Distribution / Association / Outlier / Extreme / Categorization
```

核心设计原则：每个类型描述一种**原子数据关系**（而非统计方法或可视化形式）。本 taxonomy 以 DataShot 为骨架，直接保留其中 9 类的语义与命名。

**主线 B：CoInsight — 洞察模式层级（Insight Pattern Hierarchy）**

CoInsight（IEEE TVCG 2024）提出三层洞察模式分组：

```
Point:    Value, Extreme, Outlier
Shape:    Trend, Skewness, Kurtosis, Evenness, Seasonality
Compound: Dominance, Top2, Correlation, Similarity, ChangePoint
```

CoInsight 与 DataShot 的关系：CoInsight 的分类对象是数值序列的统计模式（pattern-level），粒度比 DataShot 的原子事实（fact-level）更细。其 Shape 层的 Skewness/Kurtosis/Evenness 对应 DataShot 的 Distribution 的子属性，Compound 层的 Dominance/Top2 是原子类型的组合。

本 taxonomy 用 CoInsight 做两件事：(1) 验证覆盖完整性（13/13 全覆盖）；(2) 引入 `cluster` 类型（对应 CoInsight 的 Similarity 模式）。

**主线 C：arXiv 2404.01644 — EDA 场景实践验证**

该论文明确参考 DataShot 构建 EDA insight 分类，实际使用 7 类：

```
difference / value / trend / outlier / distribution / rank / correlation
```

这为本 taxonomy 的 EDA 场景适用性提供了直接的实践验证。

### 1.3 从 DataShot 到 v3 的推导路径

```
DataShot 11 类（原子事实类型）
  │
  ├─ 8 类直接保留 ──────────────── proportion, rank, difference, trend,
  │                                 distribution, association, outlier, extreme
  │
  ├─ 1 对有据合并 ──────────────── Value + Aggregation → value
  │
  ├─ 1 类有据吸收 ──────────────── Categorization → 非原子，分解归入其他类型
  │
  ├─ 1 类从 CoInsight 扩展 ─────── Similarity → cluster
  │
  └─ 1 类从项目实证扩展 ─────────── data_quality（run 高频 11.3%）
                                     ↓
                              v3：11 类原子类型
```

**合并依据：**
- Value + Aggregation → `value`：两者都回答"数值是多少"，区别仅在于是原始值还是计算值。在自动化 EDA 中，agent 产出的几乎全是计算值（均值、计数、总量），这一区分没有操作意义。

**吸收依据：**
- Categorization 不是原子关系类型，而是"其他类型应用于类别变量"的复合表现。"有 12 个类别"是 `value`，"Action 最多"是 `rank`，"Action 占 35%"是 `proportion`，"类别分布偏斜"是 `distribution`。v2 中 `categorical_profile` 仅占 2.1% 也印证了这一点。

**扩展依据：**
- `cluster`：CoInsight 的 Similarity 模式 + Amar & Stasko (InfoVis 2005) 的 Cluster 分析任务，两者共同支撑。EDA 中聚类/分群是常见分析产出，DataShot 的 infographic 语料中缺少此类。
- `data_quality`：DataShot/CoInsight 均未覆盖（两者面向的是数据展示/叙事场景，不涉及数据清洗）。但在 EDA 场景中，数据质量发现（缺失率、一致性问题）是标准产出，本项目 run 数据中占 11.3%，直接影响其他 insight 的可信度。

---

## 2. Taxonomy 定义（11 类）

### 2.1 统一分类维度

所有类型统一回答一个问题：**这条 atomic insight 的主句描述了什么类型的数据关系/模式？**

分类依据是 insight 主句的**谓语语义**（描述了什么关系），而非使用了什么统计方法、涉及什么变量类型、或采用了什么可视化形式。

### 2.2 类型定义表

| # | Type ID | 定义 | 包含范围 | 排除边界 | 学术锚点 |
|---|---|---|---|---|---|
| 1 | `value` | 描述数据某一方面的单个统计量、计数或聚合值，不涉及排序、对比、极值标识或时间变化 | 均值、中位数、总量、计数、标准差、R²、单个 p-value | 出现"最高/最低/峰值"→ extreme；"A vs B"→ difference；"增长/下降"→ trend | DataShot `Value` + `Aggregation` |
| 2 | `proportion` | 描述部分与整体之间的份额、百分比或构成关系 | 市场份额、占比、百分比构成、部分-整体分解 | 百分比变化（"增长了 15%"）→ trend；位置性主张（"A 以 30% 领先"）→ rank | DataShot `Proportion` |
| 3 | `rank` | 描述多个实体间的排序或位置关系，强调相对顺序 | Top-K、排名、第一/最后、领先/落后、多实体排列 | 仅标识单个极值点 → extreme；强调差距大小 → difference；强调份额 → proportion | DataShot `Rank`; CoInsight `Top2` |
| 4 | `difference` | 描述特定组、条件或时间点之间的对比，强调差距的方向或幅度 | A vs B、组间差异、gap、显著性差异结论 | 强调时间方向性 → trend；强调位置 → rank | DataShot `Difference` |
| 5 | `trend` | 描述沿时间或有序维度的方向性变化，包括增长、衰退、周期性或结构突变 | 年度趋势、季节性、变化点、持续增长/下降 | 两个时间点的静态对比 → difference；标识时序中的峰/谷点 → extreme | DataShot `Trend`; CoInsight `Trend` / `Seasonality` / `ChangePoint` |
| 6 | `distribution` | 描述一组值的形态、离散度、集中度或统计特性 | 偏态、峰态、Gini 系数、重尾、分位数、直方图形态 | 指向具体异常观测 → outlier；标识极端边界值 → extreme | DataShot `Distribution`; CoInsight `Skewness` / `Kurtosis` / `Evenness` |
| 7 | `association` | 描述两个或多个变量之间的协变、依赖或共现关系 | 相关系数、回归关系、共现模式、变量间依赖 | 离散组间对比 → difference | DataShot `Association`; CoInsight `Correlation` |
| 8 | `outlier` | 描述显著偏离一般模式的异常观测，强调其相对于整体分布/趋势的偏离性 | 统计异常点、离群值、罕见事件、偏离趋势线的观测 | 描述整体分布形态 → distribution；仅标识最大/最小值而无偏离含义 → extreme | DataShot `Outlier`; CoInsight `Outlier` |
| 9 | `extreme` | 描述数据中的最大值、最小值、峰值或谷值，标识数据范围的边界点 | 最大/最小值、峰值、谷值、历史新高/新低、记录值 | 多实体排序 → rank；偏离整体模式的异常 → outlier；无极值修饰的普通统计量 → value | DataShot `Extreme`; CoInsight `Extreme` |
| 10 | `cluster` | 描述数据中的自然分组结构或实体间的相似性模式 | 聚类结果、自然分群、相似性模式、分段结构 | 组间差异描述 → difference；变量协变 → association | CoInsight `Similarity`; Amar & Stasko `Cluster` |
| 11 | `data_quality` | 描述数据完整性、一致性或有效性方面的问题 | 缺失值、重复记录、无效值、数据不一致 | 如果缺失率仅作为背景提及而非主句核心 → 按主句关系归入其他类型 | 项目扩展（实证频率 11.3%） |

### 2.3 `extreme` 与 `outlier` 的区分说明

DataShot 将 Extreme 和 Outlier 定义为两个独立的原子事实类型，本 taxonomy 保留这一区分。两者的核心语义差异：

- **`extreme`**：标识数据范围的**边界点**（最大值、最小值、峰值、谷值）。极值不一定异常——它可能完全符合整体模式，只是恰好处于边界位置。
- **`outlier`**：标识**偏离整体模式**的异常观测。异常点不一定是极值——中间范围的值如果偏离趋势线也是 outlier。

| 表述 | 核心语义 | 归入类型 |
|---|---|---|
| "2019 年销售额达到峰值 $5M" | 标识时序边界点 | `extreme` |
| "3 个产品销售额超过中位数 10 倍" | 偏离整体分布 | `outlier` |
| "Nintendo 以 $5M 领先所有厂商" | 多实体排序 | `rank` |
| "平均销售额为 $3M" | 报告统计量 | `value` |

### 2.4 关于 `data_quality` 的定位说明

`data_quality` 是本 taxonomy 中唯一不来自 DataShot/CoInsight 的类型。保留它的理由：

1. **实证频率**：在 195 条校准 insight 中占 11.3%，是第六高频类型，高于 distribution (4.6%)、value (2.1%) 等文献原生类型。
2. **EDA 场景必要性**：数据质量发现（缺失率、一致性问题）是 EDA 的标准产出，直接影响其他 insight 的可信度。DataShot/CoInsight 面向数据展示/叙事场景，不涉及数据清洗，因此未覆盖此类型。
3. **Coverage Map 价值**：在 coverage grid 中，data_quality 行揭示"哪些列的数据质量已被评估"，这是用户决策的重要信息。

在论文中建议表述为：前 10 类直接来自或可追溯到 DataShot/CoInsight 的原子事实类型；`data_quality` 为面向 EDA 场景的实证驱动扩展。

---

## 3. 文献覆盖映射

### 3.1 DataShot 11 类 → v3 映射

```
DataShot                v3                处理方式
─────────────────────────────────────────────────────
Value            ──→    value             合并（与 Aggregation 合并）
Aggregation      ──→    value             合并（都回答"数值是多少"）
Proportion       ──→    proportion        1:1 直接保留
Rank             ──→    rank              1:1 直接保留
Difference       ──→    difference        1:1 直接保留
Trend            ──→    trend             1:1 直接保留
Distribution     ──→    distribution      1:1 直接保留
Association      ──→    association       1:1 直接保留
Outlier          ──→    outlier           1:1 直接保留
Extreme          ──→    extreme           1:1 直接保留
Categorization   ──→    (吸收)            非原子类型，按谓语关系归入
                                          value/rank/proportion/distribution
```

覆盖率：11/11（8 直接保留 + 1 有据合并 + 1 有据吸收 + 1 扩展来源）。

对 DataShot 的改动极小：仅合并 Value+Aggregation（EDA 场景无区分必要）、吸收 Categorization（非原子类型）。其余 8 类原样保留语义与命名。

### 3.2 CoInsight 细分类型 → v3 映射

```
CoInsight               v3                映射说明
─────────────────────────────────────────────────────
Point
  Value          ──→    value             直接对应
  Extreme        ──→    extreme           直接对应
  Outlier        ──→    outlier           直接对应

Shape
  Trend          ──→    trend             直接对应
  Skewness       ──→    distribution      分布形态的子属性
  Kurtosis       ──→    distribution      分布形态的子属性
  Evenness       ──→    distribution      分布形态的子属性
  Seasonality    ──→    trend             时序方向性变化的子模式

Compound
  Dominance      ──→    rank / proportion 按谓语消歧（见第 4 节）
  Top2           ──→    rank              直接对应
  Correlation    ──→    association       直接对应
  Similarity     ──→    cluster           直接对应
  ChangePoint    ──→    trend             时序结构突变的子模式
```

覆盖率：13/13 全覆盖。

### 3.3 arXiv 2404.01644 的 7 类 → v3 映射

```
arXiv 2404.01644        v3                映射说明
─────────────────────────────────────────────────────
difference       ──→    difference        直接对应
value            ──→    value             直接对应
trend            ──→    trend             直接对应
outlier          ──→    outlier           直接对应
distribution     ──→    distribution      直接对应
rank             ──→    rank              直接对应
correlation      ──→    association       直接对应
```

覆盖率：7/7 全覆盖。

---

## 4. 消歧协议（Disambiguation Protocol）

### 4.1 为什么需要消歧

自然语言 insight 可以同时包含多种数据关系。例如"Nintendo 以 20% 市场份额领先"同时涉及 rank（谁领先）和 proportion（占多少）。DataShot 将此类情况称为 compound fact。

本 taxonomy 要求每条 atomic insight 分配**唯一主类型**（predominant type）。为保证标注一致性，定义以下两步消歧协议。

### 4.2 第一步：主谓语关系判定

看 insight 主句的谓语动词/形容词描述的是什么数据关系：

| 谓语信号 | 判定类型 | 示例 |
|---|---|---|
| ranks, leads, tops, #1, dominant, 排名前 K | `rank` | "Nintendo ranks #1 in global sales." |
| accounts for, comprises, split as, X% of, share | `proportion` | "Action games account for 35% of titles." |
| higher/lower than, vs, gap, exceeds, differs, outperforms | `difference` | "Region A outperforms Region B by 2x." |
| grew, declined, increased, over time, since, year-over-year | `trend` | "Sales declined 15% year-over-year." |
| correlates, associated with, co-varies, relationship between | `association` | "Price and rating show positive correlation." |
| skewed, concentrated, distributed, spread, heavy-tail | `distribution` | "Sales follow a right-skewed distribution." |
| anomalous, outlier, deviates, rare, unexpected, unusual | `outlier` | "Three products have sales >10x the median." |
| peak, max, min, highest, lowest, record, 峰值, 谷值 | `extreme` | "Sales peaked at $5M in 2019." |
| clusters, segments, groups into, similar to, resembles | `cluster` | "K-means reveals 3 distinct customer segments." |
| is, equals, totals, averages（无对比/排序/极值/时间修饰） | `value` | "The average price is $45." |
| missing, null, incomplete, invalid, duplicate, inconsistent | `data_quality` | "35% of revenue values are missing." |

### 4.3 第二步：优先级裁决（Tie-breaking）

当主谓语信号不明确或同时命中多个类型时，按以下固定顺序，首个命中即返回：

```
 1. data_quality    ← 数据可靠性问题优先于分析发现
 2. trend           ← 时间/序列方向性是最强的语义信号
 3. association     ← 变量间协变关系
 4. difference      ← 组间对比
 5. outlier         ← 偏离模式的异常观测
 6. extreme         ← 数据范围的边界极值
 7. distribution    ← 值的形态
 8. rank            ← 位置关系
 9. proportion      ← 份额关系
10. cluster         ← 自然分组
11. value           ← 兜底：单一统计量
```

**优先级设计理由：**
- `data_quality` 最高：如果主句核心是"数据有问题"，附带的统计量都是为了说明数据问题。
- `trend` 高于 `difference`：时间方向性是明确的语义信号，不易误判。
- `outlier` 高于 `extreme`：如果一个极值同时被描述为偏离整体模式，偏离性是更强的语义信号。
- `value` 最低（兜底）：任何 insight 都包含数值，但只有当数值本身是唯一主张时才归入 value。

### 4.4 高频冲突场景速查表

| 冲突对 | 示例 A | → 判定 | 示例 B | → 判定 | 判定依据 |
|---|---|---|---|---|---|
| rank ↔ proportion | "Nintendo leads with 20% share" | `rank` | "Market: A 30%, B 25%, C 20%" | `proportion` | 谓语 "leads" 是位置性的；份额分解无位置谓语 |
| rank ↔ difference | "A ranks #1, followed by B" | `rank` | "A is 2x higher than B" | `difference` | "ranks" 是位置性的；"higher than" 强调差距 |
| rank ↔ extreme | "Nintendo ranks #1 among all publishers" | `rank` | "Sales peaked at $5M in 2019" | `extreme` | 前者是多实体排序；后者标识单个边界点 |
| extreme ↔ outlier | "2019 had the highest sales at $5M" | `extreme` | "2019 sales were anomalously high, 3x above trend" | `outlier` | 前者仅标识边界值；后者强调偏离模式 |
| extreme ↔ value | "The maximum temperature is 42°C" | `extreme` | "The average temperature is 25°C" | `value` | 前者有极值修饰（max）；后者是普通统计量 |
| trend ↔ difference | "Sales grew from $3M to $5M over 5 years" | `trend` | "2023 sales are 2x higher than 2020" | `difference` | 前者有时间方向性；后者是两点静态对比 |
| distribution ↔ outlier | "Heavy-tailed distribution" | `distribution` | "3 products have sales >10x median" | `outlier` | 前者描述整体形态；后者指向具体异常点 |
| association ↔ difference | "Higher price correlates with higher rating" | `association` | "Premium products have 2x higher ratings" | `difference` | 前者是连续协变；后者是离散组间对比 |
| value ↔ rank | "Average sales is $45K" | `value` | "A has the highest average at $45K" | `rank` | 前者无位置关系；后者 "highest" 引入位置 |

---

## 5. 与 v2 的变更对照

### 5.1 命名变更

v2 使用复合命名（如 `rank_dominance`），每个名字包含两个语义，导致类别间天然交叉。v3 回归单一原子命名，每个 type ID 只对应一个语义。

| v2 Type ID | v3 Type ID | 变更说明 |
|---|---|---|
| `value_aggregation` | `value` | 去掉 aggregation 后缀（已隐含在 value 定义中） |
| `proportion_composition` | `proportion` | 去掉 composition 后缀（composition 是 proportion 的自然推论） |
| `rank_dominance` | `rank` | 去掉 dominance（dominance 是 rank + proportion 的复合概念，正是 v2 中 rank↔proportion 冲突的根源） |
| `difference_comparison` | `difference` | 去掉 comparison 后缀（comparison 是 difference 的同义重复） |
| `distribution_concentration` | `distribution` | 去掉 concentration（concentration 既可指统计集中度也可指市场集中度，引入歧义） |
| `outlier_extreme` | `outlier` + `extreme` | 拆分为两个独立类型，恢复 DataShot 原始区分 |
| `segmentation_similarity` | `cluster` | 简化为 VIS 社区更通用的术语 |
| `categorical_profile` | (移除) | 非原子类型，吸收入 value/rank/proportion/distribution |
| `other` | (移除) | 代码中已显式过滤；无法分类的 insight 应在 summarizer 层面拒绝 |
| `data_quality` | `data_quality` | 保留不变 |

### 5.2 结构变更

| 维度 | v2 | v3 |
|---|---|---|
| 类型总数 | 11 + other | 11 |
| 命名风格 | 复合词 | 单一原子词 |
| 消歧机制 | 仅优先级列表 | 两步协议（主谓语判定 + 优先级裁决） |
| 文献对齐 | 部分对齐，有自造复合概念 | 严格对齐 DataShot 原子类型 |
| Categorization | 独立类型 `categorical_profile` | 吸收（非原子） |
| other | 保留为兜底 | 移除（summarizer 层面处理） |

---

## 6. 在 Coverage Map 中的应用

### 6.1 Coverage Grid 的轴编码

Coverage Map 采用二维网格布局：

```
         col_A    col_B    col_C    col_A×col_B  ...
value      ●                ●●
proportion ●●       ●
rank       ●●●      ●●      ●
difference          ●
trend      ●        ●●●             ●●
distrib.                    ●●
association                          ●●●
outlier    ●                ●
extreme             ●
cluster
data_qual. ●        ●
```

- **X 轴**：数据列或列组合（来自 atomic insight 的 `columns` 字段）
- **Y 轴**：本 taxonomy 的 11 个类型
- **单元格**：该列×该类型下的 atomic insight 数量（颜色深浅编码密度）
- **空白格**：揭示"尚未被探索的分析维度"，是 coverage map 的核心价值

### 6.2 Taxonomy 对 Coverage Map 的设计约束

1. **类型数量**：11 类在 Y 轴上可以一屏展示，不需要滚动。
2. **频率均衡性**：基于 v2 校准数据的预估频率分布（去除 categorical_profile 和 other 后重新分配）：

```
rank:         ~19%    ████████████████████
trend:        ~18%    ██████████████████
association:  ~18%    ██████████████████
outlier:      ~10%    ██████████
extreme:       ~3%    ███
proportion:   ~12%    ████████████
data_quality: ~11%    ███████████
distribution:  ~5%    █████
value:         ~3%    ███
difference:    ~1%    █
cluster:       ~0%
```

3. **低频类型的价值**：`difference`、`value`、`extreme`、`cluster` 频率低不是 taxonomy 的问题，而是当前 agent 探索策略的偏好。在 coverage grid 中，这些行的空白恰好揭示了 agent 尚未充分探索的分析维度——这正是 coverage map 要传达的信息。

---

## 7. 在 Summarizer Prompt 中的应用

### 7.1 Taxonomy 定义注入

Summarizer 的 system prompt 中需要注入每个类型的定义和提示词，用于引导 LLM 进行分类：

| Type ID | Prompt Hint（注入 LLM 的简短描述） |
|---|---|
| `value` | a single aggregate statistic, count, or computed quantity |
| `proportion` | a part-whole share, percentage, or compositional breakdown |
| `rank` | an ordering or positional relationship among entities |
| `difference` | a contrast or gap between specific groups or conditions |
| `trend` | a directional change over time or ordered dimension |
| `distribution` | the shape, spread, or concentration of a set of values |
| `association` | a co-variation or dependency between variables |
| `outlier` | an anomalous observation that deviates significantly from the overall pattern |
| `extreme` | a maximum, minimum, peak, or trough value at the boundary of the data range |
| `cluster` | a natural grouping or similarity pattern in the data |
| `data_quality` | a data completeness, consistency, or validity issue |

### 7.2 分类指令

Summarizer prompt 中的分类指令应包含：

1. 分类依据是主句的谓语语义，不是统计方法或变量类型。
2. 如果一条 insight 无法稳定归入任何类型，不要输出该 atomic insight（而非给一个 fallback 标签）。
3. 每条 atomic insight 必须恰好一个 type。

---

## 8. 论文写作参考

### 8.1 建议的论文表述

> We adopt an 11-type insight taxonomy for classifying atomic findings produced by the EDA agent. The taxonomy takes DataShot's data fact classification [ref] as its backbone — the most widely adopted fact-level taxonomy in data storytelling research. We directly retain 8 of DataShot's 11 atomic types with their original semantics (*proportion*, *rank*, *difference*, *trend*, *distribution*, *association*, *outlier*, *extreme*), merge 1 pair with overlapping semantics in the EDA context (*Value* + *Aggregation* → *value*, since automated EDA produces only computed quantities), and absorb 1 non-atomic type (*Categorization*, which decomposes into other types depending on the predicate relationship). We extend the taxonomy with *cluster*, supported by CoInsight's *Similarity* pattern [ref] and Amar & Stasko's analytic task taxonomy [ref], and *data\_quality*, an empirically motivated addition that accounts for 11.3% of insights in our system's output. To ensure single-label operationalizability, we define a two-step disambiguation protocol: first classify by the predicate relationship in the insight's main clause, then apply a fixed priority order for tie-breaking.

### 8.2 可操作性验证建议

论文中最有说服力的 taxonomy 验证是 inter-annotator agreement 实验：

1. 从系统产出中抽样 50–100 条 atomic insight。
2. 由 2–3 位标注者独立使用本消歧协议进行单标签标注。
3. 报告 Cohen's Kappa（2 人）或 Fleiss' Kappa（3 人），目标 κ > 0.7。
4. 对分歧案例进行定性分析，说明主要分歧来源（通常集中在 rank↔proportion 和 trend↔difference）。

### 8.3 与 Related Work 的对接点

| 论文方向 | 本 taxonomy 的对接角色 |
|---|---|
| Data Storytelling (DataShot, Calliope, DataTales) | 本 taxonomy 直接继承 DataShot 的 fact taxonomy，可作为自动化 EDA 场景下的 operationalization |
| Insight Management (InsightLens, CoInsight) | 本 taxonomy 为 insight 提供结构化分类，支撑 coverage map 的轴编码和 insight 检索/筛选 |
| Visual Analytics Task Taxonomy (Amar & Stasko, Brehmer & Munzner) | `cluster` 类型的引入对齐了 low-level analytic task 中的 Cluster 任务 |
| Automated EDA (Lux, Voyager, SeeDB) | 本 taxonomy 可用于评估自动化系统的探索覆盖度（哪些分析维度被覆盖/遗漏） |

---

## 9. 代码实现对照

### 9.1 需要同步修改的文件

| 文件 | 修改内容 |
|---|---|
| `backend/config.py` | `INSIGHT_TAXONOMY_TYPES`、`INSIGHT_TAXONOMY_DISPLAY_NAMES`、`INSIGHT_TAXONOMY_PROMPT_HINTS` |
| `backend/framework/models.py` | `InsightType` Literal 类型定义 |
| `backend/framework/summarizer.py` | Taxonomy 注入逻辑（已自动从 config 读取，无需手动改） |
| `frontend/src/types/index.ts` | `InsightType` 类型定义、`INSIGHT_TAXONOMY_TYPES` 数组 |
| `frontend/src/config.ts` | `INSIGHT_TAXONOMY_V1` 数组、`INSIGHT_TAXONOMY_COLORS` 颜色映射 |

### 9.2 v3 Type ID 列表（代码用）

```python
INSIGHT_TAXONOMY_TYPES = [
    "value",
    "proportion",
    "rank",
    "difference",
    "trend",
    "distribution",
    "association",
    "outlier",
    "extreme",
    "cluster",
    "data_quality",
]
```

---

## 10. 参考文献

1. **DataShot** — Wang et al., "DataShot: Automatic Generation of Fact Sheets from Tabular Data," IEEE TVCG, 2020.
   MSR 技术报告 PDF: `https://www.microsoft.com/en-us/research/wp-content/uploads/2024/03/datashot-supp-final.pdf`

2. **CoInsight** — Li et al., "CoInsight: Visual Storytelling for Hierarchical Tables With Connected Insights," IEEE TVCG, 2024.
   DOI: `https://doi.org/10.1109/TVCG.2024.3388553`

3. **arXiv 2404.01644** — EDA insight 分类实践（参考 DataShot + Snowy）。
   PDF: `https://arxiv.org/pdf/2404.01644`

4. **Amar & Stasko** — "Low-Level Components of Analytic Activity in Information Visualization," IEEE InfoVis, 2005.
   （`Cluster` 作为 low-level analytic task 的学术支撑）

5. **Brehmer & Munzner** — "A Multi-Level Typology of Abstract Visualization Tasks," IEEE TVCG, 2013.
   （可视化任务分类的层级框架参考）
