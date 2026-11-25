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

Target grain: one row per company. Candidate primary key: `id`; uniqueness and completeness not yet verified.

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

## Validation status

- [x] Inspect an individual record.
- [x] Convert List to Table.
- [x] Expand Records into named columns.
- [x] Observe nested List values retained in the resulting preview.
- [ ] Compare full row counts before and after expansion.
- [ ] Verify candidate key completeness and uniqueness.
- [ ] Determine handling of nested lists according to analytical requirements.

No cleaning, explicit type assignment, final load or full refresh verification has been established yet.
