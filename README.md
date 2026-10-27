# AI Now — CAP 6938 Final Project

A scrollytelling narrative visualization on the measurable consequences of the AI boom: capital, infrastructure, applications, public perception, and intimate use.

**Live:** [alejandroaparcedo.com/ai-now](https://alejandroaparcedo.com/ai-now/) (the Command Deco version)

Ships in two visual styles. The data, prose, and structure are identical; only the styles differ.

## Run

**Sober editorial (the canonical version) — `app/`**

```
cd app
npm install
npm run dev
```

Open http://localhost:5173.

**Command Deco (alternate — Art Deco × tactical anime UI × motorsport energy) — `app-deco/`**

```
cd app-deco
npm install
npm run dev -- --port 5174
```

Open http://localhost:5174.

## Layout

```
.
├── app/             Sober editorial version (Svelte 5 + Vite + D3)
│   ├── src/         Components, charts, styles
│   └── public/data/ Chart-ready CSVs (fetched at runtime)
├── app-deco/        Command Deco version — same structure, different styles
├── chart-data/      Source-of-truth processed datasets
└── raw-data/        Original PDFs and CSVs from primary sources
    └── MANIFEST.md  Per-chart provenance, URLs, caveats
```

## The five charts

1. **The money** — private investment in U.S. AI, 2013–2025
2. **Where it lives** — top U.S. data center metros by operational megawatts
3. **Inside the hospital** — cumulative FDA authorized AI medical devices, 1995–2025
4. **How the world feels** — Ipsos AI Monitor 2025, 30 countries
5. **In the bedroom** — Common Sense Media survey of U.S. teen AI companion use

Each chart has one editorial claim. The prose carries the argument; the chart is the evidence.

## Datasets

Five charts, five datasets. Mix of pre-aggregated reports, manually transcribed PDF tables, and one large raw list (FDA) that we processed ourselves. None of the data is synthetic.

### 1. THE MONEY — Private AI investment

| Field | Value |
|---|---|
| File | `chart-data/01-investment.csv` |
| Rows | 40 |
| Schema | `year, region, category, value_usd_billions_nominal, source_note` |
| Timeframe | 2013 through 2025 (13 years) |
| Aggregation | Pre-aggregated annual totals by Stanford HAI; we transcribed and added a generative-AI sub-category breakout for years 2022 onward |
| Original collection | Continuous tracking of private funding rounds ≥ $1.5M USD by Quid via S&P Capital IQ + Crunchbase |
| Published | Stanford AI Index Report 2026, Chapter 4 (Economy), released early 2026; chart extends through end of 2025 |
| Excludes | Publicly traded firms (Microsoft, Google, Meta), internal corporate R&D, and public-sector funding |
| Currency | All figures in nominal U.S. dollars (not inflation-adjusted) |

### 2. WHERE IT LIVES — Data center geography

| Field | Value |
|---|---|
| File | `chart-data/02-geography.csv` |
| Rows | 7 (top U.S. metros) |
| Schema | `metro, state_or_region, lat, lon, total_mw_2025, under_construction_mw, source_note` |
| Timeframe | Snapshot at year-end 2025 (with under-construction figures for 2026) |
| Aggregation | **Hand-transcribed.** CBRE publishes their report as an HTML book with no CSV download. We pulled the per-metro MW figures from the chapter pages manually |
| Original collection | CBRE's research team surveys colocation operators twice a year (H1 and H2 reports) |
| Published | CBRE *H2 2025 North America Data Center Trends*, January 2026 |
| Caveat | Silicon Valley figure reflects 2024 reporting; new construction stalled in 2025 due to power constraints |

### 3. INSIDE THE HOSPITAL — FDA AI medical devices

This chart uses three derived files from one large raw source.

| Field | Value |
|---|---|
| Raw source | `app/public/data/03-fda-raw.csv` |
| Raw rows | **1,431 individual device authorizations** (every FDA AI/ML clearance ever issued) |
| Raw schema | `Date of Final Decision, Submission Number, Device, Company, Panel (Lead), Primary Product Code` |
| Derived: cumulative | `chart-data/03-fda-cumulative.csv` — 25 rows, one per year, with `year_count` and `cumulative_count` |
| Derived: by panel | `chart-data/03-fda-by-panel.csv` — 17 rows, panel name + count + share % |
| Timeframe | 1995 through 2025 (30 years; chart x-axis displays 2015–2025 where the curve is meaningful) |
| Aggregation | **We computed both derivatives ourselves.** The cumulative-by-year was rolled up from the raw `Date of Final Decision` column; the panel breakdown was rolled up from the `Panel (Lead)` column |
| Original collection | FDA tracks every cleared/authorized AI-enabled device since 1995. List refreshed quarterly |
| Published | FDA *Artificial Intelligence Enabled Medical Device List*, March 9 2026 release |
| Caveat | FDA states the list is "not a comprehensive resource" — definitions of "AI-enabled" evolve, and devices may be added retroactively |
| Headline numbers | 1,430 cumulative through 2025 (up from 42 a decade earlier); 76.5% are radiology devices |

### 4. HOW THE WORLD FEELS — Ipsos AI Monitor

| Field | Value |
|---|---|
| File | `chart-data/04-perception.csv` |
| Rows | 31 (30 countries from 2025 wave + China carried forward from 2024 wave) |
| Schema | `country, iso3, pct_agree, year, is_us, source_note` |
| Timeframe | 2025 wave fielded March–April 2025; China figure from 2024 wave fielded April–May 2024 |
| Aggregation | **Hand-transcribed.** Ipsos publishes only PDF reports — no CSV download. Top-two-box "agree" already combined by Ipsos; we typed the per-country percentages from the PDF tables |
| Original collection | Ipsos Global Advisor online panel; 2025 wave n = 23,216; 2024 wave n = 23,685 |
| Published | *Ipsos AI Monitor 2025*, June 2025 |
| Caveat | Samples in middle-income countries (China, Brazil, Indonesia, Thailand, Türkiye) skew urban, educated, and affluent — not nationally representative |

### 5. IN THE BEDROOM — Teen AI companion use

| Field | Value |
|---|---|
| Files | `chart-data/05-teen-ai-frequency.csv` + `chart-data/05-teen-ai-stats.md` |
| Rows | 7 frequency categories (CSV) + supporting Q-by-Q stats (markdown) |
| Schema | `category, pct, bucket, source_note` |
| Timeframe | Survey fielded April–May 2025 |
| Aggregation | Pre-aggregated by NORC; we typed the values from the toplines PDF |
| Original collection | NORC at the University of Chicago, AmeriSpeak probability-based panel, fielded online; weighted to Census benchmarks |
| Sample | n = 1,060 U.S. teens ages 13–17 (nationally representative) |
| Margin of error | ±3.4 pp at 95% confidence |
| Published | Common Sense Media, *"Talk, Trust, and Trade-Offs: How and Why Teens Use AI Companions,"* July 16 2025 |
| Headline numbers | 72% of teens have used an AI companion; among those users (n = 758), 33% have chosen the chatbot over a real person for a serious conversation |

## Sources

See [`raw-data/MANIFEST.md`](raw-data/MANIFEST.md) for primary source URLs and per-file provenance.
