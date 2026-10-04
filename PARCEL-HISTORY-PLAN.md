# Parcel History & Roll Lineage Plan

Track how MAO assessment parcels change over time: prior geometry and area, when each change happened, and prior roll numbers for subdivisions and consolidations. The main use is **matching sales to the parcel as it existed on the sale date**. A secondary use is showing prior outlines on the map.

## Current state of this repo

- `MBAssessmentParcels.qmd` is a manual, one-shot download to `data/mb-assessment-parcels-{date}.parquet`. Those files are gitignored and never committed.
- `map-mb-assessment-parcels.qmd` reads only the newest parquet.
- Nothing in the repo detects changes. Nothing calls `st_area()` and nothing compares snapshots.
- The nightly MAO scan, which notes "deep changes", lives only in the local folder `D:\Dropbox\ClaudeCode\MBOpenData`. It is not in this repo.

**First step:** find the nightly scan and confirm what it keeps.
- Full nightly snapshots with geometry: history can be rebuilt backwards from them.
- Only a log of differences without geometry: prior shapes can be captured from now on only.

## Source facts (checked 2026-10)

The source is `https://services.arcgis.com/mMUesHYPkXjaFGfS/arcgis/rest/services/ROLL_ENTRY/FeatureServer/0`.

- `editFieldsInfo` is `NULL`, so editor tracking is off.
- The schema has no date-type fields.
- `editingInfo$lastEditDate` is about 2026-09-23. It applies to the whole layer and most likely marks a bulk refresh. It is not a per-parcel date.
- So no per-parcel change date comes from MAO. Change dates come from the nightly scans: the change window is `(last_seen_prior, detected_date]`, which is at most 1 day with nightly scans.
- **Still to check:** run `sapply(meta$fields, \(f) f$name)` and look for a prior or parent roll field (`Prior_Roll`, `Parent_Roll`, `Old_Roll`, ...). If one exists, it is the authoritative lineage source.

## Table 1: `parcel_history` (append-only, one row per version of each roll)

| Column | Notes |
|---|---|
| `Roll_No_Txt` | service roll number |
| `geometry` | that version's polygon |
| `area_m2` | `st_area()` in EPSG:26914 (UTM 14N) |
| `geom_hash` | digest of the WKB |
| `valid_from` | first scan showing this version |
| `valid_to` | first scan showing the next version; `NA` = current |
| `last_seen_prior` | last scan before `valid_from` |
| `change_type` | `new` / `reshaped` / `retired` |

A parcel counts as **reshaped** only when the hash differs **and** the area changes beyond a tolerance (about 1% or 5 m²). The tolerance filters out re-digitizing and vertex noise.

## Table 2: `parcel_lineage` (many-to-many)

| Column | Notes |
|---|---|
| `parent_roll` | prior roll |
| `child_roll` | new roll |
| `relation` | `subdivision` (1→many), `consolidation` (many→1), `renumber` (1→1, same shape), `realignment` (many→many) |
| `overlap_pct_parent` | share of the parent's area in this child |
| `overlap_pct_child` | share of the child's area that came from this parent |
| `last_seen_prior`, `detected_date` | change window |
| `source` | `mao_field` or `spatial` |

**Spatial fallback, used when MAO has no prior-roll field:**
- Intersect the rolls retired in a window with the rolls that appeared in the same window, in EPSG:26914.
- Keep a pair only if the overlap is at least about 5% of the child's area. This drops slivers.
- Also check `reshaped` rolls against adjacent new rolls. MAO may keep the parent roll on the remainder lot, so that roll isn't retired.
- A roll retired with no geometric successor (for example a road closure) is logged as `retired` with no child.

## Sales use

- Join a sale to the version where `valid_from <= sale_date` and (`valid_to` is NA or `valid_to > sale_date`).
- Follow the lineage recursively, child → parent, to find the lot as transferred.
- This flags sales of land that was later split or merged, so adjustments aren't based on the current lot size.

## Map display (later)

- In `map-mb-assessment-parcels.qmd`, add a toggleable dashed `add_line_layer` of prior outlines for parcels where `valid_to` is set.
- Tooltip: "Changed between X and Y · area A → B m² · prior roll(s): …".
- Never label the detection date as the actual change date.

## Stack

R, sf, arrow (GeoParquet), dplyr, httr2, digest.
