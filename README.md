# Build Viewer

A browser-based 3D viewer for Rhino `.3dm` files. Open a model, inspect individual parts, and read precise dimensions — no install required. Works on desktop and iPhone.

## What it does

- **Open .3dm files** via the file picker or drag-and-drop
- **Browse objects** in a collapsible sidebar — parts with the same name are grouped with a combined volume total
- **Select a part** by clicking it in the viewport or sidebar — it highlights in orange and all others fade
- **Read dimensions** (L × W × T) computed from the part's oriented bounding box (OBB via PCA), shown in decimal inches, fractional inches, and feet/inches
- **See volume** in in³ and ft³ for solid mesh parts
- **3D dimension annotations** draw directly in the viewport when a part is selected
- **Drag parts** to reposition them in the scene; Reset Positions restores originals
- **Load multiple files** — each appears as a separate entry in the sidebar; clicking a header switches the active model

## Isolated view

Double-click (or double-tap on mobile) any part in the viewport to open it in a full-screen isolated view. You can also double-click a part in the sidebar list to open it directly.

Inside the isolated view:
- **Click any edge** to measure it — a confirmation prompt appears before committing the measurement
- **Straight edges** show total length (L) by merging all collinear tessellation segments
- **Curved edges** show radius (R) when exact BREP data is available from the file
- **Dimensions display** in decimal inches, fractional inches, and feet/inches
- The part name appears in the header and as an overlay label in the 3D viewport

## Tech stack

Single-file HTML/CSS/JS — no build step.

| Layer | Choice |
|-------|--------|
| 3D engine | [Three.js](https://threejs.org/) v0.168.0 (ES module via CDN) |
| Rhino parser | [Rhino3dmLoader](https://github.com/nicktindall/three-rhino3dm-loader) + rhino3dm WASM v8.6.1 |
| Dimension labels | Three.js CSS2DRenderer |
| OBB algorithm | Jacobi eigendecomposition of vertex covariance matrix |
| Arc radius | Circumradius via exact BREP edge sampling (rhino3dm WASM) |

## Running locally

Must be served over HTTP (not opened as a local file) because the WASM loader requires it.

```bash
# Any static server works — e.g.:
npx serve .
# then visit http://localhost:3000
```

## Controls

### Desktop

| Action | Input |
|--------|-------|
| Orbit | Left drag |
| Pan | Right drag |
| Zoom | Scroll wheel |
| Select part | Click |
| Open isolated view | Double-click part |
| Open isolated view (sidebar) | Double-click sidebar row |
| Deselect | Click selected part again, or click background |

### Mobile / iPhone

| Action | Input |
|--------|-------|
| Orbit | One-finger drag |
| Pan | Two-finger drag |
| Zoom | Pinch |
| Select part | Tap |
| Open isolated view | Double-tap part |
| Open isolated view (sidebar) | Double-tap sidebar row |

## Project structure

```
build_viewer/
├── index.html          # entire app — markup, styles, and logic
├── manifest.json       # PWA manifest
└── 3dm_viewer_icons/   # PWA app icons
```

## Notes

- Rhino is Z-up; the loader rotates models −90° on X to align with Three.js's Y-up world
- Layer colors from the .3dm file are preserved on each object
- Dimensions use the part's own principal axes (OBB), not the world grid — so a rotated part still shows correct L/W/T
- Volume is computed via the divergence theorem (signed tetrahedral sum) over the mesh triangles
- Edge measurement uses exact BREP geometry when available; falls back to merged tessellation segments for straight edges
- iOS Safari: `touch-action: none` on the canvas prevents page-scroll interference; safe-area insets keep UI clear of the notch and home indicator
