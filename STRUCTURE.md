# Data Structuring

Living documentation of structural transformations in ETL Master.

## 3.1 — List, Record and Table

| Structure | Meaning |
| --- | --- |
| List | Ordered collection of values, potentially including records or nested lists |
| Record | Collection of named fields and their values |
| Table | Rows and named columns |

### SRC-001 — observed structure

The JSON extraction returns a List of Records. Inspection of the first record (CircuitHub, id 5) showed scalar values and nested lists. Examples of list fields: `former_names`, `tags`, `tags_highlighted`, `industries`, `regions`.

Target grain: one row per company. Primary key: `id` (shown as `Column1.id` in later workbook screenshots); verified for the observed 6,408-company snapshot. See DATA_QUALITY.md.

## 3.2 — Convert List to Table

1. Return to the top-level List.
2. Select Transform > To Table.
3. Set delimiter to None.
4. Leave extra columns handling as Show as errors.
5. Click OK.

Observed result (user confirmation): a one-column table (`Column1`) containing Records.

## 3.3 — Expand Records

1. Click the expand icon in the Column1 header.
2. Select all available fields.
3. Clear Use original column name as prefix.
4. Click OK.

Observed result (user screenshot, 2026-11-25): named columns including `id`, `name`, `slug`, `former_names`, `small_logo_thumb_url`, `website`, `all_locations`, `long_description`. Nested lists remain unexpanded; visible scalar columns show the Any type indicator. The formula bar shows `Table.ExpandRecordColumn` with explicitly selected field names. The complete M expression is not visible in the screenshot.

Record expansion adds columns, whereas expanding lists into new rows may change grain. Explicitly selected fields do not automatically incorporate new fields added to the source schema. Refresh reruns the extraction and transformations; a normal refresh does not preserve historical snapshots.

## 3.4 — Nested lists and relational structuring

Three alternatives: Expand to New Rows (changes grain), Extract Values (delimited text in a single cell), or a separate child table (preserves parent grain). The selected design is a separate company-tag association table.

Procedure completed through list expansion, with user confirmation:
1. Right-click the expanded companies query in the Queries pane and choose Reference. The Reference depends on the source query result; Duplicate copies the query steps.
2. Name the new query (proposed: YC_Company_Tags).
3. Ctrl-select company ID and tags, then Remove Other Columns.
4. Expand tags using Expand to New Rows.
5. Rename fields as appropriate (recommended company_id, tag). Later screenshots and formulas show Column1.id and tags; final names may differ.

The resulting grain is initially one row per company-tag occurrence. Desired grain is one row per distinct company-tag pair. ID repetition is valid in this child table; repeated pairs are not. Preserve company ID to enable relationships. Keep the parent source intact; Reference does not guarantee only one network request or a materialized cache.

Expected model: Companies (PK id) 1 to many Company_Tags (FK company ID, composite key company ID + tag). The other observed nested lists (regions, industries, former_names, tags_highlighted) are candidates for separate child tables, not yet implemented. The later workbook uses YC_All_Companies as parent. No final Data Model relationships have been created or verified.

## 3.5 — Key and structural validation

See [DATA_QUALITY.md](DATA_QUALITY.md) for the validation and remediation protocol, observed duplicate pairs, parent key profile and anti-join result.

## Refresh behavior

Refresh reruns applied steps on the current source data. Added/removed companies and changed values flow through the query. Explicit record field selections may omit newly introduced fields; renamed/removed fields may require schema maintenance. Refresh does not retain historical snapshots. Manual Refresh All, on-open refresh and periodic refresh while Excel is open were discussed; no automatic refresh configuration has been verified.

## Validation status

- [x] Inspect an individual record.
- [x] Convert List to Table.
- [x] Expand Records into named columns.
- [x] Observe nested List values retained in the resulting preview.
- [ ] Compare full row counts before and after expansion.
- [x] Verify candidate key completeness and uniqueness for the observed snapshot.
- [x] Determine handling of tags: separate child table.
- [ ] Complete other nested-list decisions and verify final QC results.

Parent ID was explicitly assigned Whole Number. Tag pair deduplication was discussed; final verification remains open. No final load or full refresh verification has been established yet.
