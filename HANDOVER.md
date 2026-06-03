# Homefolk Builder — Handover

A single, self-contained HTML file (`homefolk-builder.html`, ~44 KB) that lets someone
drop a tiny-home pod onto a real underused site in Hackney, customise its interior in 3D,
and look around the actual street. Built as a pitch demo for community testers, branded to
the *LIVE PROJECT 2024* Homefolk document.

No build step, no bundler, no API keys to configure. Open it directly (`file://`) or serve
it over HTTP. It needs an internet connection because maps, models, art, and street imagery
are all loaded from public CDNs/APIs at runtime — the HTML itself stays small.

## Stack (all via CDN)

| Library | Role | How it loads |
|---|---|---|
| Leaflet 1.9 | Hackney map + OpenStreetMap tiles | classic `<script>`, global `L` |
| mapillary-js 4.1 | Street-view panorama | classic `<script>`, global `mapillary` |
| Three.js 0.160 | 3D pod interior | ES module via `<script type="importmap">` |
| Poppins | Brand typeface | Google Fonts |

Three.js is loaded as a module so `GLTFLoader` and `OrbitControls` (its `examples/jsm/`
addons) resolve the bare `three` specifier through the importmap. Leaflet and Mapillary are
plain globals the module reads off `window`.

## File layout

It is one file with clearly delimited regions:

1. `<head>` — CDN links + one big `<style>` block. The stylesheet is sectioned by comment
   banners: **1 Brand tokens**, **2 Landing**, **3 Map**, **4 Studio**.
2. A hidden `<svg><defs>` holding three reusable shapes referenced via `<use>`:
   `#aframe` (the logo), `#podglyph` (the pod chip/marker), `#pinglyph` (empty/error states).
3. Three full-screen `<section class="screen">` blocks: `#landing`, `#map-screen`, `#studio`.
   Only one has `.active` at a time; `showScreen(name)` toggles them.
4. The `<script type="module">` with all app logic.

### Brand tokens
All colours extracted from the PDF live as CSS custom properties in `:root`
(`--green`, `--sage`, `--coral`, `--maroon`, `--sky`, `--ink`). Change the palette there and
it propagates everywhere, including the 3D swatch options.

## Application logic

### State
A single module-scoped object:

```
App = { screen, pods[], currentPod, map, sitesLayer, sites[], mlyToken }
```

- `pods[]` — each `{ id, latlng, type, interior:{ wall, floor, roof, furniture[], art } }`.
  A pod owns its whole interior config, so re-opening it restores everything.
- `sites[]` — candidate plots as `{ type, ring:[[lat,lng]...] }`, used for the drop test.
- `mlyToken` — Mapillary access token (defaults to a working public one; editable in the UI
  and cached in `localStorage`).

Constants near the top you will likely touch: `HACKNEY` (bounding box), `SITE_STYLE`
(tag→colour/label), `FURNITURE` (the 15 curated model URLs), `SWATCHES` (recolour options).

### Screen 1 — Landing
Static markup. The Start button calls `showScreen('map')` then `initMap()`.

### Screen 2 — Map
- `initMap()` builds the Leaflet map, adds OSM tiles, locks `maxBounds` to Hackney, creates
  `App.sitesLayer`, wires drop handling, and calls `loadSites()`.
- `loadSites()` queries the **Overpass API** for underused land and draws each result as a
  coloured polygon. It runs **two** requests (parking on its own; brownfield + garages +
  construction together) to avoid the rate-limit you get from firing many at once.
  `fetchOverpass(query)` walks `OVERPASS_EPS` in order with a per-request timeout, so a slow
  or busy mirror falls through to the next. `classify(tags)` maps OSM tags to a site type;
  `renderSites()` draws polygons and updates the legend counts; `showSitesError()` offers a
  Retry if every mirror fails.
  - Note: `building=garage` is intentionally excluded (it matched every lock-up and timed
    out the query), and `overpass.osm.ch` is excluded (Switzerland-only extract).
- Drag-drop: the pod chip is a native `draggable` element. On `drop`, the map converts the
  pixel to a lat/lng (`map.mouseEventToLatLng`), and `siteAt()` runs a ray-cast
  point-in-polygon (`inRing`) against `App.sites`. A hit calls `placePod()`; a miss shows a
  hint. `placePod()` adds a marker whose click opens the studio.

### Screen 3 — Studio
Split view: 3D interior on the left, street view on the right.

**3D (Three.js).** All scene objects live in the `TD` object, created lazily by
`ensureThree()` (scene, camera, renderer, OrbitControls, lights, the pod shell — floor, three
walls, gabled roof, a back-wall art frame — plus a raycaster and a `GLTFLoader`). A single
render loop runs once created; `resizeThree()` (driven by a `ResizeObserver`) keeps it sized.

- `applySurface(kind, color)` recolours walls/floor/roof and saves to the pod.
- `addFurniture(item, saved?)` loads a GLB from Poly Pizza, normalises oversized models,
  seats it on the floor, makes it selectable, and (re)stores transforms via `saveFurniture()`.
- `wireThreePointer()` handles selection and drag-to-move on the floor plane; the on-canvas
  toolbar rotates/deletes the selected piece.
- `hangArt()` fetches a random public-domain artwork from the **Art Institute of Chicago**
  API (with a **Met Museum** fallback) and applies it as a texture on the frame.
- `openStudio(pod)` is the entry point: it shows the screen, builds the palettes once
  (`buildPalettes()`), clears and rebuilds the scene from the pod's saved interior, then
  triggers the street view.

**Street view (Mapillary).** `loadStreetView(latlng)` asks the Mapillary Graph API for the
nearest image (`nearestImage()`, widening the search box once if needed), then constructs a
`mapillary.Viewer`. If there is no imagery, `showMlyEmpty()` renders a tidy placeholder. The
token field re-runs this for the current pod.

## Data sources

| What | Source | Key? |
|---|---|---|
| Base map tiles | OpenStreetMap (`tile.openstreetmap.org`) | no |
| Candidate sites | Overpass API (`OVERPASS_EPS` mirrors) | no |
| Furniture models | Poly Pizza CDN (`static.poly.pizza`), Quaternius CC0 | no |
| Wall art | Art Institute of Chicago API, Met Open Access | no |
| Street imagery | Mapillary Graph API + viewer | free token (shipped) |

All return permissive CORS, including for a `null` origin, so the file works when opened
straight from disk.

## Extending it

- **More furniture** — add `{ n, u }` rows to `FURNITURE` (find model GLB URLs on poly.pizza;
  they resolve to `static.poly.pizza/{id}.glb`).
- **Different area** — change the `HACKNEY` bounding box and the Overpass bbox follows.
- **More site types** — add an entry to `SITE_STYLE`, a clause in `loadSites()`, and a branch
  in `classify()`; add a matching legend row in the map markup.
- **Recolour palette** — edit `SWATCHES` (and the `:root` brand tokens for global colours).
- **Your own Mapillary token** — paste it into the field top-right of the studio, or change
  the default in `App.mlyToken`.

## Known caveats

- **Overpass can be busy.** Public mirrors throttle heavy use; the app already does endpoint
  fallback + a Retry button. A normal first load returns ~850 Hackney sites in a few seconds.
- **Mapillary coverage is crowd-sourced.** Some side-streets have no imagery; the studio
  shows an empty state rather than failing.
- **Three.js "multiple instances" console warning** is harmless (an unpkg module-resolution
  quirk); the scene, model loading, and raycasting all work.
- The shipped Mapillary token is a shared public one. For anything beyond a demo, swap in a
  free token from the Mapillary developer dashboard.
