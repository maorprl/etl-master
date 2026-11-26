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
- [x] Data Profiling (learning and user-reported practice).
- [x] Data Structure: headers, grain, empty rows & columns.
- [x] Data Types: Change Type, Using Locale, Type Conversion (learning completed; full schema audit pending).
- [x] Text Cleaning (learning completed; actual application and QC results pending verification).
- [ ] Data Cleaning & Transformation — next: Text Standardization.
- [ ] Load final analytical tables into Excel / Data Model.
- [ ] Data Analysis.

## Documentation

- [Data sources](DATA_SOURCES.md) — source catalog.
- [Extract](EXTRACT.md) — connection and JSON extraction.
- [Data structuring](STRUCTURE.md) — record and list transformations.
- [Data quality](DATA_QUALITY.md) — validation procedures and observed results.
- [Data transformation](DATA_TRANSFORMATION.md) — data types, conversion logic and cleaning lessons.

Current learning stage: Data Profiling, Data Structure, Data Types and Text Cleaning completed as learning topics. Next topic: Text Standardization. The agreed model uses a two-level hierarchy based on list order, and a bridge that permits M:N but currently extracts one direct classification per classified company.

Verified baseline: `yc-data(1)(1).xlsx`, inspected 2026-11-25. All 16 queries saved; four new industry QC outputs contain zero exception rows. Actual groups: 2 Staging, 6 Data Tables, 8 Quality Control (Companies_Without_Industry is in Data Tables; intended 2/5/9). Saved tables: 6,408 companies, 11,408 industry staging rows, 59 dictionary nodes, 6,408 direct assignments. Eight queries have worksheet loads; no embedded Data Model. Live refresh and connection-only QC results were not independently executed. The older 12-query workbook is historical, not the latest baseline.

Latest learning checkpoint (2026-11-25, after the verified baseline): user worked on removing Column1. prefixes in YC_Companies, updating its dependent Companies_Without_Industry check, and converting launched_at to a date. These later edits are user-reported/discussed, not independently verified in a newly saved workbook. No full refresh, final 30-column type audit or final load configuration has been verified.

Latest learning checkpoint (2026-11-26): completed Text Cleaning, including dynamic Trim on approved typed text columns, repeated spaces, special/invisible Unicode, control characters, character selection/removal, targeted text replacement, pre-cleaning diagnostics and post-cleaning validation. A reusable Text_Quality_Check query was proposed, not implemented or verified. No new workbook was inspected and no text-cleaning transformations, issue counts or live refresh results are claimed. See DATA_TRANSFORMATION.md section 2 and DATA_QUALITY.md section 11. Next lesson: Text Standardization.
