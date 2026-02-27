# Exploration Method (MVP)

This document describes the **current** exploration method implemented in the codebase.

---

## 1) Planning Principles

### Priority = Impact only
- Priority is computed from **impact** (coverage), not from novelty or significance.
- Impact is computed using:
  - `magnitude_hint` if provided
  - fallback to auto-selected magnitude column
  - if no magnitude column, fallback to row coverage

### Novelty control (drill-down only)
Novelty is enforced by **two layers**, but not baked into priority:
1) **Planner prompt** includes the full existing insights list to discourage duplicates.
2) **Embedding dedup** filters drill-down plans by cosine similarity against existing plans/insights.

---

## 2) Drill-down Rules

- Initial plans: **reason optional** (dataset is the only cue).
- Drill-down plans: **reason required**.  
  If no valid reason, the plan is discarded.
- Drill-down generation may return **0–3** plans; no padding.

---

## 3) Evidence-First Insights

### Evidence requirements
Analyzer always runs at least one `execute_code` call.
Artifacts are saved per attempt:
- `stdout` / `stderr`
- `plots`

### Summarizer behavior
Summarizer uses the **full chronological analysis process** and outputs:
- `insight_plain`
- `insight_with_evidence` (inline evidence markers, not appended at end)

---

## 4) Embeddings

Every new Plan and Insight is embedded via the **Embedder**:
- Backend is configurable (`EMBEDDING_BACKEND="openai"|"ollama"` in backend/config.py)
- Vectors are stored directly on the Plan/Insight object
- Used for Map view clustering and drill-down dedup

---

## 5) Stopping Rule (MVP)
- Analyzer must call `complete_analysis` to finish.
- Orchestrator stops when:
  - max_steps reached
  - frontier exhausted
  - max_failures reached (final status becomes `failed`)
  - a stop is requested (Run Gateway creates `runs/{run_id}/STOP` and/or SIGTERM), resulting in `status="stopped"`
