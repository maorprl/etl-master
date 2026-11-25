# Data Quality

Living protocol for reproducible data-quality checks. Separate detection, diagnosis, remediation and verification.

## 1. Structural integrity and keys

| Check | Expected result | Power Query method |
| --- | --- | --- |
| Key completeness | No nulls or errors | Column Quality / Column Profile |
| Primary key uniqueness | One row per ID | Column Distribution; Group By + Count Rows + filter > 1 |
| Composite key uniqueness | One row per intended key combination | Group By on all key columns + Count Rows + filter > 1 |
| Referential integrity | No child records without a parent | Left Anti Merge (child first), or Right Anti (parent first) |
| Key type compatibility | Matching data types | Change Type / inspect types |
| Grain and row counts | Counts consistent with intended transformation | Compare full row counts before and after |

## 2. Profiling procedure

In View enable Column Quality, Column Distribution and/or Column Profile. Change the scope from top 1,000 rows to entire data set. Quality reports Valid, Error and Empty. Valid does not establish semantic correctness or absence of blank strings. Distinct counts different values; Unique counts values occurring exactly once. For a complete, non-null, error-free primary key expect Count = Distinct = Unique. In this workbook distribution initially appeared blank for an Any-type ID; after changing it to Whole Number it displayed correctly. This is an observed workaround, not a proven general cause.

## 3. Persistent duplicate checks

1. Reference the data query (not Duplicate).
2. Retain all columns of the key under test using Remove Other Columns.
3. Home > Group By. Basic for one key, Advanced for composite keys.
4. Group on every key column, select Count Rows, name it row_count.
5. Number Filters > Greater Than > 1. Do not select only currently observed counts from the checklist.

Zero rows = no duplicate keys detected; nonzero rows identify duplicate key groups and multiplicity. Keep Duplicates is an alternative that returns duplicate records rather than summarized key groups. Do not delete key fields before grouping; retaining them afterward supports diagnosis. To recover removed fields inspect Applied Steps and edit/remove the applicable removal step, checking downstream dependencies.

## 4. Remediation

Investigate whether duplicates originate in source company rows, repeated nested values or transformation logic. For a company-tag association table whose grain is one row per distinct company/tag pair, select both key fields and use Home > Remove Rows > Remove Duplicates. Never deduplicate by company ID or tag alone in that table. Refresh the QC query to verify the result. A QC reference downstream of the deduplication verifies the cleaned result, not the frequency of source anomalies; a pre-cleaning QC branch would be required to monitor raw duplicates. Do not blindly deduplicate the company table by ID.

## 5. Referential integrity

Reference the child table. Home > Merge Queries; select parent table and select the matching ID column in both previews (OK is disabled until keys are selected). Choose Left Anti (rows only in first). Reversing the tables and choosing Right Anti is equivalent. Right Outer is not equivalent without additional filtering. Expected result: zero orphan rows. Cardinality does not prescribe left/right join placement; the question determines it. Anti join verifies existence, not parent uniqueness.

Observed M expression from the workbook:
```powerquery
= Table.NestedJoin(Source, {"Column1.id"}, YC_All_Companies, {"Column1.id"}, "YC_All_Companies", JoinKind.LeftAnti)
```

## 6. General JOIN/Merge validation protocol

Before joining: define grain, check uniqueness where required by intended cardinality, missing keys, compatible types, unmatched keys, expected relationship (1:1, 1:N, N:1 or N:N).
After joining: verify row counts, unintended fan-out, unmatched records and control totals appropriate to the grain. Apply comparable checks to grain-changing operations such as list expansion. Duplicates on the many side are not inherently errors.

## 7. SRC-001 — observed results, 2026-11-25

| Check | Result | Evidence / caveat |
| --- | --- | --- |
| Companies ID quality | PASS | User reported 100% Valid; whole-data profiling used |
| Companies ID uniqueness | PASS | Profile showed Count = Distinct = Unique = 6,408; ID Whole Number |
| Tags column quality | PASS reported | User reported 100% Valid; blank strings not separately checked |
| Company/tag composite key before remediation | FAIL | Group By revealed repeated pairs (count 2 visible) |
| Company/tag deduplication | Procedure provided | Final post-remediation zero-row result not separately supplied |
| Company_Tags_Orphans | PASS in merge preview | 17,659 of 17,659 child rows excluded by Left Anti; zero unmatched inferred from preview |
| Companies_PK_Duplicates | Procedure provided | Final output not separately supplied |
| Company_Tag_Duplicheck | Procedure provided, repair discussed | Final key selection and zero-row output not separately supplied |
| Key type compatibility | Pending confirmation | Parent Whole Number observed; child final type not separately confirmed |

User acknowledged the final checklist; no workbook or complete final QC output was inspected. The parent key was verified at the observed snapshot, and the expected parent-to-tag relation is 1:N (a company may have zero or more tags). Source duplicate root cause, full before/after counts and refresh validation remain open.

## 8. Next

Confirm final QC outputs and both key types; organize staging, data and QC queries. Later extend the procedure to other nested lists only as needed.
