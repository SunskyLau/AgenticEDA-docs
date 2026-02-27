# Agentic EDA Framework 深度调研：贡献空间与“为什么还需要人 + 可视化交互”

> 目的：为本项目提供可引用的学术背书（related work + 理论/实证依据），回答两类核心质疑：  
> 1) “这种长链持续运行的 Agentic EDA Framework 还有什么贡献/价值？”  
> 2) “既然是 Agentic Framework，为什么还需要人为干预、可视化与交互？是否真的有需求？”

---

## 0) 我们的系统定位（用于对比，而非 related work）

本项目的目标不是“一次性回答一个问题”，而是一个**可长链持续运行**、可被停止/恢复、可追溯的 Agentic EDA Framework：
- 以 dataset 为驱动自动提出分析计划（plan），执行分析代码，产出证据（plots/stdout/stderr），并生成**带证据引用**的 insight；
- 维护 plan/insight 的树/图结构与运行工件（events/state/artifacts），支持回放、检查、对比与可视化导航（Tree/Graph/Map/Coverage Grid）。

下文要证明两点：  
1) “把 EDA 做成**可持续运行、可追溯、可交互**的 agentic framework”是仍有缺口的研究方向；  
2) 人类干预与交互式可视化不是“多余 UI”，而是可靠性治理与 sensemaking 的必要组成。

---

## 1) 是否存在与我们“完全一样”的研究工作？

### 1.1 结论（严格意义的“完全一样”很难成立）
在截至 2026-02-05 的检索范围内，我没有发现一篇论文/系统在同一工作中同时满足以下组合：
1) **长链、可恢复的 agentic 执行**（multi-step tool use + 失败恢复/重试 + 状态持久化）；  
2) **dataset 驱动的持续探索**（自动扩展 frontier，而非主要由用户问答串联）；  
3) **insight 作为一等对象的结构化管理**（plan/insight 图谱 + drill-down 链）；  
4) **强制 evidence grounding**（每条 insight 显式绑定可追溯工件：图/日志/输出/代码快照）；  
5) **面向监控/覆盖/导航/调试/接管的可视化交互 UI**（不仅是 chat log）。

但“高度相似的局部组合”是存在的（见第 2 节）：
- **Agentic data science / coding agents** 强在长链执行与自动改进，但通常不以“持续 EDA 探索 + insight 图谱 + 覆盖导航 UI”为主轴（如 Data Interpreter、TaskWeaver、DatawiseAgent、AutoKaggle、AIDE、Jupiter 等）[R1–R6]。  
- **LLM + EDA/BI 系统** 强在 GUI、NL2SQL、可视化与任务闭环，但多以“用户问题驱动/交互问答”为主，而非“无问题输入的持续探索”[R7]。  
- **insight 管理/导航系统** 强在 insight 组织、上下文与证据绑定，但未必包含长链持续自动探索的 engine，或不强调 event-sourcing 式运行回放[ R8 ]。

因此更稳健的表述是：**相关工作覆盖若干关键子模块，但缺少把这些子模块“作为一个可持续运行、可追溯、可交互的 EDA framework”整合并系统评估的研究。**

### 1.2 粗粒度对照矩阵（用于写 related work gap）
下表不追求“绝对二元真值”，而是标注“论文/系统是否把该维度作为核心问题并提供系统性方案”。

| 系统/方向 | 长链执行/恢复 | dataset 驱动持续探索 | insight 对象化 | 强 evidence grounding | 覆盖/导航式可视化 UI |
|---|---:|---:|---:|---:|---:|
| Data Interpreter [R1] | ✓ | ?（更偏任务驱动） | 部分 | 部分 | - |
| TaskWeaver [R2] | ✓ | ? | - | 部分 | - |
| DatawiseAgent [R3] | ✓ | ? | - | 部分 | - |
| AutoKaggle [R4] | ✓ | -（更偏竞赛任务） | - | 部分 | - |
| Jupiter [R6] | ✓ | ?（搜索轨迹） | - | 部分 | - |
| TiInsight [R7] | 部分 | -（偏 query 驱动） | 部分 | 部分 | ✓ |
| InsightLens [R8] | - | - | ✓ | ✓（强调证据/上下文） | ✓ |
| Data Formulator [R10] | - | - | - | - | ✓（强调交互迭代） |
| Lux / Voyager / SeeDB [R30–R32] | - | - | - | - | ✓（覆盖与推荐） |

我们可以把研究贡献定位为：**填补“长链 agentic 执行”与“insight/证据/覆盖的可视化治理”之间的空白。**

---

## 2) 相关工作版图（建议写论文时按 3+1 块组织）

### 2.1 长链 Agentic Framework（执行与失败恢复强，但 EDA/Insight 治理未必完整）
- **TaskWeaver (2023)**：面向数据分析任务的 code-first agent 框架，强调通过代码与插件完成复杂任务，并讨论人类干预/交互依赖（interactive dependencies）[R2]。  
- **DatawiseAgent (2025)**：notebook-centric 的长程数据分析 agent，强调贴近真实数据科学开发流程与迭代式构建/调试[ R3 ]。  
- **AutoKaggle (2024)**：面向竞赛/建模任务的多智能体协作，强调 end-to-end pipeline 与 human-in-the-loop 机制[ R4 ]。  
- **AIDE (2025)**：把 ML 工程视为代码空间的搜索/迭代优化，更偏“达成指标”而非 EDA sensemaking[ R5 ]。  
- **Jupiter (2025)**：把数据分析建模为可执行轨迹的搜索（MCTS），提供可量化评测路径[ R6 ]。  

补充：ReAct 作为通用 tool-using agent 范式基线[ R9 ]。

### 2.2 LLM + EDA/BI/可视化系统（交互与可视化强，但多为问题驱动）
- **TiInsight (2024/2025)**：面向“从 query 到 insight”的跨域 EDA 系统，包含 text-to-SQL、可视化与 GUI 交互闭环[ R7 ]。  
- **Data Formulator (VIS 2023)**：AI 辅助的数据变换 + 可视化生成与交互式迭代，强调“人通过交互逐步收敛意图”，与真实数据科学工作流紧密相关[ R10 ]。

### 2.3 Insight 作为对象的管理与导航（为“为什么需要 UI”提供直接背书）
- **InsightLens (2024)**：针对 chat-based LLM 数据分析中“上下文与 insight 纠缠”的问题，提出 insight 抽取、证据绑定、主题组织与 Minimap/Topic Canvas 等交互视图，明确把 insight 当成可管理对象[ R8 ]。

### 2.4 经典可视化/视觉分析与数据工作流研究（为“为什么需要人+交互”提供理论与实证）
这部分是“交互式可视化为什么不是可选项”的核心背书。经典观点是：**自动化分析**与**交互式可视化**是互补关系，尤其在开放式探索（EDA）中，交互承担了“外部化认知/假设检验/错误发现/结果沟通/责任追溯”等功能。

**(a) EDA / Sensemaking：探索本质上是循环推理**  
- Tukey 对 EDA 的经典论述强调探索性、迭代性与启发式发现（而非一次性“得出答案”）[R11]。  
- Sensemaking 理论将分析过程建模为“信息觅食（foraging）-> 结构化/假设化（sensemaking）-> 再觅食”的循环；工具价值在于降低循环成本并提供 leverage points[ R12, R13 ]。  

**(b) Visual Analytics：交互式可视化是分析推理的一部分**  
- Visual Analytics 的纲领性定义强调“由交互式可视界面所促进的分析推理”[R14]。  
- Shneiderman 的“overview first, zoom and filter, then details-on-demand”指出交互是探索的基本动作集合（并强调 history/extract 等机制）[R15]。  
- 交互在可视化中的作用被系统化分析：交互不仅是操作 UI，而是支撑比较、聚焦、重构与验证等认知活动[ R16, R17 ]。  

**(c) 自动化风险与信任校准：越自动，越需要可见性与可控性**  
- “Ironies of Automation / Out-of-the-loop”指出：自动化会把人推向监控与异常处理角色，但监控更难、恢复更慢、易失去态势感知[ R18, R19 ]。  
- 人机信任研究强调“适度依赖（appropriate reliance）”与信任校准：系统需要让用户理解何时可靠、何时不可靠，并提供纠错与验证支持[ R20, R21 ]。  

**(d) 数据分析真实工作流：强实证背书**  
- 企业数据分析访谈研究表明：分析工作高度迭代、跨工具、受沟通与组织约束影响明显；对 provenance、可解释性与协作沟通的需求长期存在[ R22 ]。  
- Wrangler、Proactive Wrangling 强调 mixed-initiative：系统提出建议，但必须允许用户预览、编辑、撤销与逐步收敛意图[ R23, R24 ]。  
- Notebook 研究指出 exploration vs explanation 的张力：用户需要保留过程、分支与证据，以便复现、沟通与审计[ R25 ]。  

> 小结：长链 agentic framework 并不会消灭交互需求，反而会把交互从“做图”提升为“治理正确性、信任与方向”的控制面板。

---

## 3) 我们的贡献/价值可以如何成立（且可被评估）

### 3.1 最稳健的贡献表述（建议优先选 2–3 条主打）

**C1. 把 EDA 做成可持续运行、可追溯、可回放的 agentic framework**  
现有 agentic data-science 工作往往以“完成指定任务/达成指标”为终点（竞赛、建模、写代码），而 EDA 的本质是开放式探索与 sensemaking[ R11–R13 ]。贡献点可强调：
- dataset 驱动持续探索（frontier 扩展）与停止/预算机制；  
- event-sourcing 式运行记录（events/state/artifacts）以支持回放与调试（与 VisTrails/noWorkflow 的 provenance 传统对齐）[ R26, R27 ]。  

**C2. Insight 作为一等对象：claim-evidence-provenance 的强约束契约**  
InsightLens 指出“对话日志并不足以管理 insight”，需要把 insight 抽取、组织并绑定证据[ R8 ]。我们可进一步把 evidence grounding 做成强契约：
- 每条 insight 绑定可追溯工件（plots/stdout/stderr/代码快照），避免“只写结论”；  
- 与“带引用的答案生成”范式一致，属于可被审计/可复查的方向（WebGPT、GopherCite）[ R28, R29 ]。  

**C3. 面向监控/覆盖/导航的可视化交互：把人从‘看 chat log’升级到‘看探索地图’**  
可视化 recommendation 与覆盖研究（Lux、Voyager、SeeDB）说明：覆盖/推荐能改善探索效率并降低“错过重要区域”的风险[ R30–R32 ]。  
在 agentic EDA 中，UI 的价值应上升为：
- 覆盖与空白可见（coverage / empty space explainable）；  
- 发现重复与偏置（dedup / focus bias）；  
- 快速定位证据与失败原因（debuggability / provenance）；  
- 支持用户接管与重定向（mixed-initiative）。  

**C4（可选）. 面向 EDA 的 insight 表述与标注：从‘单标签细粒度 taxonomy’转向‘报告式 + 多标签 data-fact types’**  
Foresight 代表“细粒度 insight type / guidepost”的统计学视角[ R33 ]，但当 insight 更像“简短 report bullet/section”时，单标签可能失配。可贡献：
- 提出并验证“report-first insight object + multi-label facts”的标注与消费方式（与 InsightLens 的 insight object 化一致）[ R8 ]。  
（项目内部已有设计讨论：`docs/Framework/Insight Taxonomy.md`）

### 3.2 “价值”如何被量化（建议写成评测表）

仅用“答案是否正确”不足以评估 EDA agent。更合理的指标集合：
- **Faithfulness / Grounding**：insight 的数值与结论是否可由绑定证据复核？引用是否充分且不误导[ R28, R29 ]。  
- **Coverage**：在预算内覆盖了多少重要维度/分组/时间段/列组合？（借鉴 Lux/Voyager/SeeDB 的覆盖与推荐视角）[ R30–R32 ]。  
- **Novelty / Redundancy**：重复探索/重复结论比例是否降低？  
- **Recovery / Robustness**：执行失败后的自恢复率、重试成本、失败可定位性（TaskWeaver、DatawiseAgent、Jupiter 强调长链可执行性）[ R2, R3, R6 ]。  
- **Human Steering Cost**：用户在 UI 中找到证据、纠错、重定向探索的时间与交互步骤（对齐 HAI 指南中的“efficient correction”）[ R34, R35 ]。  

同时可结合现有 data-analysis agent benchmark/评测基准（DataSciBench、InfiAgent-DABench、Jupiter 等）构建对照实验或自动化评分[ R36–R38 ]。

---

## 4) 为什么 Agentic Framework 仍然需要“人 + 可视化交互”？（可直接当 rebuttal）

### 4.1 EDA 的目标本身依赖人类价值判断
EDA 不只是“算出统计量”，更是决定什么是重要/可行动的发现。Sensemaking 研究表明问题会在探索中被不断重构；自动化可以加速“算”，但难替代“定方向/定重要性”的决策[ R11–R13 ]。

### 4.2 Agent 会犯错：关键不在“永远正确”，而在“可快速纠错”
LLM 与 agent 系统普遍存在幻觉与不可靠性问题，因此“人类监督 + 证据审计”是现实约束[ R39 ]。  
HAI 指南与 mixed-initiative 原则强调：系统必须让用户理解能力边界、提供可控性、并支持快速纠错与回退[ R34, R35 ]。

### 4.3 自动化会带来新的失效模式：越自动，越需要态势感知与信任校准
经典人因结论：自动化会引发 out-of-the-loop 与自动化偏置等问题，使人更难发现错误并在异常时接管[ R18–R21 ]。  
因此交互 UI 的功能不是“锦上添花”，而是：让用户看到覆盖、链路与证据，校准信任，并在错误发生时提供接管与重定向通道。

### 4.4 经验研究直接证明：真实分析工作需要交互、历史、证据与可沟通性
企业访谈、数据清洗与 notebook 研究一致指出：分析流程充满迭代、分支与返工；用户需要预览、对比、撤销、记录过程并生成可沟通产物[ R22–R25 ]。  
在 LLM 数据分析场景中，InsightLens 与 TiInsight 都把交互 UI 作为核心组成来解决上下文管理、insight 组织与可用性问题[ R7, R8 ]。

---

## 5) 对我们系统设计与论文写作的直接建议

### 5.1 论文贡献写法（模板）
可以用如下句式组织贡献（选 2–3 条即可）：
1) **Framework**：提出一个可持续运行、可追溯、可回放的 agentic EDA framework，将 EDA 从单次对话助手提升为可管理的探索过程。  
2) **Insight Contract**：定义 insight 作为一等对象的 claim-evidence-provenance 契约，并在系统中强制 evidence grounding。  
3) **Interactive Visual Control Plane**：设计面向覆盖、导航、调试与接管的可视化交互界面，支持用户对长链探索进行高效监督与重定向。  
4) **Evaluation**：提出并实证验证一组面向 EDA agent 的评测指标（grounding、coverage、recovery、human steering cost）。

### 5.2 Related Work 写法（结构建议）
按 3+1 块组织（第 2 节），每块各选 2–4 篇代表作。关键是论证“缺口”：
- agentic frameworks 缺少 EDA sensemaking 的专门治理与 UI；  
- EDA/BI 系统缺少长链自主探索 engine；  
- insight 管理系统缺少端到端的可回放执行框架；  
- 我们把三者融合并用可评测指标证明其价值。

---

## References（建议作为 related work 的核心引用池）

> 说明：优先给出可公开访问的链接（arXiv / PDF / 项目页）。部分经典论文可能存在付费墙，可用题目 + DOI 作为引用。

- [R1] Data Interpreter: An LLM Agent for Data Science (arXiv:2402.18679). https://arxiv.org/abs/2402.18679  
- [R2] TaskWeaver (arXiv:2311.17541). https://arxiv.org/abs/2311.17541  
- [R3] DatawiseAgent (arXiv:2503.07044). https://arxiv.org/abs/2503.07044  
- [R4] AutoKaggle (arXiv:2410.20424). https://arxiv.org/abs/2410.20424  
- [R5] AIDE (arXiv:2502.13138). https://arxiv.org/abs/2502.13138  
- [R6] Jupiter (arXiv:2509.09245). https://arxiv.org/abs/2509.09245  
- [R7] TiInsight (arXiv:2412.07214). https://arxiv.org/abs/2412.07214  
- [R8] InsightLens (arXiv:2404.01644). https://arxiv.org/abs/2404.01644  
- [R9] ReAct (arXiv:2210.03629). https://arxiv.org/abs/2210.03629  
- [R10] Data Formulator (VIS 2023) (arXiv:2308.00189). https://arxiv.org/abs/2308.00189  

- [R11] Tukey, J. W. (1977). Exploratory Data Analysis. (Book)  
- [R12] Pirolli, P., & Card, S. (2005). The sensemaking process and leverage points for analyst technology. (IEEE InfoVis / report)  
- [R13] Russell, D., Stefik, M., Pirolli, P., & Card, S. (1993). The Cost Structure of Sensemaking (CHI). PDF: https://www2.parc.com/istl/groups/uir/publications/items/UIR-1993-05-Russell-CHI93.pdf  
- [R14] Thomas, J., & Cook, K. (eds.) (2005). Illuminating the Path: The R&D Agenda for Visual Analytics. Record: https://www.osti.gov/biblio/910974  
- [R15] Shneiderman, B. (1996). The Eyes Have It: A Task by Data Type Taxonomy ... PDF: https://www.cs.umd.edu/~ben/papers/Shneiderman1996eyes.pdf  
- [R16] Heer, J., & Shneiderman, B. (2012). Interactive Dynamics for Visual Analysis (CACM). https://idl.cs.washington.edu/papers/interactive-dynamics/  
- [R17] Yi, J. S., Kang, Y., Stasko, J., & Jacko, J. (2007). Toward a Deeper Understanding ... PDF: https://www.cc.gatech.edu/~stasko/papers/infovis07-interaction.pdf  

- [R18] Bainbridge, L. (1983). Ironies of Automation (Automatica). https://www.sciencedirect.com/science/article/pii/0005109883900468  
- [R19] Endsley, M. R., & Kiris, E. O. (1995). The out-of-the-loop performance problem ... (Human Factors)  
- [R20] Lee, J. D., & See, K. A. (2004). Trust in Automation ... PDF: https://ltu.diva-portal.org/smash/get/diva2:1032609/FULLTEXT01.pdf  
- [R21] Parasuraman, R., & Manzey, D. H. (2010). Complacency and bias in human use of automation ... (Human Factors) PDF: https://www.researchgate.net/publication/40880041  

- [R22] Kandel, S., Paepcke, A., Hellerstein, J., & Heer, J. (2012). Enterprise Data Analysis and Visualization: An Interview Study. PDF: https://www.cs.washington.edu/educate/courses/cse512/15sp/papers/kandel.pdf  
- [R23] Kandel, S., Heer, J., Plaisant, C., et al. (2011). Wrangler ... PDF: https://vis.stanford.edu/files/2011-Wrangler-CHI.pdf  
- [R24] Guo, H., Kandel, S., Hellerstein, J., & Heer, J. (2011). Proactive Wrangling ... PDF: https://courses.cs.washington.edu/courses/cse512/13wi/readings/ProactiveWrangling.pdf  
- [R25] Rule, A., Tabard, A., & Hollan, J. (2018). Exploration and Explanation in Computational Notebooks ... PDF: https://hal.science/hal-03054873/document  

- [R26] VisTrails provenance system (paper landing): https://www.microsoft.com/en-us/research/publication/vistrails-enabling-interactive-multiple-view-visualizations/  
- [R27] noWorkflow (provenance for scripts): https://github.com/gems-uff/noworkflow  
- [R28] WebGPT (arXiv:2112.09332). https://arxiv.org/abs/2112.09332  
- [R29] GopherCite (arXiv:2203.11147). https://arxiv.org/abs/2203.11147  

- [R30] Lux (arXiv:2105.00121). https://arxiv.org/abs/2105.00121  
- [R31] Voyager (paper page). https://www.kimhal.archi/voyager/  
- [R32] SeeDB (PVLDB). PDF: https://pmc.ncbi.nlm.nih.gov/articles/PMC6694881/  
- [R33] Foresight (arXiv:1707.03877). https://arxiv.org/abs/1707.03877  

- [R34] Amershi et al. (2019). Guidelines for Human–AI Interaction (CHI). https://www.microsoft.com/en-us/research/publication/guidelines-for-human-ai-interaction/  
- [R35] Horvitz (1999). Principles of Mixed-Initiative User Interfaces (CHI). PDF: https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/chi99_mixedinitiative.pdf  

- [R36] DataSciBench (arXiv:2409.07790). https://arxiv.org/abs/2409.07790  
- [R37] InfiAgent-DABench (arXiv:2401.14255). https://arxiv.org/abs/2401.14255  
- [R38] Jupiter (same as R6).  

- [R39] A Survey of Hallucination in Large Language Generation (arXiv:2309.01219). https://arxiv.org/abs/2309.01219  

