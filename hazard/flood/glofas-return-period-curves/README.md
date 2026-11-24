# GloFAS return period curves

Per GloFAS cell, the curve that turns a forecast discharge into a return period. The floods pipeline uses it to compute the flood depth of an alert from the GloFAS cells in its area.

- `{ISO3}_glofas_{version}_return_period_curves.tif` — one GeoTIFF per country on the GloFAS grid (0.05°), with three bands:
  - `location` and `scale`: the Gumbel parameters `mu` and `sigma` of annual peak discharge (m³/s).
  - `upstream_area`: the upstream area (m²).

The files are clipped to the country and its stations, plus 0.5°. Cells without a curve are NaN.

Source: the [GloFAS auxiliary data](https://confluence.ecmwf.int/spaces/CEMS/pages/242067380/Auxiliary+Data) that ECMWF publishes, for GloFAS v4. The station thresholds in `glofas-stations/` lie on the v4 curve of each station's cell, so curves and thresholds must stay on the same GloFAS version.

Produced by `build_glofas_return_period_curves.py` in the [NRW repo](https://github.com/rodekruis/NRW/tree/main/data/data_management/seed_data_management/flood).
