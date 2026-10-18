# Data in this repository

Everything under `data/raw/` is real, public data, committed on purpose and **never edited**. Where it came from, its license, and exactly what was changed for this course:

The names of places in these files are their publishers'. Their use in this course says nothing about the political status of any place.

## EU-27 country codes across three sources

**Made for this course.** The 27 EU member states (as of 2020), one row each: the code Eurostat uses (`geo`,
which is ISO 3166-1 alpha-2 except Greece, `EL`), the ISO 3166-1 alpha-3 code Our World in Data uses
(`iso_code`), the World Bank's country `id`, and the English name. Written by hand from the three sources' own
documentation; checked by `check_values.py` in HW3's solution against every source. **License:** CC BY 4.0,
as the course materials.

## Files

| File | Bytes | SHA-256 |
|---|---:|---|
| `country_codes.csv` | 550 | `939e982ec15ae39c…` |

## Eurostat — municipal waste (env_wasmun) and greenhouse-gas emissions (env_air_gge)

*This repository holds only the files in the table below. The paragraphs describe the whole course subset, of which the other files are in other projects.*

**Source:** Eurostat, dissemination API (SDMX 2.1, TSV). Municipal waste by waste management operations,
`env_wasmun`: `https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/data/env_wasmun/?format=TSV&compressed=false&startPeriod=2010` (fetched 2026-09-19, SHA-256 `efe7d4fa295909d0…`). Greenhouse gas emissions by source sector, `env_air_gge`:
retrieved through Eurostat's asynchronous extraction of the full dataset from 2010 (the synchronous request for a
file this size is queued by Eurostat, and the old bulk-download URL is retired). Code lists for the dimensions
and the observation flags: `https://ec.europa.eu/eurostat/api/dissemination/sdmx/2.1/codelist/ESTAT/<LIST>?format=TSV`.

**License:** Eurostat data are reusable under **CC BY 4.0** (Eurostat copyright and reuse policy). Attribution:
*Source: Eurostat, env_wasmun / env_air_gge*, © European Union.

**Changes made for this course:** `env_wasmun.tsv` is unmodified. `env_air_gge_ghg.tsv` keeps only the rows whose
pollutant is `GHG` (all greenhouse gases in CO₂ equivalent), in both units, every source sector, every country;
the header line and every kept row are byte-for-byte as Eurostat published them. The four `codelist_*.tsv` files
are Eurostat's labels for the codes, unmodified.

**How to read the raw files:** the first column packs several dimensions separated by commas (its header says
which, e.g. `freq,unit,airpol,src_crf,geo\TIME_PERIOD`); each year is a column; a value may carry a flag after a
space (`628 e`); `:` means not available and may itself carry a flag (`: m`). Flag meanings: `codelist_obs_flag.tsv`.

## Files

| File | Bytes | SHA-256 |
|---|---:|---|
| `codelist_obs_flag.tsv` | 2,417 | `5631f76c7c9b8e00…` |
| `codelist_wst_oper.tsv` | 2,763 | `9c54656e6ad17c80…` |
| `env_wasmun.tsv` | 65,823 | `efe7d4fa295909d0…` |

## World Bank — World Development Indicators API, cached pages

*This repository holds only the files in the table below. The paragraphs describe the whole course subset, of which the other files are in other projects.*

**Source:** World Bank Indicators API v2 (https://api.worldbank.org/v2). Requests:
`/country/all/indicator/SP.POP.TOTL?format=json&date=2010:2024&per_page=1000&page=N` (population) and the same
for `NY.GDP.MKTP.PP.KD` (GDP, PPP, constant 2021 international dollars), pages 1 to 4 each; and
`/country?format=json&per_page=400&page=1`, the country metadata. First page: `https://api.worldbank.org/v2/country/all/indicator/SP.POP.TOTL?format=json&date=2010:2024&per_page=1000&page=1` (fetched 2026-09-19, SHA-256 `706c1f6ed0f69fb8…`).

**Response shape:** every file is a two-element JSON array. Element 0 is metadata (`page`, `pages`, `per_page`,
`total`). Element 1 is the records. **Each file is one page**: page 1 holds 1,000 of the 3,975 records. The
metadata file's `region` field is `"Aggregates"` for rows that are groups of countries (`WLD`, `EUU`, income
groups); every other row is one place.

**License:** **CC BY 4.0**. Attribution: *World Bank, World Development Indicators*.

**Changes made for this course:** none. Each file is one API response exactly as received.

## Files

| File | Bytes | SHA-256 |
|---|---:|---|
| `SP.POP.TOTL_page1.json` | 210,269 | `706c1f6ed0f69fb8…` |
| `SP.POP.TOTL_page2.json` | 197,327 | `649de57d4a1ebad7…` |
| `SP.POP.TOTL_page3.json` | 197,141 | `4039e5bf43ba298e…` |
| `SP.POP.TOTL_page4.json` | 194,617 | `542c48f24a13b6c2…` |
