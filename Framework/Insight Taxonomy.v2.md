# 洞察分类体系（v2：Predominant Type）

本版目标：给每条 Insight 分配**唯一主类型（predominant type）**，并保留学术可追溯性，支撑 Coverage Map 的 taxonomy 列。

旧版文档已保留为：`docs/Framework/Insight Taxonomy.v1.md`。

---

## 1) 证据基础（Academic + Repo）

### 1.1 文献依据

1. `2404.01644`（arXiv）明确写到：洞察类别参考 **DataShot**，统计量-类别映射参考 Snowy。文中使用的类别包含 `difference / value / trend / outlier / distribution / rank / correlation`。  
2. DataShot 给出可组合的 fact taxonomy（11 类）：`Value / Proportion / Difference / Distribution / Trend / Rank / Categorization / Aggregation / Association / Outlier / Extreme`。  
3. CoInsight（TVCG 2024 preprint）给出宏观分组：`Point / Shape / Compound`，并细化到常见类型（如 `Dominance / Top2 / Outlier / Trend / Skewness / Correlation / ChangePoint / Seasonality / Similarity`）。
4. 本项目在产线标签中不保留 `compound`，统一使用 `other` 作为兜底类型。

### 1.2 本仓库 run 观测

从 `backend/runs/*/state.json` 提取 `insight_plain`，过滤流程性文本后，共得到 **195** 条有效 insight（2026-02-06 抽样）。  
高频语义集中在：`Rank/Dominance`、`Trend`、`Association`、`Outlier/Extreme`、`Proportion/Composition`、`DataQuality`。

---

## 2) Predominant Taxonomy（v2）

下表是项目采用的主类型集合。前 10 类直接对齐 DataShot / CoInsight；`data_quality` 为项目扩展（因为当前 run 中该类占比高且影响结论可信度）。

| Type ID | 定义（主判断语义） | 典型触发词 | 学术锚点 |
|---|---|---|---|
| `association` | 变量间关系强弱/方向 | correlation, relationship, associated | DataShot `Association`; CoInsight `Correlation` |
| `trend` | 随时间/序列的变化方向 | trend, increase, decline, over years | DataShot `Trend`; CoInsight `Trend` |
| `outlier_extreme` | 异常点/极端样本驱动的结论 | outlier, anomaly, extreme, rare | DataShot `Outlier` + `Extreme`; CoInsight `Outlier` |
| `distribution_concentration` | 分布形态与集中度 | skewness, kurtosis, gini, lorenz, heavy-tail | DataShot `Distribution`; CoInsight `Skewness/Kurtosis/Evenness` |
| `proportion_composition` | 份额/占比/构成 | share, percentage, proportion, account for | DataShot `Proportion`; CoInsight `Dominance/Top2/Evenness` |
| `rank_dominance` | 排序、头部、领先关系 | top-k, highest, leading, dominant | DataShot `Rank`; CoInsight `Dominance/Top2` |
| `difference_comparison` | 组间差异/对照 | compared, vs, gap, higher/lower than | DataShot `Difference` |
| `value_aggregation` | 单组统计量或聚合量 | total, mean, median, sum, throughput | DataShot `Value` + `Aggregation` |
| `categorical_profile` | 类别结构描述（非排序主导） | category mix, segment profile | DataShot `Categorization` |
| `segmentation_similarity` | 聚类/相似群体结构 | cluster, segment, similar pattern | DataShot（cluster family）；CoInsight `Similarity` |
| `data_quality` | 缺失/一致性/可用性问题主导结论 | missing, null, completeness, data quality | 项目扩展（run 高频） |
| `other` | 无法稳定归入已定义类型的剩余情况 | 多主句并列且主语义不稳定，或语义缺失 | 项目兜底分类 |

> 说明：`other` 仅在无法稳定判定单主语义时启用，默认尽量归入前 11 类之一。

---

## 3) 单一主类型判定顺序（Deterministic）

为保证“每条 insight 只有一个主类型”，按以下顺序首个命中即返回：

1. `data_quality`  
2. `association`  
3. `trend`  
4. `outlier_extreme`  
5. `distribution_concentration`  
6. `proportion_composition`  
7. `rank_dominance`  
8. `difference_comparison`  
9. `value_aggregation`  
10. `categorical_profile`  
11. `segmentation_similarity`  
12. `other`（兜底，仅当无法稳定归入前 11 类）

此外，以下文本先行过滤为 `meta_non_insight`（不进入 taxonomy）：  
`Analysis failed...`、`...saved as ...`、纯标题行（如 `=== ... ===`）。

---

## 4) 与 Coverage Map 的对接

当前 `frontend/src/config.ts` 中 `COVERAGE_TAXONOMY_V0` 已覆盖：
- `data_quality`、`outliers`、`trend_time`、`correlation`、`group_compare`、`segmentation`、`distribution`

为与本 v2 对齐，建议在后续 `COVERAGE_TAXONOMY_V1` 新增：
- `rank_dominance`
- `proportion_composition`
- `value_aggregation`
- `categorical_profile`
- `other`

在 UI 过渡期，可先用如下回退：
- v2 中未被 V0 覆盖的类型（含 `other`）-> `unknown`

---

## 5) 当前 runs 的校准快照（195 条）

- `rank_dominance`: 36 (18.5%)
- `trend`: 35 (17.9%)
- `association`: 35 (17.9%)
- `outlier_extreme`: 25 (12.8%)
- `proportion_composition`: 23 (11.8%)
- `data_quality`: 22 (11.3%)
- `distribution_concentration`: 9 (4.6%)
- `value_aggregation`: 4 (2.1%)
- `categorical_profile`: 4 (2.1%)
- `difference_comparison`: 2 (1.0%)

该分布说明：如果 taxonomy 不包含 `rank/proportion/data_quality`，Coverage Map 会系统性丢失高频洞察类型。

---

## 6) 参考文献（可追溯链接）

1. arXiv 2404.01644 PDF: `https://arxiv.org/pdf/2404.01644`  
2. DataShot（MSR 技术报告 PDF）: `https://www.microsoft.com/en-us/research/wp-content/uploads/2024/03/datashot-supp-final.pdf`  
3. CoInsight（Insight taxonomy: Point/Shape/Compound）: `https://www.preprints.org/manuscript/202405.1985/v1/download`
