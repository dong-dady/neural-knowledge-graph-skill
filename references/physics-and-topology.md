# Neural Knowledge Graph — Physics, Topology & UI Reference

Use this reference when implementing or upgrading an interactive force-directed neural knowledge graph in HTML5 `<canvas>` and vanilla JavaScript.

---

## 1. Floating Glass Sidebar & Canvas HTML Structure

```html
<div class="graph-canvas">
    <canvas id="graph-canvas" role="img" aria-label="관계 신경망 그래프"></canvas>
    <div id="graph-tooltip" role="tooltip" hidden></div>

    <!-- Compact launcher shown when floating sidebar is collapsed -->
    <div id="graph-sidebar-launcher" class="graph-floating-launcher" hidden>
        <button type="button" onclick="resetAllGraphSettingsAndReplay()" title="새로고침">↻</button>
        <button type="button" onclick="toggleGraphSidebar(true)">⚙️ 필터 · 설정 열기</button>
    </div>

    <!-- Collapsible floating glass sidebar -->
    <div id="graph-sidebar" class="graph-controls" aria-label="그래프 노드 표시">
        <div class="nkg-sidebar-header">
            <span class="nkg-sidebar-title">신경망 그래프 설정</span>
            <div class="nkg-icon-actions">
                <button id="graph-translucent-toggle" class="nkg-icon-btn" type="button" onclick="toggleGraphTranslucent()" title="반투명 보기 전환">반투명</button>
                <button id="graph-refresh-btn" class="nkg-icon-btn" type="button" onclick="resetAllGraphSettingsAndReplay()" title="그래프 초기화 및 새로고침">↻</button>
                <button id="graph-close-sidebar" class="nkg-icon-btn" type="button" onclick="toggleGraphSidebar(false)" title="사이드 탭 닫기">✕</button>
            </div>
        </div>

        <details class="nkg-section" open>
            <summary>필터</summary>
            <div class="nkg-section-body">
                <div class="nkg-search-wrap">
                    <input id="graph-search-input" type="search" placeholder="🔍 노드 검색... (tag:, category:, tool:)" autocomplete="off" onfocus="showGraphSearchPopover(this)" oninput="handleGraphSearchInput(this)">
                    <div id="graph-search-popover" class="nkg-search-popover" hidden>
                        <div class="nkg-search-popover-head"><span>검색 옵션</span><span>ⓘ</span></div>
                        <button type="button" class="nkg-search-opt" onmousedown="insertGraphSearchToken('tag:#')"><b>tag:</b><span>태그 검색 (예: tag:#agent)</span></button>
                        <button type="button" class="nkg-search-opt" onmousedown="insertGraphSearchToken('category:')"><b>category:</b><span>일치하는 주제 카테고리</span></button>
                        <button type="button" class="nkg-search-opt" onmousedown="insertGraphSearchToken('tool:')"><b>tool:</b><span>일치하는 도구 이름</span></button>
                        <button type="button" class="nkg-search-opt" onmousedown="insertGraphSearchToken('path:')"><b>path:</b><span>일치하는 도메인/경로</span></button>
                        <button type="button" class="nkg-search-opt" onmousedown="insertGraphSearchToken('file:')"><b>file:</b><span>일치하는 자료 제목</span></button>
                        <button type="button" class="nkg-search-opt" onmousedown="insertGraphSearchToken('line:')"><b>line:</b><span>요약·본문 내 키워드 검색</span></button>
                    </div>
                </div>
                <div class="nkg-switch-row">
                    <span>미연결 노트 (홀로 떨어진 노드)</span>
                    <label class="nkg-switch">
                        <input id="graph-show-orphans" type="checkbox" checked onchange="toggleGraphOrphans(this.checked)">
                        <span class="nkg-slider"></span>
                    </label>
                </div>
            </div>
        </details>

        <details class="nkg-section" open>
            <summary>그룹 (연결 노드 동일 색상)</summary>
            <div class="nkg-section-body">
                <div id="graph-group-list" class="nkg-group-list"></div>
                <div class="nkg-group-actions">
                    <button type="button" onclick="addGraphColorGroup()">새 그룹</button>
                    <button type="button" onclick="populateCategoryColorGroups()">주제별 자동 색상</button>
                    <button type="button" onclick="clearGraphColorGroups()">초기화</button>
                </div>
            </div>
        </details>

        <details class="nkg-section" open>
            <summary>표시</summary>
            <div class="nkg-section-body">
                <div class="nkg-switch-row">
                    <span>화살표 (참조 방향 · 확대 시 선명)</span>
                    <label class="nkg-switch">
                        <input id="graph-show-arrows" type="checkbox" checked onchange="toggleGraphArrows(this.checked)">
                        <span class="nkg-slider"></span>
                    </label>
                </div>
                <label class="nkg-slider-field">
                    <div class="nkg-slider-head"><span>텍스트 크기</span></div>
                    <input type="range" min="65" max="180" step="5" value="100" data-graph-control="textSize">
                </label>

                <div style="font-size:11.5px; font-weight:700; margin-top:4px;">장력 (핵심 4개)</div>
                <div class="graph-physics-controls">
                    <label class="nkg-slider-field">
                        <div class="nkg-slider-head"><span>링크 거리</span></div>
                        <input type="range" min="35" max="240" step="5" value="95" data-graph-control="linkDistanceRatio">
                    </label>
                    <label class="nkg-slider-field">
                        <div class="nkg-slider-head"><span>반발력</span></div>
                        <input type="range" min="80" max="900" step="20" value="340" data-graph-control="repulsion">
                    </label>
                    <label class="nkg-slider-field">
                        <div class="nkg-slider-head"><span>중심 장력 (전체 응집력)</span></div>
                        <input type="range" min="0" max="500" step="5" value="140" data-graph-control="centerForce">
                    </label>
                    <label class="nkg-slider-field">
                        <div class="nkg-slider-head"><span>주제 간 거리</span></div>
                        <input type="range" min="200" max="1100" step="20" value="520" data-graph-control="clusterDistance">
                    </label>
                </div>

                <details id="graph-advanced-controls" class="graph-advanced-controls">
                    <summary>고급 설정</summary>
                    <div>
                        <label class="nkg-slider-field">
                            <div class="nkg-slider-head"><span>노드 크기</span></div>
                            <input type="range" min="55" max="180" step="5" value="100" data-graph-control="nodeSize">
                        </label>
                        <label class="nkg-slider-field">
                            <div class="nkg-slider-head"><span>링크 두께</span></div>
                            <input type="range" min="40" max="240" step="10" value="100" data-graph-control="linkWidth">
                        </label>
                        <label class="nkg-slider-field">
                            <div class="nkg-slider-head"><span>텍스트 표시 시작점</span></div>
                            <input type="range" min="55" max="260" step="5" value="130" data-graph-control="textZoomThreshold">
                        </label>
                        <label class="nkg-slider-field">
                            <div class="nkg-slider-head"><span>링크 장력</span></div>
                            <input type="range" min="5" max="600" step="5" value="145" data-graph-control="linkForce">
                        </label>
                        <label class="nkg-slider-field">
                            <div class="nkg-slider-head"><span>주제 고정력</span></div>
                            <input type="range" min="0" max="1000" step="10" value="65" data-graph-control="clusterStrength">
                        </label>
                        <label class="nkg-slider-field">
                            <div class="nkg-slider-head"><span>사이드탭 불투명도</span></div>
                            <input type="range" min="25" max="100" step="5" value="90" data-graph-control="sidebarOpacity">
                        </label>
                    </div>
                </details>
            </div>
        </details>
    </div>

    <aside id="graph-node-drawer" class="graph-panel empty-selection">
        <h2 id="graph-title">노드를 선택하세요</h2>
        <div id="graph-links" class="subtle">노드를 클릭하면 연결된 자료가 표시됩니다.</div>
    </aside>
</div>
```

---

## 2. Essential CSS Rules

Always ensure `[hidden]` overrides `display: flex` on floating launchers and search popovers:

```css
.graph-canvas {
    position: relative;
    border-radius: 12px;
    background: #0d1117;
    overflow: hidden;
    min-height: 760px;
}
.graph-floating-launcher[hidden],
.nkg-search-popover[hidden],
.graph-controls.collapsed {
    display: none !important;
}
.graph-controls {
    position: absolute;
    top: 14px;
    right: 14px;
    z-index: 6;
    width: 318px;
    max-height: calc(100% - 28px);
    overflow-y: auto;
    border: 1px solid rgba(255, 255, 255, .13);
    border-radius: 12px;
    background: rgba(22, 27, 34, var(--sidebar-alpha, 0.92));
    padding: 12px;
    backdrop-filter: blur(14px);
    -webkit-backdrop-filter: blur(14px);
}
.graph-controls.translucent {
    --sidebar-alpha: 0.56;
    backdrop-filter: blur(8px);
}
```

---

## 3. Three-Tier Neural Topology & Orphan Detection (JS)

```javascript
function isIsolatedOrphanItem(item, subclusterKey, types) {
    if (!types.length) return true;
    if (item.isBundle) return false;
    const cat = String(item.category || '').trim();
    const isGenericCat = !cat || cat === '기타' || cat === '미분류' || cat === 'Uncategorized';
    const tags = item.tags || [];
    const tools = splitFacetValues(item.tool);
    if (!subclusterKey && isGenericCat) return true;
    if (!subclusterKey && tags.length === 0 && tools.length === 0) return true;
    if (!subclusterKey && item.itemType === 'text' && tags.length <= 1 && tools.length === 0) return true;
    return false;
}
```

Key topology rules in `buildGraph(rows)`:
1. Pre-count facet frequencies across `rows`. A facet with `count >= 2` becomes a candidate `subclusterKey`.
2. In Pass 1, skip items where `isIsolatedOrphanItem(item, subclusterKey, types)` is `true` so orphan items do not artificially inflate hub counts.
3. In Pass 2:
   - If `isOrphan && !graphShowOrphans`, omit the item.
   - If `isOrphan && graphShowOrphans`, add the item with `orphanNode: true, parentHubKey: null, clusterKey: null, degree: 0` and **no edges**.
   - Connect consecutive bundle items (`item.isBundle`) with `kind: 'bundle_ref', bidirectional: true, bridgeLinkEdge: false`.
   - Connect `clusterKey => nodeKey` (`kind: 'axon'`, `bidirectional: Boolean(targetFacetNode?.bridgeFacet)`).
   - Connect `parentHubKey => linkKey` (`kind: 'dendrite'`, `bridgeLinkEdge: false`).
   - Fold redundant secondary facet-to-leaf edges (`bridgeLinkEdge: true`), allowing at most one `synapse` (`!facetNode.bridgeFacet && facetNode.count >= 4`).

---

## 4. Connected Group Color Propagation (JS)

When a group query matches a hub, tag, tool, or keyword, propagate `groupColor` to the matched node **and** all its connected nodes and edges:

```javascript
function applyGraphGroupColors(nodes = graphState.nodes, edges = graphState.edges) {
    const nodeByKey = new Map(nodes.map(n => [n.key, n]));
    for (const node of nodes) node.groupColor = null;
    for (const edge of edges) edge.groupColor = null;
    if (!graphColorGroups.length) return;

    for (const group of graphColorGroups) {
        const q = (group.query || '').trim();
        if (!q) continue;
        const color = group.color || '#3fb950';
        const matchedSeeds = nodes.filter(node => nodeMatchesGraphQuery(node, q));
        if (!matchedSeeds.length) continue;

        const coloredKeys = new Set();
        for (const seed of matchedSeeds) {
            coloredKeys.add(seed.key);
            if (seed.cluster) {
                for (const candidate of nodes) {
                    if (candidate.clusterKey === seed.key) coloredKeys.add(candidate.key);
                }
            }
            for (const edge of edges) {
                if (!shouldRenderGraphEdge(edge)) continue;
                if (edge.source === seed.key) {
                    coloredKeys.add(edge.target);
                    const targetNode = nodeByKey.get(edge.target);
                    if (targetNode && targetNode.kind === 'facet' && !targetNode.cluster && seed.cluster) {
                        for (const subEdge of edges) {
                            if (shouldRenderGraphEdge(subEdge) && subEdge.source === targetNode.key) {
                                coloredKeys.add(subEdge.target);
                            }
                        }
                    }
                }
                if (edge.target === seed.key) coloredKeys.add(edge.source);
            }
        }
        for (const key of coloredKeys) {
            const node = nodeByKey.get(key);
            if (node) node.groupColor = color;
        }
    }

    for (const edge of edges) {
        if (!shouldRenderGraphEdge(edge)) continue;
        const source = nodeByKey.get(edge.source);
        const target = nodeByKey.get(edge.target);
        if (source?.groupColor && target?.groupColor && source.groupColor === target.groupColor) {
            edge.groupColor = source.groupColor;
        } else if (source?.groupColor && target?.kind === 'link') {
            edge.groupColor = source.groupColor;
        }
    }
}
```

---

## 5. Golden-Angle Seeding, Center Compression & Outer Orphan Ring (JS)

```javascript
function arrangeGraphClusterAnchors(nodes = graphState.nodes) {
    const clusters = nodes
        .filter(node => node.cluster)
        .sort((a, b) => b.degree - a.degree || a.label.localeCompare(b.label));
    const baseDistance = graphState.settings.clusterDistance;
    const centerRaw = graphState.settings.centerForceRaw ?? 140;
    const centerCompression = Math.max(0.28, 1.42 - (centerRaw / 500) * 1.08);
    const relations = graphClusterRelations(clusters, nodes);
    const orderedClusters = graphOrderClustersByRelations(clusters, relations);

    orderedClusters.forEach((node, index) => {
        const hAngle = graphHash(`${node.key}::angle`);
        const hRad = graphHash(`${node.key}::rad`);
        const goldenAngle = index * 2.399963229728653 + (hAngle - 0.5) * 0.58;
        const radialStep = Math.max(110, baseDistance * 0.54 * centerCompression);
        const radius = index === 0
            ? radialStep * 0.25
            : Math.sqrt(index + 0.45) * radialStep * (0.76 + hRad * 0.52);
        node.anchorX = Math.cos(goldenAngle) * radius * 1.22;
        node.anchorY = Math.sin(goldenAngle) * radius * 0.92;
        if (!node.hasClusterAnchorPosition) {
            node.x = node.anchorX;
            node.y = node.anchorY;
            node.vx = 0;
            node.vy = 0;
            node.hasClusterAnchorPosition = true;
        }
    });
}
```

For orphan nodes (`node.orphanNode`), place them in an outer periphery ring in `positionGraphClusterMembers`:

```javascript
const baseOuterRadius = Math.max(360, (graphState.settings.clusterDistance * 1.08 + 180) * centerCompression);
orphanNodes.forEach((node, idx) => {
    const h1 = graphHash(`${node.key}::oa`);
    const h2 = graphHash(`${node.key}::or`);
    const angle = (idx + 1) * 2.399963229728653 + (h1 - 0.5) * 0.35;
    const ringRadius = baseOuterRadius + (h2 - 0.35) * 145 + Math.sqrt(idx + 1) * 12;
    node.orphanTargetX = Math.cos(angle) * ringRadius * 1.18;
    node.orphanTargetY = Math.sin(angle) * ringRadius * 0.92;
    if (!node.hasClusterPosition) {
        node.x = node.orphanTargetX;
        node.y = node.orphanTargetY;
        node.vx = 0;
        node.vy = 0;
        node.hasClusterPosition = true;
    }
});
```

---

## 6. Zoom-Sensitive Directional & Bidirectional Micro-Arrows (JS)

```javascript
function drawMicroArrowhead(ctx, fromX, fromY, toX, toY, targetRadius, arrowLen) {
    const dx = toX - fromX;
    const dy = toY - fromY;
    const dist = Math.hypot(dx, dy);
    if (dist <= targetRadius + arrowLen * 1.4) return;
    const ux = dx / dist;
    const uy = dy / dist;
    const tipX = toX - ux * (targetRadius + 0.8 / graphState.zoom);
    const tipY = toY - uy * (targetRadius + 0.8 / graphState.zoom);
    const baseX = tipX - ux * arrowLen;
    const baseY = tipY - uy * arrowLen;
    const wing = arrowLen * 0.48;
    const px = -uy * wing;
    const py = ux * wing;
    ctx.moveTo(tipX, tipY);
    ctx.lineTo(baseX + px, baseY + py);
    ctx.lineTo(baseX - px, baseY - py);
    ctx.closePath();
}
```
