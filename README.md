# ETL Master

## Purpose

Build and maintain an evolving technology-company dataset while developing practical ETL skills through real-world data.

The project starts with Excel and Power Query. Additional data sources, tools, and processes will be introduced gradually as practical needs arise.

## Initial data source

| Property | Value |
| --- | --- |
| Source | Y Combinator Companies (YC OSS, unofficial API) |
| Endpoint | https://yc-oss.github.io/api/companies/all.json |
| Format | JSON |
| ETL tool | Power Query |
| Initial destination | Microsoft Excel |

Source documentation: https://github.com/yc-oss/api

## Working principles

- Understand the source and its structure before transforming data.
- Keep transformations reproducible and document significant decisions and data-quality checks.
- Develop the project incrementally; introduce structure only when justified by actual work.
- Use GitHub to track documentation and transformation logic.

## Current status

- [x] Create the GitHub repository.
- [x] Document the project purpose and initial data source.
- [x] Connect to the JSON source using Power Query in Excel.
- [x] Inspect the source structure and expand records into columns.
- [x] Create a separate company-tag query using Reference and list expansion.
- [x] Verify the company ID key in the observed 6,408-row snapshot.
- [x] Detect duplicate company/tag pairs and inspect referential integrity.
- [x] Create company-region association table.
- [x] Design query groups: Staging, Data Tables, Quality Control.
- [x] Build industry staging with List.Positions and List.Zip.
- [x] Build two-level industry dictionary and direct-assignment bridge.
- [x] Define industry QC and coverage checks; user reports creation of four additional QC queries.
- [x] Finish the structural modeling learning stage (subject to final saved-workbook verification).
- [x] Verify saved 16-query inventory, company and tag corrections, and four new QC outputs.
- [ ] Confirm remaining connection-only QC outputs, final analytical column selection and load settings.
- [ ] Verify full refresh.
- [ ] Data Profiling.
- [ ] Data Cleaning & Transformation.
- [ ] Load final analytical tables into Excel / Data Model.
- [ ] Data Analysis.

## Documentation

- [Data sources](DATA_SOURCES.md) — source catalog.
- [Extract](EXTRACT.md) — connection and JSON extraction.
- [Data structuring](STRUCTURE.md) — record and list transformations.
- [Data quality](DATA_QUALITY.md) — validation procedures and observed results.

Current learning stage: Data Structuring completed by agreement; next topic is Data Profiling. The agreed model uses a two-level hierarchy based on list order, and a bridge that permits M:N but currently extracts one direct classification per classified company.

Verified baseline: `yc-data(1)(1).xlsx`, inspected 2026-11-25. All 16 queries saved; four new industry QC outputs contain zero exception rows. Actual groups: 2 Staging, 6 Data Tables, 8 Quality Control (Companies_Without_Industry is in Data Tables; intended 2/5/9). Saved tables: 6,408 companies, 11,408 industry staging rows, 59 dictionary nodes, 6,408 direct assignments. Eight queries have worksheet loads; no embedded Data Model. Live refresh and connection-only QC results were not independently executed. The older 12-query workbook is historical, not the latest baseline.
