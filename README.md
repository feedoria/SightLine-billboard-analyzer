# SightLine

**Pre-testing outdoor billboard campaigns before you pay for them.**

Give SightLine an ad creative and a few billboard locations with their monthly prices, and it tells you **which location brings the most real impressions per euro**: drivers who actually see the panel and have time to read it. It also shows the driver's view from 150, 80 and 30 m and scores the creative for outdoor readability.

Built in 10 hours by a team of four at the **ITFest × Adobe marketing hackathon** (Bucharest, November 2026). This is my copy of the team project; the original repository is [PanzaruRares/Hackathon-Project](https://github.com/PanzaruRares/Hackathon-Project).

## How it works

```
cost_per_real_impression = monthly_price / (cars_per_day × 30 × visibility_factor)
visibility_factor        = (1 − occluded%) × min(1, visible_seconds / seconds_needed_to_read)
```

- **Traffic:** estimated from OpenStreetMap road type and lanes, adjusted by the street's betweenness centrality in Bucharest's road network.
- **Visibility:** rays cast from the driver's eye height (1.2 m) towards the panel, through OSM buildings and Google's photorealistic 3D model.
- **Ad quality score:** local OCR + OpenCV analysis on 6 criteria (legibility, contrast, amount of text, clutter, hierarchy, spacing).

| Folder | Contents |
|---|---|
| `web/` | Dashboard (map, pins, prices), results page, driver's view (`sightline-view.js`), campaign brief generator |
| `scoring/` | Traffic estimate, street centrality, cost-per-impression score, local server |
| `scoring/analiza_panou/` | Outdoor Ad Quality Score (EasyOCR + OpenCV) with tests |
| `data/` | Cached OpenStreetMap data, so the demo works offline |
| `unity/` | The Unity prototype that was dropped (see below) |

## My part

I built the **driver's view and the visibility calculation**: first as a Unity + Cesium prototype, then the version that shipped, in the browser (`web/sightline-3d.html`, `web/sightline-streetview.html`, `web/sightline-view.js`). It shows Street View photos with the panel drawn at the chosen pin and computes how much of it is hidden, by casting rays through Google 3D Tiles in a hidden CesiumJS viewer.

### From Unity to Street View

The first idea was to locate each address in Unity (Cesium + Google 3D Tiles) and show a **pre-recorded render**, like a short GIF of the driver approaching the billboard. In practice:

- the renders came out **blurry**: the photogrammetry looks rough at street level, especially trees and facades;
- it was an **offline step** (open the editor, press Play, wait up to 30 s per viewpoint for the tiles to load), while the site had to answer for any pin on the map;
- Google's 3D tiles can't be cached on disk, so the renders couldn't be prepared in advance.

So I switched to **Google Street View**: real photos of the street with the billboard drawn at the chosen location, **live** in the browser, for any address. The impact was much stronger than a blurry render. The 3D model stayed in the background, in a hidden CesiumJS viewer, only to calculate how much of the panel is hidden by buildings and trees.

## APIs used

| API | Used for |
|---|---|
| Google Maps JavaScript API (Street View) | Live street-level panoramas for the driver's view |
| Google Street View Static API | Street View images at 150 / 80 / 30 m |
| Google Photorealistic 3D Tiles, via Cesium ion + CesiumJS | 3D model of the city, used to check which buildings and trees hide the panel |
| OpenStreetMap Overpass API | Roads, lanes, speed limits, buildings and trees around each location |
| OpenStreetMap tiles + Leaflet | Interactive map for placing the billboard pins |
| Gemini API (`gemini-3.1-flash-lite`) | Text suggestions based on the computed ad scores |
| Pollinations.ai | AI-drafted campaign brief (with a local template fallback) |

## Our approach to AI

- **In development:** each of us worked in parallel with an AI coding agent (Claude Code), with a shared project context (`CLAUDE.md`) and fixed JSON contracts between modules.
- **In the product, deliberately limited:** AI never computes the numbers. It only turns the measured results into suggestions (Gemini) and drafts a campaign brief.
- **No AI-generated images or videos**, out of ethics and respect for artists. SightLine never creates or edits the creative; it only measures how the work of designers, illustrators and photographers will perform on the street.

The jury felt the product needed more AI. For us, that was a conscious trade-off.

## Running locally

```bash
pip install -r scoring/requirements.txt
cp .env.example .env                           # GEMINI_API_KEY (optional)
cp web/config.example.js web/config.local.js   # Google Maps key + Cesium ion token for the driver's view
python scoring/analiza_panou/server_site.py    # from the repo root
```

Then open `http://localhost:8000`.

**Stack:** JavaScript, Leaflet, CesiumJS, Google 3D Tiles & Street View · Python, OpenStreetMap/Overpass, EasyOCR, OpenCV · Gemini · Unity 6 + Cesium for Unity (prototype)

Map data © OpenStreetMap contributors · 3D imagery: Google via Cesium ion
