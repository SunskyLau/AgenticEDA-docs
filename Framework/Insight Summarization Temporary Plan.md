# 临时方案设计：Insight Summarization 与 Predominant Type

状态：`Temporary Draft`  
适用范围：当前 Agentic-EDA 产线（不改变“每个 plan 节点仅产出一条 insight”的约束）

---

## 1. 本文档目的

在不改变当前 insight 生成风格与节点结构的前提下，给出一个可执行的临时设计方案，实现：

1. `brief_report`：结合完整 plan 链路与所有 insight 的逻辑化短报告。  
2. `insight_cards`：逐条 insight 的结构化摘要，每条唯一对应一个 `predominant_type`。  
3. `evidence`：每条 insight 的证据可追溯、可审计、与主类型对齐。

---

## 2. 用户意图推断（显性 + 隐性）

### 2.1 显性意图

1. 保持当前 insight 文本风格、长度和叙述方式，不希望“AI 改写成另一种口吻”。  
2. 每个 insight 只能有一个主类型（`predominant_type`），不能拆成多条 insight。  
3. 需要一个完整但简短的 `brief_report`，并且该报告要基于整个 analysis plan 链路。  
4. 分类体系要有学术支撑，不是随意命名。  

### 2.2 隐性意图

1. Coverage Map 的 taxonomy 列要稳定、可比较、可复现。  
2. 总结文本与结构化类型要双向可追溯（report 可追到 insight，insight 可追到 evidence）。  
3. 需要先有“可上线 MVP”，再逐步强化自动化与一致性。

---

## 3. 设计原则

1. **不拆 insight**：保留“一个 plan 节点 -> 一个 insight 节点”。  
2. **单标签强约束**：每条 insight 仅一个 `predominant_type`。  
3. **证据优先**：主类型必须能被 primary evidence 直接支撑。  
4. **风格一致**：保持现有语言模式（1-2句为主，数据+对比+转折）。  
5. **低侵入改造**：先增量扩展 summarizer 输出，不重写 planner/analyzer。

---

## 4. 输出契约（建议 vNext）

Summarizer 输出一个对象，包含两层内容：

```json
{
  "brief_report": "string",
  "insight_cards": [
    {
      "insight_id": "string",
      "plan_id": "string",
      "parent_insight_id": "string|null",
      "insight_plain": "string",
      "insight_with_evidence": "string",
      "predominant_type": "enum",
      "type_confidence": 0.0,
      "secondary_types": ["optional", "for debug only"],
      "evidence": [
        {
          "role": "primary|supporting|limitation",
          "source_path": "artifacts/...",
          "locator": "line/metric/plot-element",
          "supports_claim": "string"
        }
      ]
    }
  ]
}
```

说明：
- `secondary_types` 仅用于调试和评估，不进入 coverage 主列。  
- coverage 仅消费 `predominant_type`。  

---

## 5. Predominant Type 判定方案

### 5.1 类型集合

沿用 `docs/Framework/Insight Taxonomy.v3.md` 的集合。若无法稳定归类，不输出该 atomic insight（`other` 仅可作为内部 fallback 标记用于调试，不进入 coverage，也不对外暴露）。

### 5.2 判定流程（不拆文本）

1. 识别主断言句（默认首句权重最高；`while/however` 后句通常为补充或限定）。  
2. 按关键词、统计量类型、证据指向做打分。  
3. 若多类型接近，按固定优先级裁决（taxonomy v3 中 deterministic 顺序，见 v3 文档）。  
4. 仍无法稳定归类时，不输出该 atomic insight（或内部标记为 `other` 仅用于调试）。  

### 5.3 关键约束

1. 每条输出的 insight 必须输出 `predominant_type`。  
2. `predominant_type` 必须有至少一条 `primary` evidence 支撑。  
3. 没有证据支撑的类型降置信度，必要时不输出该 insight（`other` 仅用于调试）。

---

## 6. Evidence 设计（回答“证据怎么更好”）

每条 insight 的证据采用“主证据 + 辅证据”结构：

1. **Primary Evidence（必需，>=1）**  
   - 必须直接支撑主断言和 `predominant_type`。  
   - 示例：相关系数、Gini 变化值、Top-K 占比、异常阈值结果。  

2. **Supporting Evidence（可选，0-2）**  
   - 解释背景、补充对比、提高可读性。  

3. **Limitation Evidence（可选）**  
   - 指出数据缺失、样本偏差、统计不稳定区间。  

证据字段必须可定位：`source_path + locator`，避免“只说有图但无法核对”。

---

## 7. brief_report 生成策略

`brief_report` 不是 insight 拼接，而是“计划驱动的逻辑文本”：

1. 依据 plan 链路组织叙事：`motivation -> analysis -> finding -> implication`。  
2. 每段落绑定若干 `supporting_insight_ids`（内部字段，可不对用户展示）。  
3. 若跨 insight 汇总结论，至少引用两条 insights 的 primary evidence。  
4. 保留简短体量，但必须交代探索动机与因果推进。  

---

## 8. 风格一致性约束（保持当前 runs 风格）

基于当前 runs 观测（195 条有效 insight）：

1. `insight_plain` 目标为 1-2 句主导，避免过度模板化。  
2. 参考长度带：约 180-400 字符（极端值允许但应审查）。  
3. 保持“定量 + 转折 + 解释”的写法，不强制学术论文腔。  
4. 不因分类而重写语气；分类是标签层，不是改写层。  

---

## 9. 分阶段落地（临时版本）

### Phase A：契约先行（1-2 天）

1. 固化输出 schema（`brief_report` + `insight_cards`）。  
2. 接入 `predominant_type` 与 `type_confidence` 字段。  

### Phase B：证据对齐（2-3 天）

1. 强制每条 insight 至少 1 条 primary evidence。  
2. 给 evidence 增加 `locator`。  

### Phase C：质量门禁（1-2 天）

1. 校验“100% insight 有主类型”。  
2. 校验“主类型均有 primary evidence”。  
3. `other` 比例异常升高时触发告警（提示规则失效或输出漂移）。  

---

## 10. 验收标准（MVP）

1. 每条 insight 都有且仅有一个 `predominant_type`。  
2. 每条 insight 至少一个 `primary` evidence。  
3. `brief_report` 明确覆盖 plan 动机链路。  
4. 文本风格与当前 runs 基本一致（人工抽样可读性通过）。  
5. `other` 作为兜底，不成为主流类别。

---

## 11. 暂不纳入（Out of Scope）

1. 不改动 planner 的计划生成逻辑。  
2. 不做多标签覆盖图（当前仅单标签主列）。  
3. 不在本阶段引入新的复杂分类模型训练流程。
