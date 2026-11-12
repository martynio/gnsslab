# Satellite Orbit Explorer 🛰️

**An interactive orbital-mechanics workshop built in Python and Google Colab during Space Career Week 2026 — GNSS Academy.**

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/martynio/gnsslab/blob/main/Satellite_Orbit_Explorer.ipynb)
![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)
![Google Colab](https://img.shields.io/badge/Notebook-Google%20Colab-F9AB00?logo=googlecolab&logoColor=white)
![Status](https://img.shields.io/badge/Workshop-Completed-2ea44f)

> **From public orbital elements to equations, ground tracks, satellite velocity, and Google Earth exports.**
> This is an **educational two-body model**, not a precise orbit predictor or an operational navigation tool.

## Project at a glance

The notebook takes CelesTrak General Perturbations (GP) JSON records, converts their named OMM orbital elements into Keplerian model units, and propagates four real satellites across a shared 24-hour UTC window. It visualizes their motion above the Earth, summarizes orbital properties, and exports files for inspection in Google Earth Pro.

**Original workshop run:** 10–11 November 2026 (UTC), **5-second step**, **17,281 samples per satellite**. The original input snapshot was retrieved on **10 November 2026 at 16:30:32 UTC**. These dates are part of the supplied workshop data, **not live satellite positions**.

## What the workshop data reveals

| Satellite | NORAD | Orbit period | Perigee / apogee altitude | Inclination | Eccentricity |
|:--|--:|--:|--:|--:|--:|
| PULSAR-0 | 64541 | 1.578 h | 500 / 506 km | 97.4838° | 0.00040752 |
| GSAT0225 (GALILEO 29) | 59598 | 14.078 h | 23,217 / 23,227 km | 55.2189° | 0.00015905 |
| SES-5 | 38652 | 23.935 h | 35,778 / 35,794 km | 0.0509° | 0.00019102 |
| QZS-2 (MICHIBIKI-2) | 42738 | 23.940 h | 32,650 / 38,935 km | 39.2498° | 0.07452088 |

*Rounded values are from the completed workshop run (the notebook computes the underlying values). Perigee and apogee are heights above the simplified spherical Earth, not distances from its centre.*

Some highlights:

- **PULSAR-0:** retrograde, nearly polar orbit; many ground passes shift across longitudes during a day.
- **Galileo 29:** medium Earth orbit with a period of approximately 14.08 hours.
- **SES-5:** nearly equatorial geosynchronous orbit; its ground track stays close to **5°E**.
- **QZS-2:** inclined, eccentric geosynchronous orbit; its ground track forms an asymmetric figure-eight, with a conspicuous speed difference between perigee and apogee.

**Data freshness warning:** QZS-2 has an epoch of **17 October 2026**, over 23 days before the shared window. Its plotted trajectory is useful for learning, **not** a reliable reconstruction of its actual position on 10 November.

## Open and run

**[Launch the notebook in Google Colab](https://colab.research.google.com/github/martynio/gnsslab/blob/main/Satellite_Orbit_Explorer.ipynb)**

1. Open the notebook and connect to a Python runtime.
2. Leave `LIVE_GROUP = None` in Stage 1 to reproduce the **original workshop snapshot and quiz**.
3. Run cells from top to bottom. Stage 2 looks for `orbital_elements.json`, `data_manifest.json`, and `world_map.txt` in `/content/satellite_orbits/in`.
4. If those files are absent, the notebook requests `orbit_workshop_inputs.zip`: upload the **original workshop input ZIP** to the Colab **Files** panel under `/content`. The notebook extracts only the required three files. You can also assemble the ZIP locally from the three input files stored at the root of this repository.
5. Run the final export cell and then the separate download cell to obtain `satellite_orbit_outputs.zip`.

Colab runtimes are temporary: generated data under `/content` can disappear when the session ends. If you already have the notebook open, a new GitHub commit will not automatically refresh that tab; reopen the current revision before rerunning it.

### Generated outputs

| Filename | Purpose |
|:--|:--|
| `ground_tracks.png` | Ground tracks and segmented coastlines, using small points to avoid artificial dateline-crossing lines |
| `velocity_timeseries.png` | Orbital speed versus UTC, with one panel per satellite |
| `orbit_tracks_ecef.kml` | Earth-fixed 3D orbital paths in KML 2.2, with altitudes in **metres** |
| `orbit_animation.kml` | Time-tagged `gx:MultiTrack` ground tracks, sampled every 60 seconds and split across the ±180° longitude boundary |
| `orbit_report.txt` | Source metadata, analysis window, model limitations and reusable orbital tables |

The five deliverables are packaged into **`satellite_orbit_outputs.zip`**. Open the two KML files in **Google Earth Pro** via *File → Open*. For the animation, use its **time slider/play controls** after loading `orbit_animation.kml`.

## How it works

| Stage | Purpose |
|:--|:--|
| 1 — Configuration | Imports, model constants, file paths and analysis settings |
| 2 — Input | Load and validate CelesTrak GP JSON with OMM field names |
| 3 — Elements | Convert epochs, angles and mean motion; infer semi-major axis using Kepler's third law |
| 4 — UTC grid | Create one common, inclusive time grid and flag stale epochs |
| 5 — Propagation | Solve Kepler's equation by Newton–Raphson; compute Earth-fixed position, altitude and vis-viva speed |
| 6 — Summary | Report period, perigee, apogee, eccentricity, inclination and speed extrema |
| 7 — Plots | Draw a coastline-based ground-track map and velocity time series with Matplotlib |
| 8 — Export | Write KML, an analysis report and a downloadable output ZIP |

### Basic mathematics

- **Mean motion:** `n = 2π × MEAN_MOTION / 86400` (radians per second).
- **Semi-major axis:** `a = (μ / n²)^(1/3)` (kilometres).
- **Kepler's equation:** `M = E − e sin(E)`, solved iteratively.
- **Orbital speed (vis-viva):** `v = √[μ(2/r − 1/a)]` (km/s).
- **Earth rotation:** IAU-1982 Greenwich Mean Sidereal Time approximation is used to convert inertial positions to Earth-fixed coordinates.

The notebook also checks the GMST reference value at J2000, the Kepler-solver residual, the expected Galileo 29 period, and (for the original snapshot) SES-5's longitude range.

## Optional live CelesTrak group

After the original workshop is complete, change Stage 1 to:

```python
LIVE_GROUP = "STARLINK"
```

The optional cell **immediately after Stage 2** requests CelesTrak GP JSON for that group, validates it, caches the response locally for **two hours**, and selects the **four lowest NORAD catalog IDs**. This changes `sats` and `source_note`; downstream calculations then use the selected records.

CelesTrak asks users to avoid frequent polling. The cache is local to the current Colab runtime, so a fresh runtime may require a new request. The workshop explanations in certain Markdown cells still describe the **original four-satellite snapshot** and should not be mistaken for analysis of a new live selection. Likewise, the certification quiz refers only to the snapshot.

## Accuracy and limitations

The input GP values are **SGP4 mean elements**. Directly treating them as a Keplerian two-body orbit is an approximation: the notebook ignores perturbations, drag and other forces, uses a spherical Earth for latitude/altitude, and applies a simplified Greenwich rotation. There is **no SGP4 propagation or precision Earth-orientation model** here. Results are intended for education and visualization, **not tracking, collision assessment, navigation or safety-critical decisions**.

The input files are included for reproducibility:

- [orbital_elements.json](orbital_elements.json) — original CelesTrak GP JSON records
- [data_manifest.json](data_manifest.json) — retrieval timestamps, source URLs and provenance notes
- [world_map.txt](world_map.txt) — segmented coastline coordinate pairs (latitude first)
- [Satellite_Orbit_Explorer.ipynb](Satellite_Orbit_Explorer.ipynb) — self-contained notebook

## Credits

Created by **Martyn Białek** as a hands-on learning project during **Space Career Week 2026**, following the **GNSS Academy Satellite Orbit Explorer** workshop.

Orbital data: [CelesTrak](https://celestrak.org/) (source URLs and capture metadata in `data_manifest.json`). Workshop inspiration and learning materials: [GNSS Academy](https://gnssacademy.com/). Map source and any redistribution terms should be checked before reusing the coastline dataset outside this educational context.

*This is an independent educational implementation, not an official GNSS Academy or CelesTrak software product.*
