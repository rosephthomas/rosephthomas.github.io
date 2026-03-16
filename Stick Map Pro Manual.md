# Stick Map Pro — User & Developer Manual
**Version 1.0.0 | Build Date: 03-16-2026**

---

## Table of Contents

1. [What Is Stick Map Pro?](#1-what-is-stick-map-pro)
2. [Getting Started](#2-getting-started)
3. [The Interface](#3-the-interface)
4. [Modes](#4-modes)
5. [The Format Bar](#5-the-format-bar)
6. [Bulk Add Sites](#6-bulk-add-sites)
7. [Auto-Layout](#7-auto-layout)
8. [Notes](#8-notes)
9. [GPS Coordinates and KML Export](#9-gps-coordinates-and-kml-export)
10. [File Operations](#10-file-operations)
11. [Themes](#11-themes)
12. [Developer Overview — Program Architecture](#12-developer-overview--program-architecture)
13. [Data Structures](#13-data-structures)
14. [Rendering Pipeline](#14-rendering-pipeline)
15. [Interaction System](#15-interaction-system)
16. [Auto-Layout and Edge Routing Algorithms](#16-auto-layout-and-edge-routing-algorithms)
17. [The Note System](#17-the-note-system)
18. [Extending and Forking Stick Map Pro](#18-extending-and-forking-stick-map-pro)
19. [Configuration Reference](#19-configuration-reference)

---

# Part One — User Guide

---

## 1. What Is Stick Map Pro?

Stick Map Pro is a browser-based network diagramming tool designed for telecommunications technicians, fiber network designers, construction coordinators, and field engineers. It allows users to quickly build visual stick maps — schematic representations of network infrastructure — directly in a web browser with no installation, no login, and no internet connection required after the file is downloaded.

A stick map in this context is a simplified node-and-line diagram. Sites can represent any physical or logical element in a network: splice enclosures, terminals, distribution nodes, muxes, hubs, central offices, buildings, or any other infrastructure point. Connections between them represent fiber routes, conduit paths, or logical links and can be labeled with footage, fiber counts, cable IDs, or any other annotation the user finds useful.

Stick Map Pro is delivered as a single self-contained `.html` file. Everything — the interface, the drawing engine, the export logic — lives in that one file. It runs on desktop browsers and Android Chrome equally well.

---

## 2. Getting Started

Open `StickMapPro.html` in any modern browser. No installation required.

**Recommended browsers:**
- Google Chrome (desktop or Android) — full feature support including Save As dialog on export
- Microsoft Edge — full support
- Firefox — supported, file picker may fall back to auto-download
- Safari / iOS — basic support; file picker behavior may vary

**Basic workflow:**
1. Select **Add Site** and click the canvas to place nodes
2. Select **Connect** and click pairs of nodes to draw links
3. Label links and name nodes via the **Edit** mode popup
4. Use **Auto-Layout** to tidy the diagram
5. **Export** as `.json` to save your work, or **PNG** to share it

---

## 3. The Interface

The interface is divided into four main areas:

### Branding Bar
The top strip displays the application name, version, and build date.

### File Bar
Contains file operations (Export, Load, PNG, KML), Help, theme selector.

### Format Bar
Controls the visual properties applied to newly placed sites and links — color, shape, line style, and canvas background color.

### Sidebar
The left panel contains all mode buttons and tools organized into three sections: **Place**, **Edit**, and **Tools**. The hint box at the bottom displays context-sensitive instructions for the active mode.

### Canvas
The main drawing surface. All nodes, links, and notes are drawn here using the HTML5 Canvas 2D API.

### Import Panel
A collapsible right panel for bulk-adding sites using the indented chain syntax.

---

## 4. Modes

Modes are exclusive — only one is active at a time. The active mode button is highlighted. Switching modes resets any in-progress action (connect chain, drag, etc.).

### + Add Site
Click anywhere on the canvas to place a new node. A property popup opens immediately so you can name the site, set its color, shape, and GPS coordinates. Clicking away or pressing **Escape** dismisses the popup and defaults the name to "Site."

### ↔ Connect
Click one node to begin a link, then click a second node to complete it. A link label editor opens immediately. Press **Enter** to save the label or **Escape** to skip. Clicking the same node twice cancels the connection.

### 📝 Note
Click anywhere on the canvas to place a resizable text note box. A popup opens to set the text content, background color, and text color. See Chapter 8 for details.

### ✏ Edit
Click any node, link, or note to open its property editor. All properties are editable: name, color, shape, GPS coordinates for nodes; label, color, and style for links; text and colors for notes.

### ✥ Move
Drag any node or note to reposition it. In Move mode, notes display a triangular resize handle in their bottom-right corner — drag it to resize the box. Node drag clears auto-layout routing data since positions are no longer guaranteed clean.

### ✕ Delete
Click any node, link, or note to remove it. Deleting a node also removes all links connected to it.

---

## 5. The Format Bar

The Format Bar sets defaults applied to newly created objects. It does not retroactively change existing objects — use Edit mode for that.

| Control | Applies To | Description |
|---|---|---|
| Node Color | New sites | Fill color of placed nodes |
| Node Shape | New sites | Circle, Square, Diamond, or Hub |
| Link Color | New links | Stroke color of drawn connections |
| Link Style | New links | Solid, Dashed, or Dotted |
| BG | Canvas | Background color, updates live |

**Node Shapes:**
- **● Circle** — General node, splice point, generic site
- **■ Square** — Terminal, distribution point
- **◆ Diamond** — Splice enclosure, passive node
- **⌂ Hub** — Headend, hub, central office

---

## 6. Bulk Add Sites

The **Bulk Add** panel allows rapid entry of multiple sites and their connections using a simple text syntax. Open it from the sidebar.

### Syntax

**Flat chain** — comma-separated names on one line are placed and connected left to right:
```
HUB-1, Node-A, Node-B, Node-C
```

**Floating node** — a name on its own line with no indentation is placed unconnected:
```
Term-1
```

**Indented tree** — use Tab characters to indent children under a parent. Each tab level = one level of depth:
```
HUB-1
    Node-A, Term-1, Term-2
    Node-B
        Term-3
        Term-4
    Node-C
```
This produces: HUB-1 connected to Node-A, Node-B, and Node-C. Node-A chains to Term-1 and Term-2. Node-B connects to Term-3 and Term-4.

**Name reuse** — if a name already exists on the canvas it is reused rather than duplicated. This allows chains to branch from existing nodes across multiple Bulk Add operations.

**Tab key behavior** — pressing Tab inside the text area inserts a real tab character rather than moving focus to the next control.

After adding, use **Auto-Layout** to arrange the nodes into a clean hierarchy, or use **Move** to position them manually.

---

## 7. Auto-Layout

Auto-Layout automatically arranges all nodes into one of two patterns depending on the topology:

**Tree Layout** — used for most networks. The algorithm:
1. Identifies the root node as the highest-degree node (most connections)
2. Runs a breadth-first search (BFS) from the root to assign each node a level
3. Distributes nodes evenly across horizontal tiers, one tier per BFS level
4. Runs a conflict resolution pass to displace any node that would appear to sit on a line it is not connected to (up to 40 iterations)
5. Computes hop arcs for any remaining line-line crossings

**Ring Layout** — used when every node has exactly 2 connections and the graph is fully connected. Nodes are arranged in a circle.

**Unconnected nodes** are placed along the bottom edge of the canvas.

After layout, use **Fit View** to center everything in the viewport.

---

## 8. Notes

Notes are free-floating text boxes that can be placed anywhere on the canvas.

**Placing a note:** Select **Note** mode and click the canvas. A popup opens for text entry, background color, and text color.

**Text behavior:** Text wraps automatically at the box boundary. Hard line breaks (Enter in the text area) are respected. The box height auto-grows to fit the full content when saved.

**Resizing:** Switch to **Move** mode. A triangular handle appears in the bottom-right corner of each note. Drag it to resize — text reflows to the new width and the height adjusts automatically.

**Moving:** In Move mode, drag the body of the note to reposition it.

**Editing:** In Edit mode, click the note to reopen the property editor.

**Deleting:** In Delete mode, click the note to remove it.

Notes are saved with the `.json` export and rendered in PNG exports.

---

## 9. GPS Coordinates and KML Export

Individual nodes can store GPS coordinates for export to geographic mapping tools.

### Adding Coordinates

In **Edit** mode, click any node. The **Coords** field accepts a single comma-separated decimal degree string:

```
35.12345, -80.12345
```

- Latitude first, longitude second
- Negative longitude = West
- Negative latitude = South
- Use decimal degrees only — DMS (degrees/minutes/seconds) format is not supported
- 5 decimal places recommended for field-grade accuracy (~1 meter resolution)

### KML Export

Click **KML** in the file bar. Sites with GPS coordinates are exported as KML **Placemarks** with colored pins matching the node color. Connections between coordinated nodes are exported as **LineStrings**.

Sites missing coordinates are listed before export with the option to proceed or cancel. The resulting `.kml` file can be opened directly in Google Earth Pro, QGIS, or any GIS application that supports KML 2.2.

---

## 10. File Operations

| Button | Function |
|---|---|
| **💾 Export** | Saves the full map as a `.json` file including nodes, links, notes, theme, pan position, and background color. On Chrome desktop a Save As dialog appears. On Android Chrome the file saves to Downloads. |
| **📂 Load** | Opens a file picker to load a previously saved `.json` map. Fully restores all state. |
| **🖼 PNG** | Exports the canvas as a PNG image. Prompts whether to include the background or export with a transparent background. |
| **🌍 KML** | Exports GPS-coordinated nodes and links as a KML file for Google Earth Pro. |
| **🗑 Clear** | Wipes all nodes, links, and notes from the canvas after a confirmation prompt. |

---

## 11. Themes

Three themes are available from the Theme selector in the file bar:

| Theme | Description |
|---|---|
| **Minesweeper** | Classic Win95 grey with raised button borders, Courier New font, checkerboard tile canvas background |
| **Dark** | Deep navy UI with glowing accents |
| **Light** | Clean white UI with subtle grey grid |

The active theme is saved with the `.json` export and restored on load. The BG color picker adjusts the canvas background independently of the theme.

---

# Part Two — Developer & Engineer Reference

---

## 12. Developer Overview — Program Architecture

Stick Map Pro is a single-file HTML application. There are no build tools, no bundlers, no external dependencies, and no server-side components. The entire application — approximately 1,000 lines — lives in one `.html` file structured as:

```
<head>
  <style>         CSS — layout, theming via CSS custom properties
</head>
<body>
  HTML structure  — bars, sidebar, canvas, panels, popups
  <script>        JavaScript — all application logic
</body>
```

### Technology Stack
- **HTML5 Canvas 2D API** — all drawing (nodes, links, notes, grid, backgrounds)
- **Vanilla JavaScript (ES5-compatible)** — no frameworks, no transpilers
- **CSS custom properties** — full UI re-theming at runtime
- **File System Access API** (with fallback) — Save As dialog on supported browsers
- **Blob URLs** — PNG export, KML export, Help page generation

### Design Philosophy
- Zero dependencies — the file is self-contained and portable
- No persistent state between sessions except what the user explicitly exports
- All rendering is immediate-mode: the canvas is cleared and redrawn from scratch on every frame
- `var` is used throughout for maximum browser compatibility

---

## 13. Data Structures

All application state lives in three top-level arrays and a handful of scalar variables.

### `nodes` — Array of node objects
```javascript
{
  id:       Number,    // unique id from uid()
  x:        Number,    // world x coordinate
  y:        Number,    // world y coordinate
  name:     String,    // display label
  color:    String,    // hex color e.g. '#00d4ff'
  shape:    String,    // 'circle' | 'square' | 'diamond' | 'house'
  lat:      Number,    // optional decimal degrees latitude
  lon:      Number,    // optional decimal degrees longitude
}
```

### `links` — Array of link objects
```javascript
{
  id:        Number,   // unique id
  a:         Number,   // node id of endpoint A
  b:         Number,   // node id of endpoint B
  label:     String,   // optional mid-line annotation
  color:     String,   // hex color
  style:     String,   // 'solid' | 'dashed' | 'dotted'
  waypoints: Array,    // [{x,y}] — computed by auto-layout, not persisted meaningfully
  hops:      Array,    // [{seg, t, x, y}] — crossing arc data, computed at layout time
}
```

### `notes` — Array of note objects
```javascript
{
  id:        Number,   // unique id
  x:         Number,   // world x (top-left corner)
  y:         Number,   // world y (top-left corner)
  w:         Number,   // width in pixels
  h:         Number,   // height in pixels (auto-computed from content)
  text:      String,   // raw text content including newlines
  color:     String,   // background hex color
  textColor: String,   // text hex color
}
```

### Global scalars
```javascript
var panX, panY;          // canvas pan offset (world origin shift)
var currentTheme;        // 'minesweeper' | 'dark' | 'light'
var bgColor;             // canvas background hex color
var mode;                // active interaction mode string
var idCounter;           // monotonic id generator
```

### Serialization
`exportJSON` serializes `{nodes, links, notes, idCounter, bgColor, theme, panX, panY}`. The `waypoints` and `hops` arrays on links are included but are regenerated on next Auto-Layout, so they are effectively transient.

---

## 14. Rendering Pipeline

All rendering flows through `drawScene(tc, W, H, transparent, ox, oy)`.

| Parameter | Purpose |
|---|---|
| `tc` | Target canvas 2D context |
| `W, H` | Canvas dimensions |
| `transparent` | If true, skip background fill (used for transparent PNG export) |
| `ox, oy` | Pan offset — translate applied to all world-space objects |

### Draw Order
1. **Background** — solid fill, then grid (line grid or minesweeper tiles depending on theme)
2. `tc.save()` + `tc.translate(ox, oy)` — enter world space
3. **Notes** — drawn first so they appear behind links and nodes
4. **Links** — each link iterated, segments drawn via `drawSegmentWithHops()`
5. **Connect highlight** — dashed circle around the pending connect-start node
6. **Nodes** — drawn last so they always appear on top
7. `tc.restore()` — exit world space

### `drawSegmentWithHops(tc, p0, p1, segIdx, hops, lineColor, lineWidth, lineDash)`
Draws one segment of a polyline, interrupting it with semicircular hop arcs at any crossing points assigned to this segment. Uses `tc.save()` / `tc.translate()` / `tc.rotate()` to align the segment along the local X axis, making arc geometry trivial (`arc(hopX, 0, 9, Math.PI, 0, true)`).

### `draw()`
The main draw call. Clears the canvas and calls `drawScene` with the current pan offset and the main context.

### `wrapText(tc, text, maxW)`
Computes wrapped line array for a given string and maximum pixel width. Splits on newlines first, then word-wraps each paragraph. Returns `String[]`.

### `noteHeight(tc, note)`
Computes the auto-height for a note given its current text and width. Returns `max(40, lines * NOTE_LINE + NOTE_PAD * 2 + 8)`.

---

## 15. Interaction System

### Coordinate System
World coordinates are independent of pan. Screen coordinates (canvas pixels) are converted to world coordinates via `toWorld(cx, cy)` which applies `{ x: cx - panX, y: cy - panY }`.

All hit-testing is performed in world space.

### Hit Testing
- `nodeAt(wx, wy)` — returns the topmost node within 22px of the world point, iterating in reverse order (last placed = topmost)
- `linkNear(wx, wy)` — returns the first link whose nearest-point-on-segment is within 10px
- `noteAt(wx, wy)` — returns the topmost note whose bounding box contains the world point
- `noteResizeHandleAt(wx, wy)` — returns the note whose bottom-right handle is within `NOTE_HANDLE` px

### Event Flow
Mouse and touch events are handled by separate listeners but converge on `handleTap(cx, cy, clientX, clientY)` for discrete actions. Drag/resize are handled directly in `mousemove` / `touchmove` without going through `handleTap`.

**Mouse:** `mousedown` → `handleTap` or begins drag/resize. `mousemove` → updates drag/resize or hover state. `mouseup` → clears drag state.

**Touch:** `touchstart` → same branching as mousedown. `touchmove` → drag/resize only. `touchend` → clears state. A 300ms double-tap suppression prevents accidental double placement on mobile.

### Mode System
`setMode(m)` sets the global `mode` string, toggles `.active` class on the appropriate sidebar button, updates the hint text, resets `connectStart`, `dragging`, and `resizingNote`, and closes any open popups.

---

## 16. Auto-Layout and Edge Routing Algorithms

### Tree Layout
1. Build adjacency map from `links`
2. Find root: node with highest degree (most links)
3. BFS from root — assigns `level[id]` to each reachable node
4. Group nodes by level into `levels[]`
5. Distribute each level evenly across canvas width within padding
6. Park unvisited (disconnected) nodes along bottom edge

### Ring Layout
Triggered when all nodes have exactly degree 2 and the BFS from the first node reaches all nodes. Nodes are evenly spaced around a circle of radius `min(W,H) * 0.38`.

### `displaceConflictingNodes()`
Post-layout pass. For each node, checks every link it is not connected to. If the node's center is within `36px` of the line segment (foot-point distance), the node is nudged perpendicularly away by the required clearance plus 1px. Repeats up to 40 iterations until no movement occurs.

### `computeWaypoints()`
Finds all pairs of non-adjacent links that geometrically intersect. For each crossing, the link with the lower trunk score (sum of endpoint degrees) receives a hop record `{seg, t, x, y}` indicating where the arc should appear on its segment. `segIntersect()` implements the standard line-segment intersection formula.

### `trunkScore(l)`
Returns `degreeOf(l.a) + degreeOf(l.b)`. Used to decide which link hops over a crossing — higher score = more trunk-like = stays flat.

---

## 17. The Note System

Notes are canvas-drawn objects stored in the `notes[]` array. They are rendered as filled rectangles with word-wrapped text, a drop shadow, and a border.

### Resize Handle
Rendered only in Move mode as a filled triangle in the bottom-right corner. Hit-tested via `noteResizeHandleAt()`. On drag, `resizingNote.w` and `resizingNote.h` are updated directly. Minimum size is `80 × 40px`.

### Auto-Height
When a note is saved from the popup, `noteHeight(ctx, note)` is called to compute the required height for all wrapped lines. The `note.h` property is set to this value. If the user manually resizes the box taller than the auto-height, extra whitespace appears below the text — the text does not stretch.

### Text Rendering
`wrapText()` is called during every draw pass to dynamically wrap text at the current box width. This means text reflows live whenever the box is resized.

---

## 18. Extending and Forking Stick Map Pro

Stick Map Pro is intentionally simple and self-contained. Adding new features or forking it for specialized use is straightforward.

### Adding a New Node Shape

1. Add an `<option>` to `#node-shape`, `#pe-shape`, and `#import-shape` selects in the HTML
2. Add an `else if` block in `drawScene()` in the node drawing loop:
```javascript
else if(n.shape === 'yourshape') {
  tc.moveTo(...); // define your path points relative to center (0,0)
  tc.closePath();
}
```
3. No other changes needed — the shape will be saved, loaded, and exported automatically

### Adding a New Theme

Add an entry to the `THEMES` object:
```javascript
var THEMES = {
  yourtheme: {
    bg:'#...', ui:'#...', uiBorder:'#...', uiText:'#...',
    btn:'#...', btnBorder:'#...', btnActive:'#...', btnActiveText:'#...',
    input:'#...', inputBorder:'#...', inputText:'#...',
    label:'#...', barLabel:'#...', hint:'#...', sectionTitle:'#...'
  }
};
```
Add a corresponding `<option>` to `#theme-select`. The theme system handles all CSS variable injection and element styling automatically via `applyTheme()`.

### Adding a New Data Field to Nodes

1. Add the HTML input to `#pe-node-fields` in the popup editor
2. In `openPopup()`, populate the field from `obj.yourField`
3. In `savePopup()`, read and assign `obj.yourField`
4. The field will serialize automatically since `exportJSON` serializes the full `nodes[]` array

### Adding a New Mode

1. Add a button to the sidebar: `<button id="btn-yourmode" onclick="setMode('yourmode')">...</button>`
2. Add a hint string in `setMode()`: `yourmode: 'Description of mode.'`
3. Add the mode string to the active-class toggle array in `setMode()` and `applyTheme()`
4. Add handling in `handleTap()` and/or `mousedown`/`touchstart` as needed

### Adding a New Export Format

Add a function following the pattern of `exportKML()`:
```javascript
function exportXYZ(){
  // Build your string/blob from nodes[], links[], notes[]
  var blob = new Blob([content], {type:'your/mimetype'});
  var a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = 'stickmap.xyz';
  a.click();
  setTimeout(function(){ URL.revokeObjectURL(a.href); }, 1000);
}
```
Add a button to `#bar-file` calling it.

### Integrating with a Backend

To persist maps server-side, replace or augment `exportJSON()` and `importJSON()` with `fetch()` calls:
```javascript
async function saveToServer(){
  var data = JSON.stringify({nodes,links,notes,idCounter,bgColor,theme:currentTheme,panX,panY});
  await fetch('/api/maps/save', {method:'POST', body:data, headers:{'Content-Type':'application/json'}});
}
```
The data model is flat JSON — no transformation needed.

### Splitting Into Multiple Files

If you want to modularize the codebase, the natural split points are:

| Module | Contents |
|---|---|
| `themes.js` | `THEMES` object, `applyTheme()` |
| `draw.js` | `drawScene()`, `drawSegmentWithHops()`, `drawNotes()`, `blendGrid()` |
| `layout.js` | `autoLayout()`, `displaceConflictingNodes()`, `computeWaypoints()`, `segIntersect()`, `trunkScore()` |
| `interaction.js` | All event listeners, `handleTap()`, `setMode()`, `nodeAt()`, `linkNear()`, `noteAt()` |
| `io.js` | `exportJSON()`, `importJSON()`, `exportPNG()`, `exportKML()`, `applyMap()` |
| `notes.js` | `wrapText()`, `noteHeight()`, note popup functions |
| `import.js` | `importSites()`, bulk add chain parser |

---

## 19. Configuration Reference

The following constants at the top of the script control core behavior. Adjust them to tune the application without touching the logic.

| Constant | Default | Description |
|---|---|---|
| `NOTE_PAD` | `8` | Inner padding (px) of note boxes |
| `NOTE_FONT` | `13` | Font size (px) for note text |
| `NOTE_LINE` | `18` | Line height (px) for note text wrapping |
| `NOTE_HANDLE` | `10` | Size (px) of the note resize handle hit area |

The following are inline values in the logic worth knowing if tuning:

| Location | Value | Description |
|---|---|---|
| `nodeAt()` | `22` | Node hit radius in px |
| `linkNear()` | `10` | Link click tolerance in px |
| `clampPan()` | `2000` | Maximum pan distance from origin in px |
| `displaceConflictingNodes()` | `36` | Minimum clearance between node and non-connected line in px |
| `displaceConflictingNodes()` | `40` | Maximum relaxation iterations |
| `drawSegmentWithHops()` | `9` | Hop arc radius in px |
| `autoLayout()` ring | `0.38` | Ring layout radius as fraction of `min(W,H)` |
| `autoLayout()` tree | `80` | Padding (px) from canvas edges in tree layout |

---

*Stick Map Pro v1.0.0 — Built for the people who design and build the networks.*
