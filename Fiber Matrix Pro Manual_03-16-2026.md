# Fiber Matrix Pro — User & Developer Manual

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Core Workflows](#core-workflows)
4. [File Management](#file-management)
5. [Architecture & Technical Overview](#architecture--technical-overview)
6. [Data Model & Schema](#data-model--schema)
7. [System Mechanics](#system-mechanics)
8. [Troubleshooting & FAQ](#troubleshooting--faq)
9. [Developer Notes](#developer-notes)

---

## Introduction

**Fiber Matrix Pro (FMP)** is a field-ready web application for documenting fiber optic splices at geographic locations. It combines GPS-based location tracking with detailed splice records, exports data as both JSON backups and spreadsheet-ready CSVs, and runs entirely in the browser with no server dependency.

### Key Features

- **GPS-enabled location recording** with manual fallback
- **Splice documentation** with color-coded fiber bundle and fiber identification
- **Real-time search** across all locations
- **JSON session backups** for data portability
- **ZIP-packaged CSV exports** for spreadsheet analysis
- **Dark/light theme toggle**
- **Full edit capabilities** for locations and splices

### Who Should Use This

- **Field technicians** documenting fiber installations
- **Network engineers** tracking splice topology
- **Anyone managing geographic splice records**

---

## Getting Started

### First Launch

1. Open `fiber_matrix_pro.html` in a web browser
2. The app loads with an empty location list
3. All data is stored **in your browser's memory** (session data)
4. Use **💾 Save** frequently to back up to JSON

### Recording Your First Location

1. Click **📍 Record New Location**
2. Enter a descriptive name (e.g., "Tower A", "Vault 5")
3. The app attempts GPS capture:
   - **If GPS succeeds:** Location is created with precise coordinates
   - **If GPS fails:** Location is created with placeholder coordinates (`0.00000, 0.00000`)
4. A splice modal opens immediately—add your first splice or skip to edit later

### Creating Your First Splice

1. From any location, click **+ Add Splice**
2. Fill in **Side 1 (FROM)**:
   - **Cable/Device ID:** Identifier of the source (e.g., "CAB-001")
   - **Bundle:** Select fiber bundle color or use OVERRIDE for custom text
   - **Fiber:** Select individual fiber color or OVERRIDE for custom
3. Fill in **Side 2 (TO)** with the same fields
4. **Notes** (optional): Add context (e.g., "Feeds Site C")
5. Click **Confirm Splice** to save

---

## Core Workflows

### Adding & Editing Locations

#### Create a New Location
- Click **📍 Record New Location**
- Enter location name
- App attempts GPS; creates location with coords (auto or placeholder)

#### Edit an Existing Location
- Click **✏️ Edit** button next to any location
- Modify name and/or coordinates
- Click **Save Changes**
- Changes appear immediately in the list

#### Delete a Location
- Click **🗑️ Delete** button next to any location
- Confirm deletion dialog
- Location and all splices are removed

### Managing Splices

#### Add a Splice
- Click **+ Add Splice** on a location
- Fill in all fields (IDs, bundles, fibers, notes)
- Click **Confirm Splice**

#### Edit a Splice
- Scroll to the splice within a location
- Click **✏️ Edit Splice** button at the bottom of the row
- Modify fields
- Click **Save Changes**

#### Delete a Splice
- Click the **×** button (top-right corner of splice row)
- Confirm deletion
- Splice is removed, location data preserved

### Manual Overrides

For non-standard bundle or fiber colors:

1. Click **OVERRIDE** next to Bundle or Fiber dropdown
2. Text input field appears
3. Enter custom text (e.g., "CUSTOM-YELLOW", "PROPRIETARY-001")
4. Click **BACK** to revert to dropdown selection

**Display behavior:** Custom text displays in a box instead of a color swatch.

### Search & Filter

- Use the **search box** at the top of the controls
- Type location name (partial matches work)
- List filters in real-time
- Search is case-insensitive

### Splice Count Badge

Each location shows the number of splices below its name:
- "5 splices"
- "1 splice"

Useful for quick visual scan of populated vs. empty locations.

---

## File Management

### Backup Your Session (JSON)

**Button:** 💾 Save

**Output filename:** `FMPsession_MM-DD-YYYY_HH.MM.JSON`

**Contents:**
- All locations (name, lat, lon, ID)
- All splices for each location (IDs, bundles, fibers, notes)

**Use cases:**
- Daily backup routine
- Transfer data between devices
- Version control of your work
- Share with team members

**How to reload:**
1. Click **📂 Load**
2. Select a saved JSON file
3. Confirm: all data from that session is restored

### Export to Spreadsheet (ZIP)

**Button:** 📊 Export Excel CSVs

**Output filename:** `FMPcsv_MM-DD-YYYY_HH.MM.zip`

**Contents:**
- One CSV per location
- Filename: `LocationName_SpliceMatrix_MM-DD-YYYY_HH.MM.csv`

**CSV format:**
```
Location ID:,Tower A
Grid Coordinates:,43.12345 -89.54321
Export Date:,03-16-2026

CABLE/DEVICE ID 1,BUNDLE 1,FIBER 1, ,CABLE/DEVICE ID 2,BUNDLE 2,FIBER 2,SPLICE NOTES
"CAB-001","BLUE","WHITE","<>","CAB-002","ORANGE","BLUE","Feeds Site C"
```

**Use cases:**
- Import into Excel/Google Sheets for analysis
- Share with non-technical stakeholders
- Archival in spreadsheet format
- Generate reports

### Date/Time Format Reference

All exports use standardized naming:

- **MM** = Month (01-12)
- **DD** = Day (01-31)
- **YYYY** = Year (4 digits, e.g., 2026)
- **HH** = Hours in 24-hour format (00-23)
- **MM** = Minutes (00-59)

Example: `FMPsession_03-16-2026_14.23.JSON` = March 16, 2026 at 2:23 PM

---

## Architecture & Technical Overview

### High-Level Structure

Fiber Matrix Pro is a **single-file HTML/CSS/JavaScript application**:

```
fiber_matrix_pro.html
├── HTML Structure (modals, buttons, containers)
├── CSS (theme variables, responsive grid, dark mode)
└── JavaScript (app logic, data handling, file I/O)
```

**No external dependencies** except JSZip (loaded on-demand for CSV export).

### Data Flow Diagram

```
User Input → Validation → savedData Array → DOM Render → Browser Display
     ↓
   File I/O (Save/Load/Export)
```

### Core Systems

#### 1. **State Management**

The app maintains a single source of truth: the `savedData` array.

```javascript
let savedData = [
  {
    id: timestamp,
    name: string,
    lat: string (5 decimals),
    lon: string (5 decimals),
    splices: [
      {
        id: timestamp,
        id1: string,
        b1: string,
        f1: string,
        id2: string,
        b2: string,
        f2: string,
        notes: string
      }
    ]
  }
];
```

Every change updates this array, then `updateUI()` re-renders the page.

#### 2. **Modal System**

Three modals handle user input:

- **#modal** — Add/edit splices
- **#edit-modal** — Edit locations
- **#edit-splice-modal** — Edit existing splices
- **#help-modal** — Display help documentation

Toggle via CSS `.open` class:
```javascript
document.getElementById('modal').classList.add('open');    // Show
document.getElementById('modal').classList.remove('open'); // Hide
```

#### 3. **GPS & Location Recording**

**Workflow:**

1. User clicks **📍 Record New Location**
2. Prompt asks for location name
3. `navigator.geolocation.getCurrentPosition()` is called
4. **Success path:**
   - Coordinates captured with `pos.coords.latitude` and `pos.coords.longitude`
   - Location created immediately with real coords
   - Splice modal opens
5. **Failure path:**
   - GPS denied, unavailable, or timed out
   - Location still created with `0.00000, 0.00000` placeholder
   - User prompted to edit coordinates later
   - Splice modal still opens (no blocking)

**Why this design?**
- Prevents the app from "bricking" if GPS permission is denied at OS level
- User can always start documenting immediately
- Edit button allows coordinate correction anytime

#### 4. **File I/O**

**Save (JSON):**
```javascript
// Timestamp-based filename
const now = new Date();
const dateStr = formatted MM-DD-YYYY
const timeStr = formatted HH.MM
a.download = `FMPsession_${dateStr}_${timeStr}.JSON`;

// Entire savedData array as JSON
blob = JSON.stringify(savedData, null, 2)
```

**Load (JSON):**
```javascript
FileReader reads selected .JSON file
JSON.parse() reconstructs savedData
updateUI() re-renders page
```

**Export (ZIP):**
```javascript
1. Load JSZip library (CDN fallback)
2. Iterate over savedData
3. For each location, generate CSV string
4. Add CSV to zip with filename: LocationName_SpliceMatrix_MM-DD-YYYY_HH.MM.csv
5. generateAsync() creates blob
6. Download zip with filename: FMPcsv_MM-DD-YYYY_HH.MM.zip
```

#### 5. **Theme System**

Dark/light mode is toggled via `data-theme` attribute on `<body>`:

```javascript
// Light (default)
<body data-theme="light">

// Dark
<body data-theme="dark">
```

CSS uses `[data-theme="dark"]` selector to swap color variables:
```css
:root { --bg: #f4f4f9; --text: #1a1d20; }
[data-theme="dark"] { --bg: #121212; --text: #e0e0e0; }
```

Theme persists during session only (not saved to localStorage).

#### 6. **Search & Filtering**

Real-time filter implemented in `updateUI()`:

```javascript
const searchTerm = document.getElementById('search-input').value.toLowerCase();
const filtered = savedData.filter(loc => loc.name.toLowerCase().includes(searchTerm));
```

Only locations matching search term are rendered.

---

## Data Model & Schema

### Location Object

```javascript
{
  id: number,                    // Timestamp (milliseconds since epoch)
  name: string,                  // User-provided location name
  lat: string,                   // Latitude, 5 decimal places (e.g., "43.12345")
  lon: string,                   // Longitude, 5 decimal places (e.g., "-89.54321")
  splices: Splice[]              // Array of splice objects
}
```

**Example:**
```javascript
{
  id: 1710609600000,
  name: "Tower A",
  lat: "43.12345",
  lon: "-89.54321",
  splices: [ /* ... */ ]
}
```

### Splice Object

```javascript
{
  id: number,                    // Timestamp (milliseconds since epoch)
  id1: string,                   // Cable/Device ID (Side 1)
  b1: string,                    // Bundle color (Side 1)
  f1: string,                    // Fiber color (Side 1)
  id2: string,                   // Cable/Device ID (Side 2)
  b2: string,                    // Bundle color (Side 2)
  f2: string,                    // Fiber color (Side 2)
  notes: string                  // Optional splice notes
}
```

**Example:**
```javascript
{
  id: 1710609600001,
  id1: "CAB-001",
  b1: "BLUE",
  f1: "WHITE",
  id2: "CAB-002",
  b2: "ORANGE",
  f2: "BLUE",
  notes: "Feeds Site C"
}
```

### Supported Bundle/Fiber Colors

Standard EIA-598 color palette:

- BLUE, ORANGE, GREEN, BROWN, SLATE, WHITE
- RED, BLACK, YELLOW, VIOLET, ROSE, AQUA

**Striped variants:** Append `-STRIPE` to any color (e.g., `BLUE-STRIPE`, `BLACK-STRIPE`)

**Custom overrides:** Any text not matching standard colors displays as a text box instead of swatch.

### JSON Schema (Full Session)

```javascript
[
  {
    id: number,
    name: string,
    lat: string,
    lon: string,
    splices: [
      { id: number, id1: string, b1: string, f1: string, id2: string, b2: string, f2: string, notes: string },
      // ... more splices
    ]
  },
  // ... more locations
]
```

---

## System Mechanics

### GPS Geolocation Flow

```
User clicks 📍 Record New Location
         ↓
[Prompt for location name]
         ↓
    validateInput()
         ↓
navigator.geolocation.getCurrentPosition()
         ↓
     ┌───┴──────────────────────┐
     ↓                          ↓
  SUCCESS              ERROR/TIMEOUT
     ↓                          ↓
capture coords         create with placeholder
     ↓                          ↓
update location        [alert user] (optional)
     ↓                          ↓
openSpliceModal()  openSpliceModal()
     ↓                          ↓
   User adds splices
```

**Key behavior:** App never blocks on GPS failure. Location is created either way.

### CSV Generation Pipeline

```
For each location in savedData:
  1. Initialize CSV header
     Location ID:,{name}
     Grid Coordinates:,{lat} {lon}
     Export Date:,{dateStr}
  
  2. Add column headers
     CABLE/DEVICE ID 1,BUNDLE 1,FIBER 1, ,CABLE/DEVICE ID 2,...
  
  3. For each splice in location.splices:
     "id1","b1","f1","<>","id2","b2","f2","notes"
     (values quoted to handle commas)
  
  4. Add to ZIP with filename:
     {LocationName}_SpliceMatrix_MM-DD-YYYY_HH.MM.csv
```

### Splice Count Rendering

In `updateUI()`, display count under each location:

```javascript
`<div>${loc.splices.length} splice${loc.splices.length !== 1 ? 's' : ''}</div>`
```

Handles singular/plural automatically.

### Manual Override Display

When a bundle or fiber field contains text not matching standard colors:

```javascript
const isManualOverride = name && !colors[name.replace("-STRIPE", "")] && name !== "";

if (isManualOverride) {
  // Display as text box instead of color swatch
  return `<div style="...padding:4px;...">${name}</div>`;
} else {
  // Display color swatch
  return `<div class="swatch" style="background:${hex}"></div>`;
}
```

---

## Troubleshooting & FAQ

### GPS Not Working

**Symptom:** Location created with `0.00000, 0.00000` placeholder coords.

**Causes:**
1. GPS permission denied in browser
2. No GPS hardware on device (e.g., desktop PC)
3. Device location services disabled in OS
4. Browser doesn't support Geolocation API

**Solution:**
- Click **✏️ Edit** on the location
- Manually enter latitude and longitude
- Click **Save Changes**

### File Won't Load

**Symptom:** "Load failed: Invalid JSON format" alert.

**Causes:**
1. File is corrupted
2. File is not a valid JSON file
3. Wrong file extension or format

**Solution:**
- Verify file is a `.JSON` file (not `.json` or other)
- Open file in text editor; check for valid JSON syntax
- Re-save from FMP using **💾 Save** button
- Try loading again

### Location Data Lost After Reload

**Symptom:** Refreshed page and all locations disappeared.

**Cause:** FMP stores data **in browser memory only**. Closing the tab or reloading without saving clears all data.

**Solution:**
- Always click **💾 Save** before closing
- Load backup JSON file using **📂 Load** if available
- For frequent work, save every 30 minutes

### CSV Not Opening in Excel

**Symptom:** CSV opens but formatting is wrong (merged cells, misaligned).

**Cause:** Excel may not recognize the header format as metadata vs. data.

**Solution:**
- Open as **Data > From Text** import instead of direct open
- Set delimiter to comma
- First row will be parsed as headers

### Search Not Finding Location

**Symptom:** Typed location name but location doesn't appear.

**Cause:** Search is case-insensitive partial match. Ensure you're typing correctly.

**Solution:**
- Clear search box completely
- Type first few letters of location name
- Check that location exists (not accidentally deleted)

### Dark Mode Not Persisting

**Symptom:** Toggle dark mode, refresh page, and it reverts to light.

**Cause:** Theme preference is session-only (not saved to localStorage).

**Solution:**
- Theme toggle is for current session only—that's by design
- You can modify the code to use localStorage if desired (see Developer Notes)

### ZIP File Not Downloading

**Symptom:** Click "Export CSVs" and nothing happens.

**Cause:** 
1. JSZip library failed to load from CDN
2. Browser blocked download
3. No locations exist (empty zip attempted)

**Solution:**
- Check browser console (F12) for errors
- Ensure internet connection (JSZip loaded from CDN)
- Check browser download settings
- Create at least one location before exporting

---

## Developer Notes

### Forking & Extending

This tool is designed to be modified. Here are common customizations:

#### Add localStorage Persistence

Replace session-only memory with persistent storage:

```javascript
// In initOptions() or startup:
savedData = JSON.parse(localStorage.getItem('FMP_data')) || [];

// In updateUI() after rendering:
localStorage.setItem('FMP_data', JSON.stringify(savedData));
```

#### Add Custom Color Palette

Modify the `colors` object in the script:

```javascript
const colors = {
  "BLUE": "#0000FF",
  "ORANGE": "#FFA500",
  "CUSTOM_COLOR": "#AABBCC",  // Add custom
  // ...
};
```

#### Change Export Format

Modify `createZip()` to adjust CSV structure, add columns, or change delimiter.

#### Add Timestamp to Each Splice

In `commitSplice()`, add:

```javascript
splice.createdAt = new Date().toISOString();
```

#### Disable Manual Override

Comment out the OVERRIDE link in the modal HTML:

```html
<!-- <span class="override-link" onclick="toggleOverride('b1')">OVERRIDE</span> -->
```

### Code Organization

- **HTML:** Modals, input fields, button definitions
- **CSS:** Theme variables, layout grid, modal styles
- **JavaScript:**
  - State management (`savedData`)
  - Event handlers (clicks, input changes)
  - File I/O (save, load, export)
  - UI rendering (`updateUI()`)
  - GPS and modals

### Performance Notes

- App scales well to ~500 locations
- Search filter is O(n) on location count
- CSV generation is synchronous (blocks UI briefly for large datasets)
- Consider async CSV generation for 1000+ splices

### Browser Compatibility

- **Geolocation API:** All modern browsers
- **FileReader API:** IE10+, all modern browsers
- **JSZip library:** All modern browsers
- **CSS Grid:** IE11+, all modern browsers
- **Dark mode support:** All modern browsers

---

## Closing Notes

Fiber Matrix Pro is intentionally lightweight and self-contained. It prioritizes simplicity, portability, and reliability in field conditions. All data stays on your device; no cloud sync, no accounts, no tracking.

For questions, modifications, or contributions, refer to the code comments in the HTML file.

**Happy documenting!**