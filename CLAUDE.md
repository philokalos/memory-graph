# CLAUDE.md - Memory Graph

3D spatial memory visualization | Vanilla JS + Three.js (static HTML)

## Project Overview

Memory Graph is a 3D spatial memory visualization tool that connects photos, people, places, time, and tags in an interactive graph. Built with vanilla JavaScript and Three.js, it features two distinct experiences:

- **index.html**: visionOS-styled 3D graph viewer with force-directed layout
- **promotion.html**: Samsung Galaxy ecosystem landing page

## Development Setup

Since this is a static HTML project with no build system, you need a local web server due to CORS restrictions:

```bash
# Python 3
python3 -m http.server 8000

# Python 2
python -m SimpleHTTPServer 8000

# Node.js (if http-server installed)
npx http-server -p 8000

# VS Code Live Server extension
# Right-click index.html → "Open with Live Server"
```

Access at `http://localhost:8000/index.html` or `http://localhost:8000/promotion.html`

## Architecture

### Three.js Scene Structure (index.html)

The application builds a 3D force-directed graph with these core components:

**Scene Hierarchy**:
- Scene → Camera (PerspectiveCamera at 280 units radius)
- Nodes: 96 mesh objects (photos as PlaneGeometry, others as SphereGeometry)
- Links: Line objects connecting related nodes
- Lights: Ambient + Directional + 2 PointLights
- Particles: Background visual effect (200 points)

**Node System**:
```javascript
// 6 node types with distinct visual treatments
nodes = [
  { id, type: 'photo'|'person'|'place'|'time'|'tag'|'file', label, ... }
]

// Type-specific rendering:
// - photo: PlaneGeometry with image texture + border
// - others: SphereGeometry with inner glow
```

**Force-Directed Layout**:
- Runs 80 iterations on init via `applyForceLayout()`
- Repulsion force: 600 / (distance²)
- Attraction force: 0.04 * distance (for connected nodes)
- Damping: 0.85 to prevent oscillation
- Updates link positions every frame via `updateLinkPositions()`

**Camera Control**:
- Spherical coordinates: (radius, theta, phi)
- Mouse drag rotates theta/phi
- Scroll/buttons adjust radius (80-500 range)
- Focuses on targetX/Y/Z when node selected

### Data Model

**Nodes** (96 total):
- 24 photos: Unsplash images with date/location metadata
- 16 people: Person entities with emoji/desc
- 18 places: Geographic locations
- 12 time: Monthly/biannual periods
- 16 tags: Categories (음식, 여행, etc.)
- 10 files: Document references

**Links**: Source-target pairs creating semantic relationships
- Photo-Person: "who was in this photo"
- Photo-Place: "where was this taken"
- Photo-Time: "when was this"
- Photo-Tag: "what category"
- File-Place/Person: "related documents"
- Hierarchical: Place clusters, time groupings

### Performance Considerations

**Critical Paths**:
1. **Texture Preloading**: `preloadTextures()` runs before scene render
   - 24 photo textures loaded via TextureLoader
   - Promise.all waits for completion to prevent flickering

2. **Animation Loop**: Runs at requestAnimationFrame (60fps target)
   - Updates camera position
   - Updates 96+ label positions (only visible labels rendered)
   - Subtle float animation on nodes
   - Updates all link positions (can be 100+ lines)

3. **Interaction**: Mouse move triggers raycasting
   - Intersects against nodeObjects array
   - Highlights connected nodes (filters all nodes/links)
   - Use `Set` for O(1) connection lookups

**Optimization Opportunities**:
- Label visibility culling (distance < 250, vector.z < 1)
- Link visibility tied to node visibility
- Geometry instancing not used (could improve performance for many nodes)

### UI Components

**visionOS Glass UI** (index.html):
- Backdrop-filter blur(40px) for panels
- Gradient overlays for depth
- Filter pills for node type toggling
- Real-time search with incremental filtering
- Detail panel with slide-in animation

**Galaxy UI** (promotion.html):
- Samsung One UI design tokens
- Noto Sans KR font family
- Intersection Observer for scroll animations
- Responsive grid layouts (3→2→1 columns)

## File Structure

```
memory_graph/
├── index.html           # Main 3D graph viewer
│   ├── <style>         # visionOS glass design system
│   └── <script>        # Three.js scene + force layout
└── promotion.html       # Galaxy ecosystem landing page
    ├── <style>         # Samsung design tokens
    └── <script>        # Smooth scroll + animations
```

## Common Development Tasks

### Adding New Nodes

1. Add to `nodes` array with required fields:
   ```javascript
   { id: 'unique_id', type: 'photo'|'person'|..., label: '...', ... }
   ```
2. Add relationships to `links` array:
   ```javascript
   { source: 'node_id', target: 'other_node_id' }
   ```
3. Update stats in UI:
   ```javascript
   document.getElementById('node-count').textContent = nodes.length;
   ```

### Changing Visual Style

**Color Palette** (index.html line 33-42):
```javascript
--accent-blue: #0A84FF    // photo
--accent-green: #30D158   // person
--accent-orange: #FF9F0A  // place
--accent-pink: #FF375F    // time
--accent-purple: #BF5AF2  // tag
--accent-teal: #64D2FF    // file
```

**Node Sizes** (line 987-994):
```javascript
sizeMap = { photo: 10, person: 7, place: 6, time: 5, tag: 4, file: 6 }
```

### Modifying Force Layout

Adjust physics at line 1239:
```javascript
const iterations = 80;      // More = slower but better layout
const repulsion = 600;      // Higher = nodes push apart more
const attraction = 0.04;    // Higher = connected nodes pull closer
const damping = 0.85;       // Lower = more bouncy
```

## Design Systems

### visionOS (index.html)
- Glass morphism with backdrop-filter
- Floating panels with shadow-depth
- SF Pro Display font references
- Color system: rgba with low opacity layers

### Samsung Galaxy (promotion.html)
- One UI inspired spacing/typography
- Galaxy-specific color tokens (--samsung-blue: #1428a0)
- Noto Sans KR for Korean text
- Dark theme with gradient accents

## Known Limitations

- No backend: All data is static in HTML
- No persistence: Filter/search state lost on refresh
- Single page: No routing or state management
- CDN dependency: Requires internet for Three.js r128
- CORS: Must use local server (cannot open file:// directly)

## External Dependencies

- **Three.js r128**: https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js
- **Unsplash Images**: Used for photo node textures (24 images)
- **Google Fonts**: SF Pro Display, Noto Sans KR

## Anti-Patterns

| Wrong | Correct |
|-------|---------|
| Open `file://` directly | Use local web server (CORS) |
| Modify node array without link | Always add both nodes + links |
| Skip texture preload | `preloadTextures()` before render |
| Frequent raycasting | Use `Set` for O(1) connection lookup |
| Geometry per node | Consider instancing for large datasets |

## Browser Support

- Modern browsers with WebGL support
- Tested on Chrome/Safari/Edge
- Requires ES6+ (arrow functions, template literals, const/let)
- Backdrop-filter support needed for glass UI

---

## Verified Vibe Coding Protocol

이 프로젝트는 VVCS(Verified Vibe Coding System)를 따릅니다.

### 필수 워크플로우

**1. UI/시각 변경**
```
"think hard" 포함하여 영향 범위 분석
  ↓
수정 구현
  ↓
브라우저에서 확인
  ↓
/commit-push-pr
```

**2. 데이터/노드 구조 변경**
```
"think harder" 포함하여 설계
  ↓
nodes + links 동시 수정
  ↓
force layout 파라미터 확인
  ↓
브라우저에서 확인
```

### Think Mode 가이드

| 작업 복잡도 | Think Mode | 예시 |
|-------------|-----------|------|
| 단순 수정 | (기본) | 색상/텍스트 변경 |
| 다중 섹션 변경 | `think hard` | 새 노드 타입 추가 |
| 아키텍처 변경 | `think harder` | 물리 엔진 수정, 카메라 시스템 |

### Definition of Done (Memory Graph)

**필수 체크리스트**:
- [ ] 로컬 서버에서 정상 렌더링
- [ ] 노드/링크 데이터 일관성 (고아 노드 없음)
- [ ] 60fps 유지 (Performance 탭 확인)

### 자동 검증

VVCS Hooks가 자동으로:
- Plan-First 워크플로우 권장
- Think 모드 사용 권장

### 일간 모니터링

```bash
python3 ~/.claude/scripts/analyze-conversations.py
```

**목표 지표**:
- Fix 커밋 비율: < 15%
- Think 사용률: > 5%
