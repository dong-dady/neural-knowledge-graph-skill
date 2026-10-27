---
name: neural-knowledge-graph
description: Use when building, upgrading, or embedding an interactive force-directed neural knowledge graph (HTML5 Canvas or zero-CDN web view) to explore many-to-many semantic relationships across notes, links, tags, categories, tools, or documents. Do NOT use for simple website flowcharts, step-by-step diagrams, or port-based DAG workflow pipeline node editors.
---

# Neural Knowledge Graph

## Purpose

Build or upgrade interactive **force-directed neural knowledge graphs** where hundreds of items, tags, tools, and categories self-organize into organic neuron-like clusters (`Hub -> Sub-Hub -> Leaf Tuft`), cross-category bridges, and outer orphan rings without external CDN or runtime library dependencies.

## Scope & Strict Non-Triggers

### Use When
- The user wants an interactive **neural knowledge graph** (`신경망 그래프`, `신경망 노드`, `지식 그래프`, `노드 연결망`) for notes, bookmarks, archives, research papers, wiki pages, or multi-tag datasets.
- An existing force graph looks like a rigid ring (`강강수월래` / circular perimeter layout) or a tangled hairball and needs organic neuron-tuft clustering.
- The user wants floating glass graph controls: collapsible side panel, translucent glass mode, `tag:`/`category:`/`tool:` search autocomplete, connected-group color propagation, isolated orphan toggle (`미연결 노트`), zoom-adaptive micro-arrows (`→`/`↔`), and responsive tension sliders.

### Do NOT Use When
- **Simple flowcharts, sequence diagrams, or architecture hierarchies** for a webpage or document -> use standard Mermaid or SVG diagrams.
- **Port-based workflow automation / DAG pipeline editors** where rectangular step cards connect sequentially via left-to-right input/output ports. Those are directed execution pipelines, not force-directed semantic neural graphs.
- **Codebase AST / file dependency indexing CLI** -> use a dedicated repository dependency indexer.

---

## Core Architecture

### 1. Three-Tier Neural Dendrite Topology (No Hairballs, No Duplicate Facets)

To achieve an organic neural structure instead of a flat star or tangled web, classify nodes and edges into three semantic tiers plus bridges and orphans:

1. **Primary Cluster Hubs (`node.cluster = true`)**
   - Top-level organizing facet (default: `category`, switchable to `tag`, `domain`, or `month`).
2. **Local Sub-Hubs & Cross-Cluster Bridges (`kind = 'facet'`)**
   - Secondary facets (`tag`, `tool`, `contentType`, `domain`) that appear on **2 or more items** (`count >= 2`) act as local dendrite sub-hubs (`subclusterKey`).
   - **Unique Semantic Identity**: Never duplicate a shared tag/tool node per category. Track `clusterKeys` and `clusterWeights` on each facet node; when `clusterKeys.length > 1`, mark `node.bridgeFacet = true`.
3. **Leaf Items (`kind = 'link'`)**
   - Connect each leaf item primarily to its `parentHubKey = subclusterKey || clusterKey` (`kind = 'dendrite'`, `bridgeLinkEdge = false`).
   - Fold redundant secondary facet-to-leaf edges (`bridgeLinkEdge = true`) so leaves sprout outward as crisp neuron tufts around their sub-hub instead of being stretched across the canvas. Allow at most one secondary `synapse` edge for non-bridge local hubs (`count >= 4`).
4. **Sequential / Mutual Reference Edges (`bidirectional = true`)**
   - Items belonging to the same sequential context bundle (e.g., `[Bundle N/M]` or `[연속 묶음 N/M]`) or cross-cluster bridge axons (`axon` to a `bridgeFacet`) set `bidirectional: true` so they render mutual arrows (`↔`) and pull related items together.
5. **Isolated / Unconnected Orphan Notes (`orphanNode = true`)**
   - Items with no multi-item sub-hub (`!subclusterKey`) and a generic/empty category (`'기타'`, `'미분류'`, `'Uncategorized'`, `''`), or zero active facet links, are marked `orphanNode: true` with **zero edges**.
   - Place them in a soft outer periphery halo (`orphanTargetX`, `orphanTargetY`) when `미연결 노트` (`#graph-show-orphans`) is ON, and omit them when OFF.

---

### 2. Organic Force Physics (Anti-Circle / Anti-"강강수월래")

Never lock cluster anchors or leaf nodes onto a uniform circle perimeter.

- **2D Interior Golden-Angle Seeding**:
  - Order cluster hubs by inter-cluster bridge relation weight (`graphOrderClustersByRelations`), then seed their anchors using a golden-angle spiral (`index * 2.399963229728653`) with deterministic hash jitter (`graphHash`) and elliptical aspect ratio (`1.22 : 0.92`).
- **Wide-Range Center Force (`중심 장력`, `0 ~ 500`)**:
  - Couple `centerForce` both to velocity gravity (`-node.x * centerForce * gravityWeight * alpha`) **and** to a dynamic `centerCompression` factor (`Math.max(0.28, 1.42 - (centerRaw / 500) * 1.08)`) applied to cluster and sub-hub anchor distances. Sliding `중심 장력` must visibly contract or expand the entire neural network in real time.
- **Outward Dendrite Tuft Repulsion**:
  - In Coulomb many-body repulsion, apply an asymmetric hub-to-leaf charge multiplier (`0.72` for own parent hub, `2.4` for foreign hubs) and seed leaves along the outward vector away from the parent cluster center (`Math.atan2(py - cy, px - cx)`), with deterministic per-leaf stem lengths (`stemRatio = 0.25 + hash * 0.95`).

---

### 3. Floating Glass UI & Interaction Contract

Render all graph controls inside the canvas container (`.graph-canvas`) as floating overlays:

1. **Collapsible Right Floating Sidebar (`#graph-sidebar` & `#graph-sidebar-launcher`)**
   - Header includes **Translucent Glass Toggle (`반투명`)**, **Refresh & Replay (`↻`)**, and **Collapse (`✕`)**.
   - When `#graph-sidebar` is open on desktop (`width > 820`), `fitGraphToView()` must subtract the sidebar width (`usableWidth = width - 330`) so the graph center is never hidden under the panel.
   - Always include CSS `.graph-floating-launcher[hidden], .nkg-search-popover[hidden], .graph-controls.collapsed { display: none !important; }` so flex containers respect the HTML `hidden` attribute.
2. **Filter Section (`필터`)**
   - Search box (`#graph-search-input`) with focus popover (`#graph-search-popover`) offering `tag:`, `category:`, `tool:`, `path:`, `file:`, and `line:` syntax tokens.
   - Facet checkboxes (`category`, `tag`, `domain`, `contentType`, `tool`, `status`, `month`), `미연결 노트` switch (`#graph-show-orphans`), and problem/low-confidence hide buttons.
3. **Connected Group Coloring (`그룹`)**
   - Each group rule `{ id, query, color }` matches seed nodes via `nodeMatchesGraphQuery(node, query)` and propagates `node.groupColor = color` to **both the matched hub/node and all connected nodes** (cluster members + 1-hop neighbors + edges).
   - Provide quick actions: `새 그룹`, `주제별 자동 색상` (auto-assigns distinct palette colors to all categories), `초기화`, and `🎨 연결 노드 그룹 색상 지정` inside the selected node drawer.
4. **Display & Core 4 Tension Sliders (`표시`)**
   - Top-level display controls: `화살표` switch (`#graph-show-arrows`) and `텍스트 크기` slider (`data-graph-control="textSize"`).
   - Keep **only 4 core tension sliders** visible by default under `장력 (핵심 4개)`:
     - `링크 거리` (`linkDistanceRatio`)
     - `반발력` (`repulsion`)
     - `중심 장력` (`centerForce`, range `0 ~ 500`)
     - `주제 간 거리` (`clusterDistance`)
   - Put secondary controls (`노드 크기`, `링크 두께`, `텍스트 표시 시작점`, `링크 장력`, `주제 고정력`, `사이드탭 불투명도`, `덩어리 기준`) inside `<details id="graph-advanced-controls"><summary>고급 설정</summary></details>`.
5. **Zoom-Sensitive Micro-Arrows & Toggle Deselect**
   - When `graphShowArrows` is ON, omit arrowheads at far-out zoom (`zoom < 0.42` without focus) to prevent visual clutter. As the user zooms in or hovers/selects a node, draw crisp micro-arrowheads (`drawMicroArrowhead`, length `2.5px ~ 5.2px` scaled by zoom) at the target node rim (and at the source node rim when `edge.bidirectional` is true).
   - Clicking an already-selected node again (or clicking empty canvas without dragging) sets `selectedGraphNode = null` and collapses `#graph-node-drawer` to `.empty-selection`.

---

## Implementation Reference

When writing or patching code for a neural knowledge graph, read [references/physics-and-topology.md](references/physics-and-topology.md) for the complete HTML, CSS, and vanilla JS reference implementation (`buildGraph`, `isIsolatedOrphanItem`, `applyGraphGroupColors`, `arrangeGraphClusterAnchors`, `positionGraphClusterMembers`, `applyGraphPhysics`, `drawMicroArrowhead`, and `drawGraph`).

## Verification Checklist

1. **No Circular Ring**: Verify that cluster hubs and leaf nodes form organic interior clouds and outward neuron tufts, not a hollow circle.
2. **Center Force Dynamic Range**: Drag `중심 장력` from `0` to `500` and confirm the entire graph expands wide and contracts tightly.
3. **Orphan Halo**: Toggle `미연결 노트` ON/OFF and verify isolated zero-edge nodes appear/disappear around the outer periphery.
4. **Connected Group Coloring**: Verify that matching `category:...` or `tag:...` colors both the hub and all its connected nodes/edges.
5. **Micro-Arrows & Deselect**: Zoom in to confirm subtle `→` / `↔` arrowheads at node rims, and click the same node twice to confirm deselection.
