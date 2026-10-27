# Raw Data Manifest

Source files for the AI Now scrollytelling piece. One folder per chart.

For each file: the canonical URL, what it covers, and any known caveats. Files marked **MANUAL** could not be auto-downloaded and require manual fetch.

---

## 01 — Private investment (Chart 1, "The money")

**Primary:** Stanford AI Index 2026, Chapter 4 (Economy). Chart extends to 2025.

| File | Source | Notes |
|---|---|---|
| `AI-Index-2026-Full.pdf` | https://hai.stanford.edu/assets/files/ai_index_report_2026.pdf | Full report (37 MB). Chapter 4 is Economy. |
| `AI-Index-2025-Chapter4-Economy.pdf` | https://hai.stanford.edu/assets/files/hai_ai-index-report-2025_chapter4_final.pdf | 2025 edition Chapter 4 (kept for cross-reference; 2025 is the canonical citation for the $109B/2024 framing). |
| `OWID-private-investment-AI.csv` | https://ourworldindata.org/grapher/private-investment-in-artificial-intelligence.csv | Our World in Data mirror. **Constant 2021 USD** — does not match the report's nominal figures. Use only for sanity checks. |
| `OWID-private-investment-AI-by-focus-area.csv` | https://ourworldindata.org/grapher/private-investment-in-artificial-intelligence-by-focus-area.csv | OWID gen-AI breakout, also constant 2021 USD. |

**Manual:** AI Index canonical CSVs/XLSXs live in a Google Drive folder, not GitHub. Pull final data files from: https://drive.google.com/drive/folders/1oGKD04Iyw9EVhOTs5DsyrO568DORZnBS (2025 edition; 2026 equivalent should be linked from the 2026 report's Chapter 4 footer).

---

## 02 — Data center geography (Chart 2, "Where it lives")

**Primary:** CBRE H2 2025 metro MW figures (manual transcription required).

| File | Source | Notes |
|---|---|---|
| — | — | Nothing auto-downloadable. |

**Manual:**
- **CBRE H2 2025 North America Data Center Trends** — https://www.cbre.com/insights/books/north-america-data-center-trends-h2-2025 (HTML book, no CSV). Hand-transcribe ~10 metro MW figures: NoVA 4,039.6 MW; Atlanta 1,459.2; Dallas-Ft. Worth >1,000; Silicon Valley, Phoenix, Chicago, NY Tri-State, Hillsboro from chapter pages.
- **IM3 Open Source Data Center Atlas** (June 2025) — https://www.osti.gov/biblio/2550666 → https://data.msdlive.org/records/65g71-a4731 (record API only shows placeholder; download requires interactive UI). CSV + GeoPackage with lat/long for facilities (no MW field).
- **Loudoun County / Dominion Energy** annotation source — https://www.loudoun.gov/ArchiveCenter/ViewFile/Item/13979 ; NoVA crossed 4,900 MW Q1 2025.

---

## 03 — FDA AI medical devices (Chart 3, "Inside the hospital")

**Primary:** FDA AI-Enabled Medical Device List, 2026-03-09 release.

| File | Source | Notes |
|---|---|---|
| `FDA-AI-Enabled-Medical-Devices-2026-03-09.csv` | https://www.fda.gov/media/178541/download?attachment | 121 KB, 1,430 rows. Endpoint = **1,430**. Cumulative-by-year computed from `Date of Final Decision`. Radiology = 76.5% (1,094 / 1,430). |

**Caveats:** FDA states the list is "not a comprehensive resource"; devices are added retroactively as definitions evolve. Slug changed from `aiml-enabled` to `ai-enabled` — old links 301-redirect.

---

## 04 — Public perception (Chart 4, "How the world feels")

**Primary:** Ipsos AI Monitor 2025 (30 countries) + Ipsos AI Monitor 2024 (China 83% data point only, footnoted).

| File | Source | Notes |
|---|---|---|
| `Ipsos-AI-Monitor-2025.pdf` | https://www.ipsos.com/sites/default/files/ct/publication/documents/2025-06/Ipsos-AI-Monitor-2025.pdf | 30 countries; n=23,216; fielded Mar–Apr 2025. China not surveyed. |
| `Ipsos-AI-Monitor-2024.pdf` | https://www.ipsos.com/sites/default/files/ct/news/documents/2024-06/Ipsos-AI-Monitor-2024-final-APAC.pdf | 32 countries including China; n=23,685; fielded Apr–May 2024. Use only for the China 83% data point in the 2025-anchored chart. |

**Caveat:** Ipsos samples in China, Brazil, Indonesia, Thailand, Türkiye, etc. are urban/educated/affluent — not nationally representative.

---

## 05 — Teen AI companion use (Chart 5, "In the bedroom")

**Primary:** Common Sense Media, "Talk, Trust, and Trade-Offs" (July 16, 2025). NORC-fielded.

| File | Source | Notes |
|---|---|---|
| `CSM-Talk-Trust-Tradeoffs-Full-Report.pdf` | https://www.commonsensemedia.org/sites/default/files/research/report/talk-trust-and-trade-offs_2025_web.pdf | Full report; n=1,060 US teens 13–17; fielded April–May 2025. |
| `CSM-Talk-Trust-Tradeoffs-Toplines.pdf` | https://www.commonsensemedia.org/sites/default/files/research/talk-trust-and-trade-offs_2025_toplines.pdf | Toplines with verbatim Q1–Q10 percentages. |

**Q1 frequency-of-use breakdown (n=1,060):** Several times a day 8% / Once a day 5% / Few times a week 21% / Once a week 9% / Few times a month 9% / Tried but rarely 19% / Never used 28%.

**Q9 "AI over a person":** Among AI companion users (n=758), 33% Yes. Editorial line wording: "one in three teen *users*" — base is users, not all teens.
