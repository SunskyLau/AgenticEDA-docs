# Insight Report 生成交互设计

## 1. 交互入口

在 Graph 图中，用户可以与任意 **Insight 节点** 进行 `Report` 总结交互。

---

## 2. 核心目标

基于当前 Insight 的完整 **drill-down chain**，生成一份具备**严密推理逻辑**的 Analysis Report。

> **Report 的时间范围**：从 drill-down 起点 → 直到当前选中的 Insight 节点（包含其全部路径）

---

## 3. Report 结构设计

### 3.1 因果链说明

Report 必须清晰阐述 drill-down 的**因果逻辑**，每个 Plan 的 `motivation` 需要发挥作用：

```
Drill-down 路径示例：
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  Plan   │ ──▶ │ Insight │ ──▶ │  Plan   │ ──▶ │ Insight │
│    A    │     │    A    │     │    B    │     │    B    │
│(动机A)  │     │(发现A)  │     │(动机B)  │     │(发现B)  │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
```

### 3.2 逻辑推理结构

每个 Insight 中的**论点**需包含：

| 层级 | 内容 | 说明 |
|------|------|------|
| **论点** | Claim | 该 Insight 想要传达的核心发现 |
| **证据** | Evidence [N] | 支撑论点的可视化/数据证据（引用标注） |
| **推理** | Reasoning | **关键**：解释如何从证据得出论点 |

---

## 4. 证据引用与推理示例

```
论点：北京地区销售额显著高于其他地区

推理过程：
- 从 [1] 中，可以看到北京地区的柱子最高，说明其销售额在所有城市中排名第一
- 从 [2] 中可以具体看到北京地区的数值为 ¥1,234,567，远超第二名上海的 ¥876,543
- 结合 [1] 的视觉分布和 [2] 的精确数值，可推断北京是核心销售区域
```

> **注**：[1]、[2] 对应可视化图表中的具体视觉元素或数据标注

## 生成过程
构建一个reporter Agent，同样使用gemini模型
给他所有chain上的plan和insight节点。对于plan，其plan内容和相应的motivation，完整的分析流程(reflection/code/plots/stdout)都要有；对于insight节点，其evidence版本的文本和evidences都应该要让reporter看到。输出的report应该按照顺序每个insight一段，并且要讲明为什么要探索每个insight，也就是要讲清楚motivation，形成逻辑严密、可读性高、完全真实、引人入胜的analysis report。当然，每段内对于insight的分析也不可或缺，需要向用户细致地解释如何得到insight的，尤其需要从evidence入手进行解释

---

## 当前实现（MVP）

### 前端触发
- Run Gateway：`POST /api/runs/:runId/report`
- Body：`{ insight_id, force?: boolean, language?: "en" | "zh" }`

### 产物与事件
- 报告正文：`artifacts/reports/report_{insight_id}.md`
- 报告输入快照：`artifacts/report_packs/{insight_id}/...`
- 事件：`report_generated`（写入 `events.jsonl`，供 Timeline/Inspector 实时更新）
