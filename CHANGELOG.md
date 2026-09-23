# Changelog

All notable changes to `survey-geodesy`. Follows semantic versioning.

## [0.1.0] - 2026-09-23
### Added
- Initial release: pure-Python, dependency-free geodetic engine.
- `ellipsoids`: WGS84, GRS80, Clarke 1866, Airy 1830 with derived
  parameters (b, e^2, e'^2).
- `conversions`: geodetic <-> ECEF (iterative inverse).
- `helmert`: 7-parameter (position-vector) and 3-parameter datum
  transformations, with parameter inversion.
- `transverse_mercator`: forward/inverse TM (Snyder series).
- `utm`: zone lookup and lat/lon <-> UTM wrappers (WGS84).
- `vincenty`: inverse geodetic problem (distance + forward/back azimuths).
- 19 unit tests (stdlib `unittest`); TM and geodesic values cross-checked
  against pyproj.
