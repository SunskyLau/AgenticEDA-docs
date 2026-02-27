# 洞察分类体系（报告优先 / Report-first）

本文档说明：如何以符合当前系统输出形式（**证据优先**、**简短的报告式要点**）的方式，对我们生成的 Insight（洞察）进行标注/分类；而不是强行给每条 Insight 套一个单一的细粒度标签。

---

## 1) 在我们系统里，“Insight”是什么（当前版本）

在 `state.json` 中，一个 Insight 节点本质上就是一条简短的报告要点（bullet）：
- `insight_plain` / `insight_with_evidence` 中的主张（claims）
- 引用 artifacts（plots/stdout/stderr）的行内证据标记
- 一棵树结构（`parent_insight_id`, `children_insight_ids`），它天然就像“报告章节 -> 子章节”的结构

这意味着：一条 Insight 往往包含多个“微事实”（例如：排名 + 占比 + 异常值说明），因此“每条 Insight 只能有一个细粒度标签”的分类法经常不匹配。

示例（最新一次 run）：
- `backend/runs/run_20260204_110756_10c565/state.json`

---

## 2) 为什么“单一细粒度标签”经常失效

我们当前的 Insights 常常把下面这些内容组合在一起：
- 同一段落里包含多种事实类型（例如：“A 以 X 销量领先（汇总）并占 Y%（占比），其他为 Z%（占比）”）。
- 有一个小叙事弧线（对比 -> 解释 -> 限制/异常点），而不是单一的原子化陈述。
- 证据落地约束（必须引用 artifacts），这会鼓励把同一张图相关的主张打包在一起写。

所以更合适的问题通常不是“这条 Insight 是哪 **一种** 类型？”，而是：
- “这条 Insight 包含了哪 **一组** 事实类型？”
- “这条 Insight 在报告树里扮演什么角色（根部总览 vs 向下钻取的细节）？”

---

## 3) 推荐做法：两层标注

### Level A - 覆盖/主题（粗粒度、对 UI 友好）

我们已经有一套用于 Coverage Grid Map 的启发式覆盖分类：
- `frontend/src/config.ts` -> `COVERAGE_TAXONOMY_V0`

把它当作“覆盖可视化”的粗粒度主题（适合回答“我们探索了哪些领域？”），而不是 Insight 唯一的语义真相。

### Level B - 数据事实类型（细粒度、可组合、可多标签）

用一套“数据事实（data-facts）”分类，去标注一条 Insight 内部包含的微事实。常见类别包括：
- 分布（偏态、长尾、分位数、离散程度）
- 趋势（随时间/按年份变化）
- 排名（Top-K、领先者、排序）
- 占比（份额、百分比、比率）
- 差异/比较（A vs B）
- 关联（相关性/关系）
- 极端/异常（罕见/极端值）
- （可选）汇总（总量、均值、分组聚合）

重要：同一条 Insight 可以同时拥有多个事实类型标签。

---

## 4) 具体映射指南（MVP）

### 保持 Insight 节点“报告化”

不要强迫每条 Insight 变成一个原子事实。相反：
- 保持段落形式（主张 + 证据 + 推理）。
- 为段落内包含的内容添加“多标签”的事实类型标注。

### 建议的最小标签集（先从小做起）

MVP 阶段建议从 6–8 个标签开始：
- `distribution`, `trend`, `rank`, `proportion`, `difference`, `association`, `outlier`, `aggregation`

### 示例映射（来自 run_20260204_110756_10c565）

- `Global_Sales` 的重尾分布 -> `distribution`, `outlier`
- “Nintendo 以 20% 市场份额占主导……” -> `rank`, `aggregation`, `proportion`
- “集中度随时间增加（Gini 上升）” -> `trend`, `aggregation`
- “集中度与年度发行量相关” -> `association`, `trend`

---

## 5) 建议的数据契约扩展（可选、向后兼容）

如果我们决定把标签持久化（而不是在 UI 里每次重分类）：
- 在每条 Insight 上新增可选字段 `fact_types: string[]`（多标签）。
- （可选，后续）把段落拆成 `claims: { text, fact_types, evidence_refs }[]`。

向后兼容规则：
- 如果缺少 `fact_types`，UI 可回退使用 `COVERAGE_TAXONOMY_V0` 的启发式分类。

---

## 6) 可参考的相关工作（用于搜索/对齐概念）

这个讨论更接近“用于报告/叙事的数据事实”，而不是“单标签的 EDA 洞察类型”。

有用的参考（按名字搜索）：
- DataShot：面向报告的数据事实分类（fact types 可组合）
- AutoClips：用紧凑的数据事实类别集生成“片段式”EDA 总结
- Foresight：引导式/洞察模式（适合候选生成，但往往过于原子化，不适合报告要点）
- Amar & Stasko task taxonomy：分析任务分类（更适合 UI/任务建模，而不是给报告文本打标签）
