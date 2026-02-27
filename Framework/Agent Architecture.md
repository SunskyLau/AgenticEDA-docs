# Agentic EDA Framework - Current MVP Architecture

This document is the **source of truth** for the current MVP implementation.  
Everything below reflects the actual code paths. Items not implemented are explicitly marked.

---

## Implemented (Current MVP)

### Core Loop (Planner -> Dispatcher -> Analyzer -> Summarizer)
- **Planner** generates initial analysis plans (no required reason).
- **Dispatcher** schedules plans via priority-first execution.
- **Analyzer** runs multi-turn tool protocol (schema → reflection → code → reflection → complete).
- **Summarizer** produces a single insight in two forms:
  - `insight_plain`
  - `insight_with_evidence` (inline evidence markers, no forced append)

### Evidence & Artifacts
- Each `execute_code` emits **stdout / stderr / plots** as separate artifacts (per attempt).
- Evidence markers reference artifact paths only.
- Summarizer consumes the **chronological analysis process** (including reflections, code, stdout/stderr, plots).

### Embedding & Dedup (Novelty Control)
- Every new **Plan** and **Insight** gets a text embedding via the **Embedder** (backend/framework/embedder.py).
  - Backend is configurable (`EMBEDDING_BACKEND="openai"|"ollama"` in backend/config.py).
  - The current default is an **OpenAI-compatible embeddings API** (not Ollama-only).
- Drill-down plans are filtered by embedding cosine similarity to avoid duplicates.
- Planner drill-down prompt includes the **full existing insights list** (plain text) to discourage repetition (second layer of novelty control).

### Planner Reason / Motivation
- **Initial plans**: reason optional (dataset is the only cue).
- **Drill-down plans**: reason required. Plans without a valid reason are discarded.
- UI shows `Motivation` field in the plan inspector.

### Streaming & UI
- SSE event streaming is **incremental** (true file tail with partial line buffer).
- Analysis stream outputs are smoother (Python subprocess unbuffered).
- Running plan is visible immediately after load (status hydrated from events).

### Views
- **Tree** view (hierarchical)
- **Graph** view (node-link)
- **Map** view (embedding-aware force layout)
  - Similar nodes cluster by embedding similarity
  - Voronoi lineage regions colored by "Depth Level" roots (plans at the selected depth)
  - Zoom scales canvas only (no hierarchy gating)
  - Selecting a node highlights its lineage path and region

### Reporter (Long-form Insight Report)
- Optional: generate an evidence-backed report for an Insight (and its drill-down chain).
- Artifacts:
  - `artifacts/reports/report_{insight_id}.md`
  - `artifacts/report_packs/{insight_id}/...` (reporter inputs snapshot)
- Frontend triggers via Run Gateway `POST /api/runs/:runId/report` and receives a `report_generated` event.

### Stop / Shutdown
- The exploration can be stopped gracefully:
  - Run Gateway creates a `STOP` file under the run directory, and/or sends SIGTERM.
  - Orchestrator polls for stop requests and transitions the run state to `stopped`.
  - A final `run_completed` event is emitted with `final_status="stopped"`.
- Budget-based termination:
  - `max_failures` results in `status="failed"`; other budget/frontier exhaustion results in `status="completed"`.

---

## Not Implemented (Planned / Deferred)

- Global, multi-run insight memory and cross-run novelty tracking
- Performance-optimized force layout (current MVP uses lightweight custom physics)
- Advanced summarizer scoring beyond evidence grounding
- Replay/branching UI (removed for MVP simplification)

---

## Key Data Structures (Current)

### PlanItem
- `text`: analysis plan text
- `filters`: structured filters
- `reason`: optional initial, required for drill-down
- `priority_params`: `impact_filters`, `magnitude_hint`
- `embedding`: vector from the configured embedding backend (OpenAI-compatible or Ollama)

### Insight
- `insight_plain`
- `insight_with_evidence`
- `embedding`

---

## Key Implementation Details

### Planner
- Generates JSON array only
- Drill-down prompt receives:
  - parent insight
  - dataset schema
  - full existing insight list
- Enforces reason validity for drill-down

### Analyzer
- Tool protocol enforced: schema → reflection → code → reflection → complete
- Plots auto-saved into `artifacts/plots/`
- Full analysis stream saved in `artifacts/sessions/{plan}.analysis.json`

### Summarizer
- Reads analysis stream in chronological order
- Uses inline evidence markers with hard constraints

### Orchestrator
- Priority = impact only
- Embeddings computed on new plans/insights
- Drill-down plans deduped by embedding similarity
