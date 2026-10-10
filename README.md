> [!IMPORTANT]
> **Consolidated into [SurveySuite](https://github.com/crieck2010/survey-suite)**  
> This engine has been consolidated into the unified [SurveySuite monorepo](https://github.com/crieck2010/survey-suite). Active development, bug fixes, releases, and unified testing now live in `survey-suite`. This repository is preserved as an archive.

# survey-geodesy

A pure-Python, dependency-free **geodetic computation engine** for surveying: reference ellipsoids, geodetic/ECEF conversions, Helmert datum transformations, Transverse Mercator and UTM projections, and Vincenty geodesic distances.

This is module four of a larger surveying + remote sensing suite. It is engine-only on purpose — no UI, no file I/O — so it can be imported by scripts, embedded in a desktop app, or installed as a library.

## Features

- **Ellipsoids** — WGS84, GRS80, Clarke 1866, Airy 1830 with derived parameters (b, e², e′²)
- **Geodetic ↔ ECEF** — closed-form forward, iterative inverse
- **Helmert transforms** — 7-parameter (position-vector convention) and 3-parameter datum shifts, with parameter inversion
- **Transverse Mercator** — forward/inverse projection (Snyder series, mm-level near the central meridian)
- **UTM** — zone lookup and lat/lon ↔ UTM wrappers on WGS84
- **Vincenty inverse** — geodesic distance plus forward and back azimuths

## Installation

Requires Python 3.9+.

```bash
# From source
pip install .

# Editable install (for development)
pip install -e .
```

## Quickstart

```python
from geodesy import latlon_to_utm, geodesic_inverse, geodetic_to_ecef, HelmertParams, helmert_7param

# UTM
e, n, zone, hemi = latlon_to_utm(40.0, -75.0)
print(f"{e:.3f} E, {n:.3f} N, zone {zone}{hemi}")
# 500000.000 E, 4427757.219 N, zone 18N

# Geodesic distance and azimuths
dist, az12, az21 = geodesic_inverse(40.0, -75.0, 41.0, -74.0)
print(f"{dist:.3f} m, forward {az12:.4f}°, back {az21:.4f}°")
# 139698.755 m, forward 37.0333°, back 217.6828°

# Datum shift on ECEF coordinates
x, y, z = geodetic_to_ecef(40.0, -75.0, 100.0)
x2, y2, z2 = helmert_7param(x, y, z, HelmertParams(dx=1.0, dy=-2.0, dz=3.0))
```

## API reference

### `geodesy.ellipsoids`

| Name | Description |
|---|---|
| `Ellipsoid(name, a, f)` | Reference ellipsoid; exposes `b`, `e2`, `ep2`, `inv_f` |
| `WGS84`, `GRS80`, `CLARKE1866`, `AIRY1830`, `ELLIPSOIDS` | Built-ins and registry |

### `geodesy.conversions`

| Function | Description |
|---|---|
| `geodetic_to_ecef(lat, lon, h, ellipsoid=WGS84)` | → `(x, y, z)` in metres |
| `ecef_to_geodetic(x, y, z, ellipsoid=WGS84)` | → `(lat_deg, lon_deg, h)` |

### `geodesy.helmert`

| Function | Description |
|---|---|
| `helmert_7param(x, y, z, params)` | Position-vector Helmert; rotations in arcseconds, scale in ppm |
| `helmert_3param(x, y, z, dx, dy, dz)` | Translation-only shift |
| `HelmertParams(...).inverse()` | Negated parameters for the reverse transformation |

### `geodesy.transverse_mercator` / `geodesy.utm`

| Function | Description |
|---|---|
| `tm_forward(lat, lon, lon0, k0=1.0, false_easting=0.0, false_northing=0.0, ellipsoid=WGS84)` | → `(easting, northing)` |
| `tm_inverse(easting, northing, lon0, ...)` | → `(lat_deg, lon_deg)` |
| `utm_zone(lon)` | Zone number 1–60 |
| `latlon_to_utm(lat, lon, zone=None)` | → `(easting, northing, zone, hemisphere)` |
| `utm_to_latlon(easting, northing, zone, hemisphere="N")` | → `(lat_deg, lon_deg)` |

### `geodesy.vincenty`

| Function | Description |
|---|---|
| `geodesic_inverse(lat1, lon1, lat2, lon2, ellipsoid=WGS84)` | → `(distance_m, fwd_azimuth_deg, rev_azimuth_deg)`; reverse azimuth points from point 2 back to point 1 |

## Accuracy notes

- The TM series implementation agrees with pyproj to better than a millimetre across tested UTM zones; the inverse round-trips to a few millimetres.
- Vincenty distances agree with pyproj's `Geod.inv` to ~0.002 mm on tested lines (nearly antipodal points raise `ValueError` instead of diverging).
- Helmert inversion by negated parameters is exact for the linearised form; residuals are far below a millimetre for realistic parameter magnitudes.

## Conventions

- Angles in degrees unless noted; distances in metres.
- Helmert rotations are in arcseconds, scale in parts per million, following the position-vector convention.
- All functions are pure (no I/O, no global state) and raise `ValueError` on invalid input.

## Testing

No test-runner dependency — stdlib `unittest` only:

```bash
python -m unittest discover -s tests
```

19 tests; UTM and geodesic reference values were cross-checked against pyproj (used as a test-time reference only — the engine has no dependency on it).

## Project structure

```
survey-geodesy/
├── src/geodesy/
│   ├── __init__.py            # exports + __version__
│   ├── ellipsoids.py          # reference ellipsoids
│   ├── conversions.py         # geodetic <-> ECEF
│   ├── helmert.py             # datum transformations
│   ├── transverse_mercator.py # TM forward/inverse
│   ├── utm.py                 # UTM wrappers
│   └── vincenty.py            # inverse geodetic problem
├── tests/test_geodesy.py
├── pyproject.toml
├── README.md
├── CHANGELOG.md
└── LICENSE
```

## Roadmap

Part of the surveying + remote sensing suite alongside `survey-cogo`, `survey-levels`, and `survey-adjust`. Planned siblings: `survey-raster` (remote sensing), `survey-pointcloud` (LiDAR/DEM/volumes), `survey-gnss` (RINEX/baselines) — all tied together by a `survey-suite` desktop app. Future geodesy work: State Plane zone tables, Lambert conformal conic, grid-based datum shifts (NADCON-style), geoid models.

## License

MIT — see [LICENSE](LICENSE).

