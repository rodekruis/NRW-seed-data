# GloFAS Station Thresholds

The `*_station_thresholds.json` files contain one record per GloFAS station. The `pcodes` field contains the admin-area place codes used for that country's flood alert configuration and runtime exposure calculation.

## Migrating Station Mappings

Mappings are migrated by the flood-specific
`migrate_glofas_station_area_mapping` script in the NRW repository. The script
transfers the existing station footprints from the old admin-area dataset to
the new admin-area dataset using spatial overlap:

```bash
cd /path/to/NRW/data
uv run python -m data_management.seed_data_management.flood.migrate_glofas_station_area_mapping \
	--old-seed-repo /path/to/old-NRW-seed-data \
	--new-seed-repo /path/to/NRW-seed-data \
	--old-seed-revision <old-seed-revision> \
	--new-seed-revision <new-seed-revision> \
	--output-directory /tmp/nrw-glofas-migration
```

The default per-area threshold is 25%: a new area must have at least 25% of its
area covered by the old station footprint. A migration is flagged for review
when the accepted new areas cover less than 90% of the old footprint. These
thresholds are recorded in each `{country}_migration_manifest.json` file.

The migration processes ETH, KEN, MWI, PHL, SSD, UGA, and ZMB. ZMB migrates
old adm3 station mappings to new adm4 areas; the other countries retain their
existing deepest admin level. Review the generated reports before copying the
threshold files into this directory.

```bash
cp /tmp/nrw-glofas-migration/*_station_thresholds.json /path/to/NRW-seed-data/hazard/flood/glofas-stations/
cp /tmp/nrw-glofas-migration/*_migration_manifest.json /path/to/NRW-seed-data/hazard/flood/glofas-stations/
```

The migration manifests preserve the old and new seed revisions, source paths,
thresholds, and output paths. Commit each manifest with its corresponding
threshold file so the one-time mapping decision remains reproducible. The
spatial migration is not a recurring forecast update; rerun it only when the
admin-area dataset or the station mappings are intentionally migrated again.
The consolidated station and country summary is available in
[`station-area-migration-report.md`](station-area-migration-report.md).

## Station Location Fixes

The pipeline reads each station's forecast in the GloFAS cell of its coordinates. In November 2026 (AB#45972), 17 stations were moved to the centre of their river cell:

- 13 stations lay on a cell boundary next to a much smaller stream, so the forecast could come from the wrong cell. Their thresholds already belonged to the river cell and are unchanged.
- 4 stations (G5142, G1966, G1967, G5368) lay in a small cell next to the river. They moved to GloFAS's own river cell (LISFLOOD X/Y in the GloFAS viewer), with the v4 thresholds of that cell.

The method and the proposal are in `data/analysis/glofas_v5_thresholds/` in the NRW repository (`results/station_location_proposal.csv`); production is updated by a Prisma data migration in the NRW repository. The mock forecasts for ETH, KEN and PHL were regenerated for the new locations.
