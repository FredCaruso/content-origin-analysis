# Content Origin Analysis

Academic research project — University of São Paulo (USP), Brazil.

## What this is

A reproducible Python pipeline that measures **travelability**: how far
film and television content produced in a given country reaches beyond its
domestic market.

For each title, travelability is defined as the number of distinct foreign
countries in which the title appeared in a publicly published weekly Top 10
ranking, excluding its country of origin. Titles are aggregated by country
of origin to compare distributions.

The research question: *does content from some producing countries reach
international audiences systematically more often than content from others?*

## Why the TMDB API is used

Public viewership rankings contain title names only — no metadata. The TMDB
API is used to enrich each title with:

- `original_language`
- `origin_country`
- `production_countries`
- genre
- release date

This metadata is what allows each title to be classified by country of
origin. No other use is made of the API.

### Classification rule

The three origin fields frequently disagree — a Korean-language series
produced under a US-based entity may list `US` in `production_countries`.
The pipeline therefore applies an explicit priority rule:

1. `original_language`, mapped to an anchor country
2. if `original_language == "en"` → fall back to `origin_country`
3. if still ambiguous → `production_countries[0]`
4. otherwise → `UNRESOLVED` (counted and reported, never silently dropped)

A robustness check recomputes all results using `production_countries` as
the primary criterion and reports how many titles change classification.

## Method notes

- **Denominator:** the full universe of titles, including those that never
  charted, so the analysis estimates a genuine base rate rather than
  conditioning the sample on success.
- **Statistics:** medians, quartiles, full distributions and the share of
  titles with zero foreign reach. Means are not reported in isolation — the
  distribution is heavily right-skewed and a mean would be dominated by a
  handful of outliers.
- **Limitation:** results are descriptive and show association, not
  causation. Country of origin is confounded with production budget, slate
  size and marketing priority.

## API usage practices

- All API responses are cached to local disk; the collection runs once.
- Requests are rate-limited well below the documented threshold, with
  backoff on HTTP 429.
- No TMDB data is redistributed, resold, or served to third parties.
  Outputs are aggregate statistics and charts only.
- Credentials are read from a local `.env` file, which is never committed.

## Setup

```bash
cp .env.example .env     # add your TMDB read access token
python3 tmdb_check.py    # validates the credential
```

## Attribution

This product uses the TMDB API but is not endorsed or certified by TMDB.

## Licence and scope

Non-commercial, academic use only.
