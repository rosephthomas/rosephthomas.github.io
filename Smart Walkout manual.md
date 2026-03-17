# Smart Walkout – User & Developer Manual

---

## Chapter 1 – Use Intention

### 1.1 Purpose

**Smart Walkout** is a lightweight, browser-based tool for:

- Capturing GPS-based station points in the field.
- Visually connecting those points to represent routes or segments.
- Estimating segment lengths (in feet) using geographic distance.
- Exporting the final layout to a KML file for use in GIS tools like Google Earth Pro.

It is designed to:

- Run entirely in the browser (no backend).
- Work both on laptops and mobile devices with GPS.
- Persist sessions locally using the browser’s `localStorage`.
- Be simple for field users, but transparent enough for developers to extend.

### 1.2 Typical Use Cases

Common scenarios include:

- Construction or engineering **walkouts** for planned cable or conduit routes.
- High-level **site surveys** where full CAD/GIS is overkill.
- Quick **as-built** sketches where approximate positions and lengths are acceptable.
- Any workflow that needs:
  - A list of station points with coordinates;
  - Line segments between those points with approximate lengths;
  - Simple export to KML.

---

## Chapter 2 – How to Use Smart Walkout

### 2.1 Requirements & Setup

- A modern browser (Chrome, Edge, Firefox, Safari).
- GPS access if you want to record true positions:
  - Desktop: may be coarse, based on Wi‑Fi/IP.
  - Mobile: typically uses GPS.
- No installation:
  - Save the HTML file locally.
  - Open it in the browser.

All data is stored in the device’s **localStorage** for that browser. It does not sync between devices or browsers.

---

### 2.2 Project ID / Name

At the top of the page:

1. Locate: **Project ID / Name**.
2. Enter a descriptive label, e.g.:
   - `FTTH – Town Center Job 1234`
   - `Ring A – North Loop`

This project name is used:

- As the KML `<Document>` name.
- As the master folder name in the KML file.
- In the exported filename, formatted as:  
  `Project_Name_MM-DD-YYYY.kml`  
  (special characters are sanitized for the filename).

---

### 2.3 Recording GPS Points

1. In the **Record Current Location** card:
   - Enter a **Location Name** (e.g. `Station 1`, `Node A`, `Pole 15`).

2. Click **Record Current GPS**:
   - The browser will prompt you for **location access**. Click *Allow*.
   - Smart Walkout uses `navigator.geolocation` to read your current position.

3. After a successful read:
   - A new point is created with:
     - Your entered name.
     - Latitude and longitude from the device.
   - The status line shows something like:  
     `Recorded: Station 1 (Lat: 40.123456, Lon: -74.987654)`.

4. The new point appears:
   - As a row in **Points (Editable)**.
   - As a yellow node on the **Canvas View**.

---

### 2.4 Editing Points in the Table

In the **Points (Editable)** card:

- **Name column**:
  - Click the input to rename a point.
- **Latitude / Longitude columns**:
  - You can manually adjust coordinates (to correct GPS or use known survey values).
  - When lat/lon is changed:
    - Canvas layout is recomputed.
    - All link distances associated with that point are recalculated.
    - The Links table and canvas labels update.

To **delete a point**:

- Click **Del** in the Actions column.
- All links that reference that point are removed at the same time.

---

### 2.5 Connecting Points on the Canvas

In the **Canvas View (Tap to Connect Points)** card:

1. Each point is drawn as a **yellow circle** with its name next to it.
2. To create a connection (link):
   - Click one point (this becomes the **From** point).
   - Then click another point (the **To** point).

3. A **New Connection** modal appears:
   - Shows:
     - `From: <Point Name>`
     - `To: <Point Name>`
   - Optionally enter a **Connection Name** (e.g. `Span 1`, `Segment A`).
   - Click **Confirm** to create the link, or **Cancel** to discard.

4. When confirmed:
   - A line is drawn between the two points on the canvas.
   - The line label shows:
     - The connection name (if set).
     - The distance in feet (approximate).
   - A new row is added to **Links (Created via Canvas)**.

To **cancel a pending selection**:

- If you click a point as the first selection, then click the same point again, the selection is cleared.

---

### 2.6 Reviewing & Editing Links

In the **Links (Created via Canvas)** card:

- Each row displays:
  - Index number.
  - **Name** (editable).
  - **From** point name.
  - **To** point name.
  - **Distance (ft)**.
- You can:
  - Rename a link directly in the **Name** input.
  - Delete a link using **Del** in the Actions column.

Changing the link’s name updates the label on the canvas.

---

### 2.7 Autosave & Session Management

At the top-right of the screen:

- **Autosave Indicator**:
  - **Red** dot + text: `Unsaved changes`.
  - **Green** dot + text: `Saved (last auto-save: YYYY-MM-DD HH:MM:SS)`.

How it works:

- Any change to project name, points, or links calls `markDirty()`.
- After about 1 second of inactivity, `saveState()` writes the current state to `localStorage`.

On page reload (same device & browser):

- The last saved state is automatically reloaded:
  - Project name.
  - All points and links.
  - Internal ID counters.

---

### 2.8 Clear Session

In the **Export & Session** card:

1. Click **Clear Session**.
2. A confirmation dialog warns that this action cannot be undone.
3. If you confirm:
   - All points and links are cleared.
   - Project name is cleared.
   - Stored state in `localStorage` is removed.
   - The UI resets to a “fresh” project.

Use Clear Session when you’re finished with a project and want to start a new one on the same device.

---

### 2.9 Exporting to KML

In the **Export & Session** card:

1. Click **Export to .kml**.
2. If there are no points, you’ll see an alert: `No points to export.`.
3. Otherwise:
   - A KML file is generated with:
     - `<Document>` named after your project.
     - A master `<Folder>` with the same project name.
     - Two subfolders:
       - `<Folder name="Stations">` — one Placemark per point.
       - `<Folder name="Lines">` — one Placemark per link.

KML structure (simplified):

- Document and Master Project folder:
  - `<Document>`
    - `<name>Project Name</name>`
    - `<Folder>`
      - `<name>Project Name</name>`
      - `<Folder name="Stations">` …points… `</Folder>`
      - `<Folder name="Lines">` …links… `</Folder>`
    - `</Folder>`
  - `</Document>`

Stations (points):

- Each point is a `<Placemark>` with:
  - `<name>` = point name.
  - `<Point><coordinates>lon,lat,0</coordinates></Point>`.

Lines (links):

- Each link is a `<Placemark>` with:
  - `<name>` = link name (or `Link N` if unnamed).
  - Optional `<description>` = `Distance: XXXX ft`.
  - `<LineString>` from the “from” point coordinates to the “to” point coordinates.

Filename:

- `Project_Name_MM-DD-YYYY.kml`  
  (where `Project_Name` is a sanitized version of the project name and the date stamp uses your current date).

Open this file in **Google Earth Pro** or other KML-capable tools to visualize your walkout.

---

## Chapter 3 – Code Structure & Architecture

This chapter helps future developers understand and modify the code.

### 3.1 Technology Overview

- Plain **HTML**, **CSS**, and **JavaScript**.
- No external libraries or frameworks.
- All logic is inside a single `<script>` tag in the HTML file.
- Runs entirely in the browser:
  - Uses `navigator.geolocation` for GPS.
  - Uses an HTML `<canvas>` for visualization.
  - Uses `localStorage` for persistence.

There is no server-side component and no network I/O.

---

### 3.2 Initialization Flow

On page load:

1. `loadState()`:
   - Attempts to load saved JSON from `localStorage` under the key `gps_canvas_links_state_v1`.
   - If valid, reconstructs:
     - `projectName`
     - `points` array
     - `links` array
     - `nextPointId` and `nextLinkId`
     - `lastSavedAt`

2. `renderPoints()`:
   - Builds the **Points** table rows from the `points` array.

3. `renderLinks()`:
   - Builds the **Links** table rows from the `links` array.

4. `resizeCanvas()`:
   - Sizes the canvas to fit its container and the viewport.
   - Calls `layoutPointsToCanvas()` and `drawCanvas()`.

5. Sets `isDirty = false` and calls `updateAutosaveIndicator()` to show “Saved”.

---

## Chapter 4 – Data Model

### 4.1 Points

Each point is stored as:

    {
      id: Number,      // unique identifier
      name: String,    // user-friendly label
      lat: Number,     // latitude in decimal degrees
      lon: Number,     // longitude in decimal degrees
      x: Number|null,  // canvas x coordinate (derived)
      y: Number|null   // canvas y coordinate (derived)
    }

- `id` is used to link `links` back to specific points.
- `x` and `y` are computed from `lat`/`lon` and not persisted.

Only `id`, `name`, `lat`, and `lon` are saved to `localStorage`.

---

### 4.2 Links

Each link (connection between two points) is stored as:

    {
      id: Number,          // unique identifier
      name: String,        // optional user label
      fromId: Number,      // id of start point
      toId: Number,        // id of end point
      distanceFeet: Number // cached distance in feet
    }

- `fromId` and `toId` reference point IDs.
- `distanceFeet` is calculated via Haversine distance and cached.

---

### 4.3 Persisted State

State saved to `localStorage`:

    {
      projectName: String,
      points: [ { id, name, lat, lon } ],
      links: [ { id, name, fromId, toId, distanceFeet } ],
      nextPointId: Number,
      nextLinkId: Number,
      lastSavedAt: Number // timestamp (ms since epoch)
    }

The storage key is `gps_canvas_links_state_v1`.

---

## Chapter 5 – Functional Modules

### 5.1 Autosave & Dirty State

Important globals:

- `isDirty` – whether unsaved changes exist.
- `lastSavedAt` – JavaScript `Date` of last save.
- `AUTOSAVE_DELAY_MS` – delay (ms) before auto-saving after the last change.

Key functions:

- `markDirty()`:
  - Sets `isDirty = true`.
  - Updates the autosave indicator.
  - Schedules `saveState()` via `setTimeout` (debounced).

- `saveState()`:
  - Builds a serializable state object.
  - Calls `localStorage.setItem(STORAGE_KEY, JSON.stringify(state))`.
  - Sets `lastSavedAt` and `isDirty = false`.
  - Updates the autosave indicator.

If the codebase is extended with new persistent fields, both `saveState()` and `loadState()` should be updated.

---

### 5.2 Canvas Layout & Rendering

Responsibilities:

- Map geographic coordinates to a 2D canvas.
- Render points and links.
- Provide visual feedback for selections.

Core functions:

1. `layoutPointsToCanvas()`:
   - Finds min/max latitude and longitude.
   - Applies a margin (padding) on the canvas edge.
   - Rescales all points so they fit into canvas dimensions.
   - Stores pixel coordinates into each point’s `x` and `y`.

2. `drawCanvas()`:
   - Clears the canvas.
   - Draws all links:
     - A line between `fromPt` and `toPt`.
     - A label at the midpoint containing:
       - Link name.
       - Distance (feet).
   - Draws all points:
     - Yellow filled circle with black stroke.
     - Name label next to each point.
   - Highlights `pendingFromPoint` (the selected start point) with a green ring.

3. `getPointAtCanvasCoords(x, y)`:
   - Performs a radius-based hit test to see if a click is close to any point’s `(x, y)`.
   - Returns the matched point or `null`.

---

### 5.3 Geolocation Integration

The **Record Current GPS** button handler:

- Calls `navigator.geolocation.getCurrentPosition` with options:
  - `enableHighAccuracy: true`
  - `timeout: 10000`
  - `maximumAge: 0`
- On success:
  - Extracts `lat` and `lon`.
  - Uses the current `pointNameInput` value (or a fallback like `Point N`).
  - Calls `addPoint(name, lat, lon)`.

On error (user denies permission, timeout, etc.):

- Updates the status text with a message such as:  
  `Error getting location: <message>`.

---

### 5.4 Distance Computation (Haversine)

Distance between two points is computed in meters using the **Haversine formula**, then converted to feet:

- `haversineDistanceMeters(lat1, lon1, lat2, lon2)`:
  - Uses Earth radius `R = 6371000` meters.
  - Converts degrees to radians.
  - Computes great-circle distance.

- `distanceFeetBetweenPoints(p1, p2)`:
  - Calls `haversineDistanceMeters(p1.lat, p1.lon, p2.lat, p2.lon)`.
  - Multiplies meters by `3.28084` to get feet.

Every link created uses this to initialize `distanceFeet`. Whenever a point’s coordinates change, `recalcAllLinkDistances()` recomputes link distances.

---

### 5.5 KML Exporter

Within the `exportBtn` click handler:

1. Validates that there is at least one point.
2. Determines:
   - `projectName` (or a default like `Smart Walkout Project`).
   - `dateStamp` from `getDateStampForFilename()` (MM-DD-YYYY).
   - `safeProjectNameForFile`, which is a sanitized version of the project name for filenames.
3. Builds the KML as a concatenation of:
   - `kmlHeader`:
     - XML declaration.
     - `<kml>` and `<Document>` tags.
     - Project-level `<name>`.
     - Opening project `<Folder>` with project name.
   - `stationsFolder`:
     - `<Folder name="Stations">`.
     - One `<Placemark>` per point with `<name>` and `<Point><coordinates>`.
   - `linesFolder`:
     - `<Folder name="Lines">`.
     - One `<Placemark>` per link with `<name>`, optional `<description>`, `<Style>`, and `<LineString>`.
   - `kmlFooter`:
     - Closing project folder, `</Document>` and `</kml>`.

4. Downloads the KML:
   - Creates a `Blob` from the KML string.
   - Creates a temporary `<a>` element with `href` set to a `blob:` URL.
   - Sets `download` to `Project_Name_MM-DD-YYYY.kml`.
   - Triggers a click on this link, then removes it and revokes the URL.

---

### 5.6 Help Overlay

Help behavior:

- **Open Help**:
  - Clicking the **Help** button sets `helpOverlay.style.display = 'flex'`.
- **Close Help**:
  - Clicking the **Close Help** button sets `helpOverlay.style.display = 'none'`.
  - Clicking outside the help blob (on the translucent background) also closes it:
    - The event handler checks `if (e.target === helpOverlay)`.

The help text mirrors the key user flows described in this manual.

---

## Chapter 6 – Extending and Maintaining the Code

### 6.1 Adding New Fields

To add a new property to the data model:

1. **Points** (e.g., `type`, `notes`):
   - Add the property when creating points in `addPoint()`.
   - Include it in `saveState()` when serializing.
   - Restore it in `loadState()`.
   - Optionally render it in `renderPoints()` (e.g., new table column).

2. **Links** (e.g., `cableSize`, `conduitType`):
   - Add the property when creating links in `addLink()`.
   - Include it in `saveState()` and `loadState()`.
   - Optionally show or edit it in `renderLinks()`.

If you want these new properties to appear in KML:

- Add them to `<description>` or additional KML elements in the export builder.

---

### 6.2 Adding New Export Formats

To support CSV, GeoJSON, or another format:

1. Add a new button and handler (e.g., `Export to CSV`).
2. Build a string (or object) representation from `points` and `links`.
3. Use the same `Blob` + `<a>` download pattern:
   - Create a `Blob` with the appropriate MIME type.
   - Create a temporary link pointing to the blob URL.
   - Assign a meaningful filename, then trigger the download.

This preserves the self-contained nature of the tool while extending interoperability.

---

### 6.3 Splitting Into Separate Files (Optional)

For larger projects, you might want to split:

- `index.html` – structure.
- `styles.css` – styles.
- `app.js` – logic.

Within `app.js`, you could group functions into modules:

- `Storage`: `saveState`, `loadState`, `clearSession`.
- `CanvasView`: `layoutPointsToCanvas`, `drawCanvas`, `getPointAtCanvasCoords`.
- `Geo`: distance calculations (Haversine).
- `Exporter`: KML and other export formats.
- `Ui`: event wiring for buttons and inputs.

The current single-file approach is optimized for portability and ease of deployment.

---

## Chapter 7 – Summary

Smart Walkout:

- Gives field teams a quick way to capture stations, connect them, and estimate distances.
- Stores everything locally and exports clean KML with a project-centered folder structure.
- Uses simple, readable JavaScript with no dependencies or backend services.

This manual is structured so that:

- **Chapters 1–2** help end users understand how to operate the tool in the field.
- **Chapters 3–6** help developers understand how the code is organized, how data flows, and where to extend or maintain it.

With this manual and the HTML file, a future developer should be able to pick up Smart Walkout, understand its intent and structure, and confidently evolve it to meet new requirements.
