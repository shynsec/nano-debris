<img width="128" height="128" alt="logo" src="https://github.com/user-attachments/assets/44d90ce1-bf74-4c61-8f8c-76ca703fd85a" />


# Nano Debris - Orbital Intelligence

A real-time space debris monitoring dashboard built as a single self-contained HTML file. No backend, no build step, no dependencies to install. Open it in a browser and it fetches live TLE data from CelesTrak, propagates real satellite positions using SGP4 orbital mechanics, and renders everything on an interactive 3D globe.

<img width="1919" height="1015" alt="Screenshot from 2026-03-31 22-09-28" src="https://github.com/user-attachments/assets/5f182024-0726-48e4-b068-c0739e4ed411" />


---

## Who is this for?

- **Satellite operators & mission controllers** who want a fast visual check of the debris environment around their orbital regime
- **SSA analysts** at space agencies who need to communicate conjunction risk clearly
- **Space policy researchers & academics** studying orbital sustainability and the Kessler cascade
- **NewSpace startups & university CubeSat teams** who need a free, accessible alternative to enterprise SSA tools
- **Space enthusiasts** who want to understand what's actually up there

---

## Features

### Live Data
- Fetches real Two-Line Element (TLE) sets directly from [CelesTrak](https://celestrak.org) on page load across four catalogues: space stations, active satellites, Cosmos-1408 debris cloud, and rocket bodies
- Positions are computed using **SGP4/SDP4 orbital propagation** via [satellite.js](https://github.com/shashwatak/satellite-js) — the same standard used operationally
- Data re-propagates every 10 seconds against real UTC time
- Falls back gracefully to a hardcoded dataset if CelesTrak is unreachable

### 3D Globe & 2D Map
- Interactive rotating globe — drag to rotate, scroll to zoom, click any object to select it
- Toggle to a **2D equirectangular map view** for ground track and coverage analysis
- Orbit band rings for LEO, MEO, and GEO shown as dashed overlays
- Density heatmap highlights the most congested orbital shells

### Ground Track
- Select any object and enable **Track** to draw its 90-minute orbital path — 10 minutes of history plus 80 minutes of forecast
- Renders on both the 3D globe and the 2D map view

### Conjunction Risk
- Simulates [SOCRATES](https://celestrak.org/SOCRATES/)-style conjunction analysis, pairing debris objects with active satellites in similar altitude bands
- Risk events ranked by miss distance with probability of collision and Time of Closest Approach (TCA)
- HIGH risk events trigger a banner alert and a pulsing line between the two objects on the globe

### Reentry Predictor
- For any selected LEO object, estimates atmospheric reentry timeline based on orbital altitude
- Ranges from months (sub-300 km) to centuries (above 1,000 km)
- Shown in both the detail panel and the hover tooltip

### Historical Events Timeline
- Five annotated events in the right panel: Fengyun-1C (2007), Iridium 33 / Cosmos 2251 (2009), Cosmos 1408 ASAT test (2021), Starlink megaconstellation context, and the Kessler Syndrome threshold
- Click any event for a summary panel with fragment counts, altitudes, and impact data

### Export
- One-click **CSV export** of the full conjunction table, including object names, NORAD IDs, miss distances, collision probabilities, and TCA values
- File is named with today's date automatically

### Search
- Search by satellite name or NORAD ID — results show orbit band, altitude, and risk level inline

---

## Getting Started

No installation required.

**Option 1 — Open directly**
```
Download index.html → double-click → opens in your browser
```

**Option 2 — VS Code with Live Server (recommended for development)**
1. Install the [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) extension
2. Open the project folder in VS Code
3. Right-click `index.html` → **Open with Live Server**
4. The app opens at `http://127.0.0.1:5500` and auto-reloads on save

**Option 3 — GitHub Pages**

Push `index.html` to the root of a public GitHub repository, then go to:

`Settings → Pages → Source: Deploy from branch → main / (root)`

Your live URL will be:
```
https://yourusername.github.io/your-repo-name
```

> **Note:** An internet connection is required on first load. The app fetches TLE data from CelesTrak and loads fonts from Google Fonts. Once loaded, all rendering and propagation runs entirely client-side.

---

## Object Types

| Symbol | Colour | Type |
|--------|--------|------|
| ● | White | Space Station (ISS, Tiangong) |
| ● | Indigo | Active Satellite |
| ● | Orange | Debris Fragment |
| ● | Purple | Rocket Body |
| ● | Red | HIGH Conjunction Risk |
| ● | Yellow | MED Conjunction Risk |

---

## Controls

| Action | Control |
|--------|---------|
| Rotate globe | Click and drag |
| Zoom | Scroll wheel |
| Select object | Click |
| Hover for quick info | Mouse over any dot |
| Switch to 2D map | Toolbar → **2D Map** |
| Show ground track | Select object → **Track** |
| Toggle heatmap | Toolbar → **Heatmap** |
| Filter by type | Toolbar pill buttons |
| Filter by orbit band | LEO / MEO / GEO toggles |
| Export conjunctions | Header → **↓ CSV** |
| Refresh live data | Header → **↺ Refresh** |

---

## Data Sources

| Source | What it provides |
|--------|-----------------|
| [CelesTrak GP/JSON](https://celestrak.org/CCSDS/bulk.php) | Live TLE data for all object types |
| [satellite.js](https://github.com/shashwatak/satellite-js) | SGP4/SDP4 orbital propagation library |
| SOCRATES methodology | Conjunction risk model (simulated — real SOCRATES requires authentication) |

> The conjunction risk figures shown are **simulated** using altitude-proximity heuristics to replicate the SOCRATES methodology. They are illustrative, not operationally certified. Do not use for flight safety decisions.

---

## Technical Notes

**Architecture**
- Single HTML file — all CSS, JavaScript, and data in one place
- No frameworks, no bundler, no package manager
- Canvas-based rendering using the native 2D API
- SGP4 propagation via satellite.js loaded from cdnjs

**Performance**
- Propagates up to ~670 objects (configurable via `maxLoad` in the `FEEDS` array)
- Re-propagates every 10 seconds in the render loop
- Ground track computes 100 SGP4 steps on demand

**Increasing object count**
To load more objects, edit the `maxLoad` values in the `FEEDS` array near the top of the script:

```js
const FEEDS = [
  { key:'station', label:'Stations', url:'...', maxLoad: 20  },  // increase as needed
  { key:'active',  label:'Active',   url:'...', maxLoad: 300 },
  { key:'debris',  label:'Debris',   url:'...', maxLoad: 200 },
  { key:'rocket',  label:'Rockets',  url:'...', maxLoad: 150 },
];
```

Loading more than ~1,500 objects may affect performance on lower-end hardware.

**Adding more debris clouds**
CelesTrak publishes separate feeds for Fengyun-1C, IRIDIUM-33, and the full catalogue. Add additional entries to the `FEEDS` array with the corresponding CelesTrak GROUP parameter.

---

## Security

Six security patches are applied in this codebase:

- **XSS prevention** — all external API data is HTML-escaped before insertion into the DOM
- **Subresource Integrity** — satellite.js is loaded with a SHA-512 integrity hash
- **Content Security Policy** — restricts script sources, network connections, and blocks `object-src`
- **CSV injection guard** — formula-triggering characters are prefixed before export
- **Error logging** — propagation and parse errors are surfaced to the console rather than silently swallowed
- **Proper error types** — fetch failures throw `Error` objects with stack traces

---

## Licence

MIT — free to use, modify, and deploy. Attribution appreciated but not required.

---

## Acknowledgements

- [CelesTrak](https://celestrak.org) by Dr T.S. Kelso — for making orbital data freely and reliably accessible
- [satellite.js](https://github.com/shashwatak/satellite-js) — JavaScript SGP4/SDP4 implementation
- NASA Orbital Debris Program Office — for public research on the debris environment
- ESA Space Debris Office — for SOCRATES methodology documentation
