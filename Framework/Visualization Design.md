# Visualization Design (MVP)

This document reflects the **current UI implementation**.

---

## 1) Views
- **Tree**: hierarchical exploration tree
- **Graph**: node-link graph
- **Semantic Map**: embedding-aware force layout
- **Coverage Grid Map**: axis-encoded grid (taxonomy × column-combinations)

---

## 2) Semantic Map View (Implemented)

### Purpose
Show the semantic landscape of exploration:
- Similar plan/insight nodes cluster
- Distant topics separate
- Lineage regions are visible by Voronoi coloring

### Layout (Force-based)
Forces applied each frame:
1) Repulsion between real nodes (`~ 1/dist`)
2) Similarity springs (edges only for `sim >= MAP_ATTR_SIM_THRESHOLD`, strength `~ sim^2`)
3) Center anchor spring (pulls toward map center)

### Region Coloring
Regions are visualized via **Voronoi cells**:
- Voronoi is computed over real nodes plus a fixed ring of "ghost" points (not rendered) to avoid unbounded hull cells
- All cells are clipped to a global boundary circle (`MAP_BOUNDARY_RADIUS`) to prevent infinite/huge outer regions

### Depth Level Coloring
A Depth Level control changes **only** the coloring strategy:
- Current depth: each plan at that depth becomes a "region root"; its descendants' Voronoi cells are filled
- Shallower than current depth: no fill (stroke only)
- Current depth roots: special marker icon

### Zoom
Zoom controls **canvas scale only** (no hierarchy gating).

### Selection
Selecting a node:
- Highlights the directed path from root -> selected node, and selected node -> descendants (arrowed)

---

## 3) Evidence in UI
Insights render:
- Inline evidence markers (e.g. `[1]`)
- Evidence list below with:
  - text artifacts (stdout/stderr)
  - plots (image previews + zoom)

---

## 4) Coverage Grid Map (Implemented)

### Purpose
Make coverage and empty space explainable:
- **X**: column combinations (derived from plan filters + plan/insight text; ordered by first appearance)
- **Y**: heuristic insight taxonomy (v0) with `Unknown`
- **Cells**: aggregate Insights; empty cells are explicitly visible (fog-of-war)

### Interaction
- Click a non-empty cell (or dot) to select an Insight and open it in Inspector.
