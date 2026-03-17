# Smart Walkout – User & Developer Manual (Updated)

---

## Chapter 1 – Use Intention

### 1.1 Purpose

Smart Walkout is a lightweight, browser-based tool designed to:

- Capture GPS-based stations in the field.
- Visually connect those stations into lines.
- Record walking paths as tracked polylines.
- Estimate distances (in feet) for lines and tracked polylines.
- Export the resulting layout as KML for use in tools like Google Earth Pro.

The application:

- Runs entirely in the browser (no backend).
- Uses device geolocation when available.
- Persists your work locally using the browser’s localStorage.
- Is simple enough for field use, yet structured for easy code maintenance.

### 1.2 Typical Use Cases

- Construction or engineering walkouts (planned cable/conduit routes).
- High-level site surveys where full CAD/GIS is overkill.
- Quick as-built sketches with approximate positions and lengths.
- Any field workflow needing:
  - Stations with coordinates.
  - Lines between stations.
  - Tracked paths with total footage.
  - KML export for visualization.

---

## Chapter 2 – How to Use Smart Walkout

### 2.1 Requirements & Setup

- A modern browser: Chrome, Edge, Firefox, Safari.
- Device/location services enabled if you want live GPS capture.
- No installation:
  - Save the HTML file.
  - Open it directly in the browser.

All data is stored in your browser’s localStorage on that device. It does not automatically sync between devices or browsers.

---

### 2.2 Project ID / Name

At the top:

1. Locate “Project ID / Name”.
2. Enter a descriptive label, for example:
   - FTTH – Town Center Job 1234
   - Ring A – North Loop

This project name is used:

- As the KML Document name.
- As the master folder name in the KML file.
- In the exported filename:

  Project_Name_MM-DD-YYYY.kml

  (special characters are sanitized for filenames).

---

### 2.3 Recording Stations (Points)

In the “Record Current Location” card:

1. Enter a Location Name (for example, Station 1, Node A, Pole 15).
2. Click “Record Current GPS”:
   - The browser prompts for location access (once per session). Allow it.
   - Smart Walkout reads your current position from navigator.geolocation.
3. On success:
   - A new station is created with:
     - Your chosen name.
     - Latitude and longitude from the device.
   - Status line example:
     - Recorded: Station 1 (Lat: 40.123456, Lon: -74.987654)
4. The station is added to:
   - The Record Inventory table (Type = ●).
   - The canvas as a clickable point.

---

## Chapter 3 – Record Inventory Table

### 3.1 Unified Table Overview

The “Record Inventory (Stations, Lines, Tracks)” card holds everything:

- Stations (points you record) – Type = ●
- Lines (connections between two stations) – Type = ━
- Tracks (polylines from walking) – Type = ≋

Table columns:

- # – Row index.
- Type –
  - ● = Station
  - ━ = Station-to-station line
  - ≋ = Tracked polyline
- Name – Editable record name.
- Col A / Col B – Meaning depends on Type (described below).
- Distance (ft) – Total length for lines and tracks.
- Actions – Currently “Del” to delete the record.

### 3.2 Per-Type Behavior

Stations (●):

- Name:
  - Editable. Renaming updates the label in the canvas and in any From/To references.
- Col A:
  - Latitude (editable).
- Col B:
  - Longitude (editable).
- Distance (ft):
  - Blank for stations.
- Actions:
  - Del removes the station and any lines attached to it.

Lines (━):

- Created when you connect two stations on the canvas.
- Name:
  - Editable.
- Col A:
  - From station name (read-only).
- Col B:
  - To station name (read-only).
- Distance (ft):
  - Approximate distance between the two station coordinates, in feet.
- Actions:
  - Del removes only that line.

Tracks (≋):

- Created by Start Track / Stop Track (polyline tracking).
- Name:
  - Editable.
- Col A:
  - “Polyline”.
- Col B:
  - Vertex count (for example, “17 pts”).
- Distance (ft):
  - Sum of all segment distances in the track, in feet.
- Actions:
  - Del removes only that tracked polyline.

---

## Chapter 4 – Canvas: Viewing and Creating Lines

### 4.1 What You See

In the “Canvas View (Tap to Connect Points)” card:

- Stations (●) appear as yellow circles with labels.
- Lines (━) and tracks (≋) appear as yellow linework overlaid on the canvas.
- Distances are shown near each line/track label (in feet).

Stations have been made larger and the hit area has been increased to make them easier to tap with a finger.

### 4.2 Creating a Line Between Stations

1. Tap or click a station on the canvas:
   - This becomes the From station.
   - It is highlighted with a green ring.
2. Tap or click another station:
   - This becomes the To station.
3. A “New Connection” modal appears:
   - Shows From and To station names.
   - Lets you enter an optional Connection Name (for example, Segment 1).
4. Confirm:
   - A new line record (Type = ━) is added to the inventory table.
   - The line is drawn between the two stations on the canvas.
   - Distance (ft) is computed immediately.

Cancel behavior:

- If you click the same station twice, it clears the selection.
- You can also cancel from the modal to abandon the line.

---

## Chapter 5 – Tracking a Polyline (Walking a Route)

### 5.1 Controls

In the “Record Current Location” card:

- Start Track
- Stop Track

These control the polyline tracker.

### 5.2 Vertex Placement Logic

While tracking:

- A first vertex is recorded at your location as soon as tracking starts.
- Additional vertices are only added when your position changes at least:

  0.00001 degrees

  measured as the square root of (Δlat² + Δlon²) compared to the last vertex.

This threshold filters out very small movements and GPS jitter so you do not get an excessive number of points.

When you press “Stop Track”:

- The watchPosition subscription is cleared.
- A final vertex is recorded at your current location if it is sufficiently different from the last vertex (by the same 0.00001-degree rule).
- If there are at least two vertices:
  - You are prompted for a track name (default: “Track N”).
  - A new track record (Type = ≋) is added to the inventory table.
  - Total distance is computed as the sum of segment distances between all consecutive vertices.
- If there are fewer than two vertices, the track is discarded with a warning.

### 5.3 How Tracks Display

In the inventory table:

- Type = ≋
- Name = user-provided or default (“Track N”).
- Col A = “Polyline”.
- Col B = “X pts” (number of vertices captured).
- Distance (ft) = total track length.

On the canvas:

- Tracks are drawn as multi-segment polylines.
- A label near the midpoint shows the name and total distance (ft).

---

## Chapter 6 – Autosave & Session Management

### 6.1 Autosave Indicator

Top right: a pill showing:

- Red dot + text “Unsaved changes” when unsaved edits exist.
- Green dot + “Saved (last auto-save: YYYY-MM-DD HH:MM:SS)” after a successful save.

### 6.2 How Autosave Works

- Any change to:
  - Project name
  - Stations (name, lat, lon)
  - Lines (name, creation/deletion)
  - Tracks (creation, name, deletion)
- Calls markDirty(), which:
  - Sets isDirty = true.
  - Updates the indicator.
  - Schedules saveState() after 1 second of inactivity.

saveState():

- Serializes:
  - projectName
  - points
  - links (including polylines)
  - nextPointId
  - nextLinkId
  - lastSavedAt (timestamp)
- Writes the JSON to localStorage under a fixed key.
- Sets isDirty = false and updates lastSavedAt and the indicator.

### 6.3 Reloading and Clearing Sessions

On reload (same device + browser):

- loadState() rehydrates projectName, stations, lines, tracks, and IDs.
- The unified inventory table and canvas are rendered accordingly.

Clear Session:

- Removes all stations, lines, tracks, and saved state.
- Resets project name and internal counters.
- Stops any active tracking and resets UI state.

---

## Chapter 7 – Export to KML

### 7.1 What Gets Exported

When you click “Export to .kml”:

- If there are no stations and no tracks/lines, you get a warning.
- Otherwise, a KML file is built with:
  - One “Stations” folder.
  - One “Lines” folder.

### 7.2 Stations Folder

For each station (●):

- A Placemark is created with:
  - name = station name (or default “Point N”).
  - Point geometry with lon,lat,0 coordinates.

### 7.3 Lines Folder

For each line or track (━ or ≋):

- A Placemark is created with:
  - name = record name (or “Link N” fallback).
  - Optional description = “Distance: XXXX ft” (rounded).
  - A LineStyle for color/width.
  - A LineString with coordinates:
    - For lines (━):
      - Two coordinates: fromStationLon,fromStationLat,0 and toStationLon,toStationLat,0.
    - For tracks (≋):
      - All vertices in the polyline: lon1,lat1,0 lon2,lat2,0 … etc.

### 7.4 KML File Name

- Derived from:
  - Sanitized project name (non-alphanumeric replaced with underscores).
  - Date stamp (MM-DD-YYYY).
- Format:

  Project_Name_MM-DD-YYYY.kml

---

## Chapter 8 – Developer Architecture

### 8.1 Technology Stack

- HTML for structure.
- CSS for layout and dark-themed UI.
- Vanilla JavaScript for:
  - Data model (stations, lines, tracks).
  - Canvas rendering.
  - Geolocation tracking.
  - Autosave and KML export.
- Runs entirely in the browser; no external libraries or backend.

### 8.2 Data Structures

Stations:

    {
      id: Number,
      name: String,
      lat: Number,
      lon: Number,
      x: Number|null,
      y: Number|null
    }

Links and tracks:

    {
      id: Number,
      name: String,
      fromId: Number|null,
      toId: Number|null,
      distanceFeet: Number,
      polyline: [
        { lat: Number, lon: Number },
        ...
      ] | null
    }

Semantics:

- fromId/toId are set for simple lines (━).
- polyline is set (array) for tracked paths (≋) and null for simple lines.
- distanceFeet is always the total distance of the line or polyline.

### 8.3 Persisted State

Stored in localStorage:

    {
      projectName: String,
      points: [ { id, name, lat, lon } ],
      links: [ { id, name, fromId, toId, distanceFeet, polyline } ],
      nextPointId: Number,
      nextLinkId: Number,
      lastSavedAt: Number
    }

Upon load:

- points and links are reconstructed into in-memory arrays.
- x and y for stations are recomputed by layoutPointsToCanvas().

### 8.4 Layout and Drawing

layoutPointsToCanvas():

- Computes global bounds from:
  - All station lat/lon.
  - All polyline vertices (tracks).
- Adds small padding if lat or lon ranges collapse.
- Stores:
  - minLatGlobal, maxLatGlobal, minLonGlobal, maxLonGlobal.
- Projects each station:
  - Normalized x and y into the canvas drawing area with padding.

drawCanvas():

- Clears canvas.
- Draws links and polylines:
  - For simple lines:
    - Looks up from/to stations by id and draws a single segment.
  - For polylines:
    - Projects each vertex and renders a multi-segment path.
  - Labels each with its name and total distance.
- Draws stations:
  - Yellow circles (radius ≈ 10px).
  - Labels offset to the right.
- Highlights a pending “From” station when creating a new line.

getPointAtCanvasCoords():

- Hit-test function using a radius ≈ 18 pixels for finger-friendly selection.

### 8.5 Geolocation Integration

Single-point capture (Record Current GPS):

- Uses navigator.geolocation.getCurrentPosition.
- On success, calls addPoint(name, lat, lon).

Tracking (Start Track / Stop Track):

- Uses navigator.geolocation.watchPosition to stream positions.
- Adds vertices to currentTrack when the distance in degrees exceeds MIN_TRACK_DELTA_DEGREES (0.00001).
- On Stop Track, optionally adds a final vertex and finalizes into a track record.

### 8.6 Distance Calculation

Haversine-based:

- haversineDistanceMeters(lat1, lon1, lat2, lon2)
- distanceFeetBetweenPoints(p1, p2) converts meters to feet via factor 3.28084.

Uses:

- When creating station-to-station lines.
- When finalizing tracked polylines.
- When reprojecting station coordinates and recomputing link distances.

---

## Chapter 9 – Extending and Maintaining the Code

### 9.1 Adding New Fields

To add new properties, for example:

- Station attributes:
  - type (pole, handhole, node).
  - notes.
- Link attributes:
  - cableType.
  - conduitSize.

You would:

1. Extend objects in addPoint() or link creation.
2. Update saveState() and loadState() to include the new fields.
3. Update renderInventory() to display/edit them where appropriate.
4. Optionally reflect them in KML export (for example, in description).

### 9.2 Adding New Export Formats

To add CSV, GeoJSON, or other exports:

- Create a new button and handler.
- Iterate over points and links to build the desired format.
- Use the Blob + temporary link pattern (same as KML) to trigger download.

### 9.3 Refactoring into Modules

For larger-scale maintenance, consider:

- Splitting into:
  - index.html (structure)
  - styles.css (styling)
  - app.js (logic)
- Within app.js, grouping into modules:
  - Storage (save/load/clear).
  - Geo (distance, projections).
  - CanvasView (layout, draw, hit tests).
  - Inventory (renderInventory, CRUD operations).
  - Export (KML and other formats).
  - Ui (event handlers, modals, help).

The current single-file approach is intentional for portability and offline use, but modularization is straightforward.

---

## Chapter 10 – Summary

The current version of Smart Walkout provides:

- Unified record inventory for all stations, lines, and tracks in a single table.
- Finger-friendly station selection and clear visual feedback on the canvas.
- Polyline tracking with a fine-grained vertex filter (0.00001°) and total track distance in feet.
- KML export that accurately represents stations, simple lines, and multi-vertex polylines.
- Autosave to localStorage and complete session clearing.

With this updated manual and the HTML source, a future developer can understand how stations, lines, and polylines are represented, how they are rendered, and how to extend the tool to fit new field workflows or data requirements.
