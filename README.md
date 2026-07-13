# IRS-GEO

Downloader for an organized mirror of the IRS SOI **individual income tax
data by geographic area**:
https://www.irs.gov/statistics/soi-tax-stats-data-by-geographic-area

This repo holds the **code only** — data is downloaded on demand, either into
the repo's own (gitignored) `data/` folder or to a separate location of your
choosing. All source files are U.S. federal government works (public domain).
Data is stored as gzipped CSVs exactly as published by SOI (no
transformation); R and most tools read `.csv.gz` directly
(`readr::read_csv('file.csv.gz')`).

## Usage

```bash
Rscript download_irs_geo.R                        # -> ./data, years 2011-2022
Rscript download_irs_geo.R 2017 2023              # custom year range
Rscript download_irs_geo.R --dest /path/to/store  # separate destination
```

Budget Lab internal users: the canonical shared destination (already
populated, with a consolidated `NOTES.md` at its root) is documented
internally — pass it via `--dest`.

The script is idempotent (existing files are skipped; delete a file to
re-fetch), tolerates unpublished years (HTTP 404s skipped with a message),
and rewrites a checksummed `manifest.csv` (path, source URL, year, bytes,
md5, retrieval date) at the destination each run.

## Data layout (under the destination)

```
state/HT2/          ht2_{year}.csv.gz                 Historic Table 2: returns and income
                                                      items by state x AGI class (filers only)
state/percentile/   state_shares_{year}.csv.gz        AGI percentile data by state
                    state_shares_docguide_{year}.pdf  SOI documentation guide
county/             county_{year}_agi.csv.gz          County income data, by AGI class
                    county_{year}_noagi.csv.gz        County income data, county totals
zip/                zip_{year}_agi.csv.gz             ZIP code data, by AGI class
                    zip_{year}_noagi.csv.gz           ZIP code data, ZIP totals
manifest.csv        path, source url, year, bytes, md5, retrieval date
```

## Notes on the data

Per-family documentation of what each file is and how it changed over time —
variable additions/removals by year, disclosure-rule changes, unit anomalies,
and analysis gotchas — compiled from the SOI docguides and verified against
the files:

- [notes/ht2.md](notes/ht2.md) — Historic Table 2 (incl. the silent
  suppression-cell collapsing, the N2 exemptions→individuals relabel, PR
  joining in 2018)
- [notes/percentile.md](notes/percentile.md) — state AGI percentile shares
  (incl. the 2014/2016/2017 unit + scientific-notation anomaly and the 2018
  combined-IRA/pension one-off)
- [notes/county.md](notes/county.md) — county income data
- [notes/zip.md](notes/zip.md) — ZIP code data

The SOI documentation guides themselves are downloaded alongside the data
(`*docguide*` files in each destination folder).

## Coverage and source-naming quirks

| Family | Years available | SOI filename pattern |
|---|---|---|
| HT2 all-states CSV | 2012, 2014–2022 | `{yy}in54cmcsv.csv` (2012–17; 2013 unpublished), `18in55cmagi.csv` (2018, one-off), `{yy}in55cmcsv.csv` (2019+) |
| State percentile shares | 2013–2022 | `{yy}instateshares.csv` (+ `...docguide.pdf`) |
| County income | 2011, 2013–2022 | `{yy}incyallagi.csv` / `{yy}incyallnoagi.csv` (2012 and earlier are zip archives, not pulled) |
| ZIP code data | 2011–2022 | `{yy}zpallagi.csv` / `{yy}zpallnoagi.csv` |

Other HT2 notes: the `N2` column is *number of exemptions* through tax year
2017 and *number of individuals* from 2018 (TCJA); state rows include the 50
states, DC, and PR/"other areas" (some vintages separate PR from OA).

When SOI publishes a new year, extend the range:
`Rscript download_irs_geo.R --dest <store> 2011 2023`.

## Known consumers

- **Tax-Simulator state weights** (Budget-Lab-Yale/Tax-Simulator, `state-tax`
  branch): `state/HT2/` supplies the filer calibration targets for the split
  state weights; `county/` is the planned target source for sub-state
  (locality) weights — see `other/state_tax_research/` there.
