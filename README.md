# GeoZones mapping

Matches data.gouv.fr datasets that have a bounding box (`spatial.geom`) to the best-fitting
administrative zone (région, département, commune, EPCI…) using IoU scoring.

## Notebooks

Run in order:

### 1. `geozones-global.ipynb`

Downloads the [geozones archive](https://www.data.gouv.fr/api/1/datasets/r/4ec9e77d-572a-40f9-9940-ade023ce8b78)
and extracts the 6 top-level geographic containers (world, EU, France, France métro, DROM, DROM-COM).

**Output:** `zones_global.geojson`

### 2. `geozones-filtered.ipynb`

Downloads the [export-geozones dataset](https://demo.data.gouv.fr/api/1/datasets/r/7971e15c-3255-4724-b690-77171bc6d9ad)
(French territorial zones: régions, départements, EPCIs, communes) and combines it with
`zones_global.geojson`.

**Output:** `zones_filtered.geojson`

### 3. `geozones-mapping.ipynb`

Downloads the [datasets export](https://www.data.gouv.fr/api/1/datasets/r/f868cca6-8da1-4369-a78d-47463f19a9a3),
then for each dataset with a bounding box finds the best-matching zone using:
- `sjoin(predicate='intersects')` for candidate pairs
- area-ratio pre-filter (lossless, reduces ~130M → ~170K candidates)
- actual `shapely.intersection` IoU on remaining candidates
- best match must satisfy IoU ≥ 0.4 and zone area ≤ bbox area

**Output:** `geozones-mapping-output.csv` — one row per matched dataset:

| Column | Description |
|---|---|
| `id` | Dataset ID |
| `title` | Dataset title |
| `zone_id` | Matched zone ID (e.g. `fr:departement:38`, `fr:commune:75056`) |
| `iou_score` | IoU score of the match (0.4–1.0) |

Last run: **32,338 datasets matched** out of 45,424 with a bounding box.

| Level | Count |
|---|---|
| `fr:departement` | 14,657 |
| `fr:region` | 8,390 |
| `fr:commune` | 7,358 |
| `fr:epci` | 1,197 |
| `country-subset:fr` | 680 |
| `country:fr` | 56 |

### `geozones-debug.ipynb`

Interactive map to inspect individual dataset matches. Not part of the pipeline.

![](debug.png)

## Running

```bash
uv run jupyter lab
```

All downloads are skipped if the files already exist locally.
