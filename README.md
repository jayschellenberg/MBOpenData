# Manitoba Assessment Parcels — 3D Value Map

Interactive 3D visualization of 437,000+ property assessment parcels across Manitoba, with height encoding assessed value and color encoding municipality.

**Live map**: [jayschellenberg.github.io/MBOpenData](https://jayschellenberg.github.io/MBOpenData)

## What this shows

Every property parcel in Manitoba's provincial assessment roll (outside Winnipeg), sourced from the [Manitoba GeoPortal](https://geoportal.gov.mb.ca/datasets/8106acf39b124422a5f03a5c4e55d269_0/explore). The map includes:

- **Address search** — find any location in Manitoba
- **3D extrusions** — taller buildings = higher assessed values
- **Hover tooltips** — assessed value, roll number, municipality median for context
- **Municipality summary** — total assessed value, median, and parcel count by municipality

## Pipeline

```
ArcGIS REST API (Manitoba GeoPortal)
  → MBAssessmentParcels.qmd (batch download + GeoParquet)
  → map-mb-assessment-parcels.qmd (PMTiles + 3D map)
  → docs/index.html (GitHub Pages)
```

## Reproduce

### Prerequisites

R 4.3+ with these packages:

```r
install.packages(c(
  "httr2", "readr", "dplyr", "sf", "arrow", "geoarrow",
  "cli", "fs", "glue", "ggplot2", "scales", "stringr",
  "duckdb", "DBI", "mapgl"
))

# freestiler from r-universe (not on CRAN)
install.packages("freestiler",
  repos = c("https://walkerke.r-universe.dev", "https://cloud.r-project.org"))
```

[Quarto](https://quarto.org/) 1.4+.

### Steps

1. **Download data** — open `MBAssessmentParcels.qmd` in RStudio, set `eval: true`, and render. Downloads ~437K parcels from the ArcGIS API (~15 min).
2. **Build the map** — render `map-mb-assessment-parcels.qmd`. Generates PMTiles and the interactive HTML.
3. **Deploy** — output goes to `docs/index.html` for GitHub Pages.

## Data source

Manitoba Assessment Parcels (ROLL_ENTRY) via the [Manitoba GeoPortal ArcGIS REST API](https://services.arcgis.com/mMUesHYPkXjaFGfS/arcgis/rest/services/ROLL_ENTRY/FeatureServer/0). Data is published by the Province of Manitoba under open data terms.

## License

Code: MIT. Data: subject to Province of Manitoba open data terms.
