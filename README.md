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
- [ ] Confirm final QC query outputs and compatible key types.
- [ ] Organize staging, data and quality-control queries.
- [ ] Verify full refresh and load final tables into Excel / Data Model.

## Documentation

- [Data sources](DATA_SOURCES.md) — source catalog.
- [Extract](EXTRACT.md) — connection and JSON extraction.
- [Data structuring](STRUCTURE.md) — record and list transformations.
- [Data quality](DATA_QUALITY.md) — validation procedures and observed results.

Current state: Power Query preview and structural checks are underway. Final workbook loading and repeat refresh validation have not been verified.
