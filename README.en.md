# FlowBoard · Single-file Collaborative Whiteboard

> A **zero-dependency, single-file, double-click-to-run** Miro-style infinite-canvas whiteboard. Pure HTML + CSS + JavaScript (Canvas 2D) — no build step, no third-party libraries, works on both mobile and desktop.

[中文文档 → README.md](./README.md)

---

## ✨ Features

| Category | Capability |
| --- | --- |
| **Canvas** | Infinite canvas, pan (drag / Space / hand tool), wheel & pinch zoom, adaptive grid, minimap navigation |
| **Sticky notes** | One-tap add, double-click edit, drag to move, 8-direction resize handles, 6 colors |
| **Drawing** | Freehand pen (6 colors + custom picker + width), eraser (tap / drag to erase) |
| **Shapes** | Rectangle / ellipse tools with stroke↔fill toggle, sharing the same select/resize system as notes |
| **Connectors** | Free linking between notes with auto edge-anchoring + arrows; **orthogonal routing that bends around shape obstacles** (A* over a sparse grid with a turn penalty) |
| **Undo / Redo** | Full history stack (120 steps), `Ctrl+Z` / `Ctrl+Shift+Z`, buttons reflect disabled state |
| **Export / Import** | Export **PNG** (2× raster), **SVG** (vector), **canonical JSON** (quantized coords · fixed key order · sorted by id · git-diffable · hashable); import JSON to restore a project |
| **Persistence** | Auto-save to browser `localStorage` (400ms debounce); survives refresh |
| **Verifiability** | Built-in `window.__t.__selftest()`: checks transform round-trip, serialization idempotence, export→import identity, and route clearance — runnable headless |

---

## 🚀 Quick Start

- **Option 1 (simplest):** Double-click `index.html` to open it in any browser.
- **Option 2 (online):** Enable GitHub Pages, then visit `https://edgarzwj.github.io/flowboard/`.
- All data stays in your local browser by default — **nothing is uploaded to any server**.

---

## ⌨️ Keyboard Shortcuts

| Key | Action | Key | Action |
| --- | --- | --- | --- |
| `V` | Select | `H` | Hand / pan |
| `N` | Sticky note | `P` | Pen |
| `E` | Eraser | `C` | Connector |
| `R` | Rectangle | `O` | Ellipse |
| `Space` | Temporary pan | `Delete` / `Backspace` | Delete selection |
| `Ctrl+Z` | Undo | `Ctrl+Shift+Z` / `Ctrl+Y` | Redo |
| Double-click | Edit note | Right-click / long-press | Delete object |

**Touch:** one finger drags to pan, two fingers pinch to zoom, long-press to delete, double-tap a note to edit.

---

## 📁 Project Structure

```
flowboard/
├── index.html     # All code (HTML+CSS+JS in one file)
├── README.md      # Chinese documentation
├── README.en.md   # English documentation (this file)
├── LICENSE        # MIT License
└── .gitignore     # Ignore tokens and local artifacts
```

---

## 💡 Design: What We Borrowed & What We Added

Before writing code we studied three families of mature open-source tools, absorbed their strengths, then made trade-offs for a "single-file, portable, mobile-first" board:

**Borrowed (common capabilities)**
- **Excalidraw** (MIT, hand-drawn): PNG / SVG / JSON export, undo/redo, local-first workflow.
- **tldraw** (Apache-2.0): smart connector anchors, resize handles, minimap navigation.
- **tiny_whiteboard** (vanilla, ~293★) and **drawme / Khushal-Me**: zero-dependency note + pen + undo/redo + download implementations proving a full whiteboard can live in pure frontend.

**Innovated (our trade-offs & enhancements)**
1. **Truly single-file, zero-dependency:** Canvas rendering, infinite-coordinate transforms, unified Pointer Events input, and localStorage autosave all live in one `index.html` — copy it anywhere, run offline, no build, no CDN.
2. **Mobile-first:** Unified Pointer Events cover mouse / touch / pen; pinch-zoom, long-press delete, and double-tap edit are built for touch; the top bar auto-scrolls on narrow screens.
3. **Unified object model for shapes + notes:** Rectangle / ellipse share the same "select → 8-handle resize → recolor → delete" logic as notes, minimizing mental overhead.
4. **All-in-one export menu:** PNG / SVG / JSON export plus JSON import live behind one dropdown; SVG is pure vector (text re-flowed line-by-line) for easy post-editing.
5. **Orthogonal obstacle-avoiding connectors + canonical, diff-stable JSON:** see "Why these three things" below.

---

## 🧭 Why These Three Things: A Board You Can Put in Git and Trust

Mainstream whiteboards treat the file as an opaque blob. FlowBoard's differentiation comes from one judgment: **a board file should be version-controllable like source code, and its geometric correctness should be verifiable.** We did three things mainstream lightweight tools do not prioritize:

### 1. Canonical, diff-stable serialization (determinism)
Peer tools export JSON with floating-point noise, unstable key order, and unstable object ordering — so every save, even with no real change, turns a git diff red and you cannot review "what actually changed" in a PR. FlowBoard's export quantizes coordinates (`GRID_QUANTUM = 0.5` world units), emits keys in a fixed order, and sorts objects by `id` — **identical boards produce byte-identical files**, yielding meaningful diffs. It also ships a dependency-free content hash (`contentHash()`, FNV-1a 32-bit) for one-line change detection.

### 2. Orthogonal, obstacle-avoiding connectors (algorithm)
Most lightweight boards draw connectors as straight lines that plow through shapes. FlowBoard routes connectors via A* over a **sparse grid** built from obstacle-edge coordinates plus the endpoints, with a turn penalty that favors fewer bends, so the polyline bends around shapes. When obstacle count exceeds a threshold (`ROUTE_MAX_OBSTACLES = 24`) it falls back to a simple elbow route (a known-ceiling fallback that won't blow up the grid). This is a "underlying principle" capability, fully deterministic and zero-dependency.

### 3. Verifiable correctness (self-test)
The page embeds `window.__t.__selftest()` which asserts: screen↔world transform round-trips, canonical serialization is idempotent, export→import is identical at the canonical level, and routing never crosses an obstacle's interior. It runs in a **browser-less Node environment** (this project verifies it headless with a DOM/Canvas stub), reflecting the engineering discipline of "leave at least one runnable check per non-trivial piece of logic."

> The code follows the `code-no-slop` standard from [deaify](https://github.com/Edgarzwj/deaify): zero dependencies, named constants (no bare magic numbers), no narration comments, and a runnable self-test.

---

## 🛠 Tech Stack

- **Language:** Vanilla JavaScript (ES2020), strict mode
- **Rendering:** Canvas 2D (device-pixel-ratio aware)
- **Input:** Pointer Events (mouse / touch / pen unified)
- **Storage:** `localStorage`
- **Export:** Offscreen Canvas → PNG; hand-written SVG serialization (orthogonal polylines) → vector; **canonical JSON** (quantized + fixed key order + sorted by id) → project file / diffable
- **Algorithm:** orthogonal connector routing (A* over a sparse grid with a turn penalty), pure-function and deterministic
- **Self-test:** `window.__t.__selftest()`, runnable headless under Node
- **Build:** None (pure static file)

---

## 📜 License

[MIT License](./LICENSE) © 2026 Edgarzwj
