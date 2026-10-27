# neural-knowledge-graph

An agent skill for building interactive, **zero-dependency Force-Directed Neural Knowledge Graphs** (`HTML5 <canvas>` + vanilla JavaScript) to explore many-to-many semantic relationships across notes, links, documents, tags, categories, and tools.

> **Scope Notice**: This skill is designed for **organic, neuron-like semantic knowledge graphs** (`Hub -> Sub-Hub -> Leaf Tuft`), **not** simple website flowcharts or port-based DAG workflow/pipeline editors.

---

## Key Features

1. **Three-Tier Neural Dendrite Topology**
   - Organizes items into **Primary Cluster Hubs** (`Category`), **Shared Local Sub-Hubs** (`Tag`/`Tool` with `count >= 2`), **Cross-Cluster Bridge Nodes** (`bridgeFacet`), **Leaf Tufts**, **Bidirectional Context Bundles** (`↔`), and an **Outer Unconnected Orphan Ring** (`미연결 노트`).
   - Preserves unique node identity across clusters (shared tags act as single bridge nodes rather than duplicating per category).
2. **Anti-Ring ("강강수월래" 방지) Organic Force Physics**
   - Uses 2D interior golden-angle spiral seeding with elliptical aspect ratio and asymmetric hub-to-leaf Coulomb repulsion so leaf nodes fan outward like neuron dendrites instead of forming a hollow circle.
   - Couples `centerForce` (`0 ~ 500`) to both velocity gravity and cluster radial compression so sliding `중심 장력` visibly contracts or expands the entire web in real time.
3. **Collapsible Floating Glass Sidebar & UX**
   - Translucent glass mode (`반투명`), one-click refresh/replay (`↻`), search box with syntax autocomplete (`tag:`, `category:`, `tool:`, `path:`, `file:`, `line:`), **connected-group color propagation**, zoom-adaptive directional/bidirectional micro-arrows (`→`/`↔`), 4 core tension sliders, and click-again deselection.

---

## Repository Structure

```text
neural-knowledge-graph/
├── SKILL.md                             # Core skill instructions & trigger boundaries
├── agents/
│   └── openai.yaml                      # UI metadata & invocation config
└── references/
    └── physics-and-topology.md          # Complete HTML/CSS/JS physics & topology reference
```

---

## Installation

Clone or copy into your global or project skills directory:

```bash
git clone https://github.com/dong-dady/neural-knowledge-graph-skill.git ~/.agents/skills/neural-knowledge-graph
```

## License

MIT
