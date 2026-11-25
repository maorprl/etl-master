# Data Sources

Central catalog of data sources used by ETL Master.

## Source inventory

| Source ID | Name | Format | Status |
| --- | --- | --- | --- |
| SRC-001 | Y Combinator Companies | JSON | Registered |

## SRC-001 — Y Combinator Companies

| Property | Value |
| --- | --- |
| Source ID | SRC-001 |
| Source name | Y Combinator Companies |
| Provider | YC OSS (unofficial community project) |
| Original data origin | Y Combinator website's Algolia search index |
| Source type | Public web API / static JSON endpoint |
| Data format | JSON |
| Endpoint | https://yc-oss.github.io/api/companies/all.json |
| Documentation | https://github.com/yc-oss/api |
| Metadata endpoint | https://yc-oss.github.io/api/meta.json |
| Authentication | No credentials documented for public endpoint |
| Update frequency | Daily workflow, according to provider documentation |
| Cost | Public access; no fee documented |
| License / data reuse rights | Not verified; public access does not establish redistribution rights |
| Coverage | Publicly launched YC companies with pages on the YC website |
| Limitations | Unofficial source; excludes companies outside documented coverage; completeness and accuracy not independently verified |
| Date registered | 2026-11-25 |
| Status | Registered; initial extraction successful |

### Provider-reported metadata (2026-11-25)

- Companies: 6,408
- Batches: 51
- Industries: 59
- Tags: 338

These figures are a dated reference from the provider documentation, not counts independently verified by ETL Master.

### Next action

Initial extraction and structure inspection completed; see [EXTRACT.md](EXTRACT.md), [STRUCTURE.md](STRUCTURE.md) and [DATA_QUALITY.md](DATA_QUALITY.md). Company profiling showed 6,408 IDs in the observed snapshot; source completeness beyond the returned data and repeat refresh remain unverified.
