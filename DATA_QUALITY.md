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

## 8. Industry model validation (2026-11-25)

### Permanent QC queries agreed for this iteration

Create references in Quality Control; do not modify the data tables when diagnosing problems.

| QC query | Source | Method | Expected result | Latest status (four saved outputs verified below) |
| --- | --- | --- | --- | --- |
| Industry_PK_Duplicates | YC_Industry_Hierarchy | Keep industry_id, Group By industry_id, Count Rows, Count > 1 | 0 rows | Empty / PASS |
| Industry_Parent_Orphans | YC_Industry_Hierarchy | Filter parent_id != null; Left Anti Merge parent_id against YC_Industry_Hierarchy.industry_id | 0 rows | Empty / PASS |
| Company_Industry_Duplicates | YC_Company_Industry | Group By company_id + industry_id, Count Rows, Count > 1 | 0 rows | Empty / PASS (saved worksheet verified) |
| Companies_Without_Industry | YC_Companies | Keep Column1.id; Left Anti Merge vs YC_Company_Industry.company_id | 0 rows for this snapshot | Empty / PASS |

The dictionary PK must be checked against the dictionary, not against staging where repeated node IDs are expected. Distinct/Unique statistics after grouping do not demonstrate uniqueness of the original table. Root parent_id nulls are legitimate; exclude them before the parent anti-join. Coverage checks show absence of missing company assignments, not absence of incorrect or extra assignments. A company without classification at source may legitimately be missing from the bridge; investigate before treating this as a failure.

The originally proposed Company_Industry_Company_Orphans and Company_Industry_Industry_Orphans were deferred by agreement because both data tables derive from the same staging data, with no intentional independent filtering. These are defensive checks worth reconsidering if filters, independent sources or key transformations are introduced; sharing a source alone is not a proof of referential integrity. The planned inventory is 16, not 18 queries.

Additional checks: whole-data Column Quality; no null/error IDs, text IDs for industry keys and Whole Number for company keys; depth in {0,1}; roots have null parent, children have non-null valid parent; direct assignments have valid IDs. The user accepted the list ordering convention and chose not to add a cross-column hierarchy consistency check. This is an explicit assumption, not a confirmed semantic match to industry/subindustry. A future source with longer, empty or invalid lists may break the current transform or produce invalid assignments.

### Evidence from supplied yc-data(1).xlsx (older snapshot)

The saved M package was reopened and contained 12 shared queries. The later four industry QC queries are absent in this historical copy; see the newer snapshot below. Loaded worksheet snapshots show:
- Industry staging: 11,408 data rows, 6,408 distinct company IDs, 59 distinct industry IDs; depth 0 = 6,408 rows, depth 1 = 5,000 rows.
- Dictionary: 59 distinct industry IDs, zero missing IDs, 9 roots and 50 children.
- Bridge: 6,408 rows and 6,408 distinct company IDs, 59 distinct industry IDs, zero missing company/industry IDs.
- No Excel error cells in the three loaded industry tables.
- YC_Companies still returned a two-item list due to accidental navigation, and tag query did not yet contain Table.Distinct. The two existing tag/region anti-joins and duplicate-check definitions are present. Worksheet loads exist for four queries, not the intended connection-only setup.

These figures describe a saved snapshot, not a live Power Query refresh. The user's later reports state that the two issues were corrected, regions checked, groups organized and four new QC queries created, but the current uploaded file predates those changes. At this historical checkpoint those items had not been independently verified; see the newer snapshot below. Do not infer a successful refresh or current source completeness from this snapshot.

### Newer verified snapshot: yc-data(1)(1).xlsx

All 16 query definitions are present. The four industry QC queries use the expected source and group/filter or anti-join logic. Each saved result contains zero nonempty data rows and zero Excel error cells:

| Query | Saved exception rows | Result |
| --- | ---: | --- |
| Industry_PK_Duplicates | 0 | PASS |
| Industry_Parent_Orphans | 0 | PASS |
| Company_Industry_Duplicates | 0 | PASS |
| Companies_Without_Industry | 0 | PASS |

Saved data row counts: YC_Companies 6,408 (30 columns), YC_Industry_Staging 11,408, YC_Industry_Hierarchy 59, YC_Company_Industry 6,408. All four saved tables have zero Excel error cells. The restored companies query returns the complete source table and tag query now applies Table.Distinct. Region query remains undeduplicated by design if the region duplicate check passes. Existing five connection-only QC definitions were inspected, but their final output row counts are not stored as worksheets and have not been independently rerun. No live Power Query execution was performed. List order remains the agreed hierarchy assumption; no additional semantic QC query is planned for this iteration.

## 9. Next

Data Profiling is completed as a learning topic; the user reported completing the exercises. Data Structure and Data Types have also been covered. Text Cleaning is now completed as a learning topic; see section 11. Next topic: Text Standardization. The 16-query inventory, corrected company output, tag deduplication code, and four saved industry QC results have now been verified. Remaining operational checks: confirm full refresh and connection-only QC results; move Companies_Without_Industry to Quality Control if desired; decide final load settings. These are not prerequisites to repeat the completed structural lessons. Keep detection and remediation separate.


## 10. Profiling and transformation validation — learning checkpoint (2026-11-25)

Column Quality: counts/percentages of valid, error and empty values. Column Distribution: frequencies, Distinct (different values) and Unique (values appearing exactly once). Column Profile: selected-column statistics and value distribution; available statistics depend on type (e.g. min/max, average, standard deviation for numeric fields). Enable from View; switch from top 1,000 rows to entire data set for full-scope diagnosis. These are preview/diagnostic tools, not Applied Steps and not persistent QC queries. User reported completing all three exercises; no new numerical results were provided. Existing verified QC results remain historical as described above.

### Missing values: decision principle

Detect missingness technically; decide treatment based on structural, business and analytical requirements. A missing required primary key should be isolated and investigated before inclusion in a valid entity table; a missing optional website or team size need not cause deletion. A null parent_id is legitimate for a hierarchy root. A missing measure may justify exclusion from a specific analysis, with missingness disclosed, rather than deletion from the master dataset. Do not equate null, blank string, whitespace and error. Do not apply blanket null filtering. The complete Missing Values lesson remains ahead.

### Type conversion checks

Define target type by field semantics; automatic detection is a starting point, not a schema contract. Inspect identifiers/codes and leading zeros, date conventions, decimals, numeric precision, mixed types and join-key type compatibility. Check conversion errors with whole-data Column Quality; successful conversion does not prove preservation of meaning (silent errors include leading-zero loss, rounding and swapped day/month). Where necessary compare before/after values, ranges and row counts, test edge cases and check dependent queries. Correct an erroneous conversion at its original step or from Staging; changing the type of an already altered value may not restore lost information. Keep raw source values in Staging. See DATA_TRANSFORMATION.md for the taught conversion procedures.


## 11. Text Cleaning — diagnostic and validation protocol (2026-11-26)

Learning completed; actual YC text-quality counts and application have not been verified. Detection -> Assessment -> Cleaning -> Validation. Keep diagnostics separate from remediation, preserve staging, and evaluate the field's business meaning before changing its content. Trim is routine for approved fields; other cleaning is conditional. A technically detectable character is not necessarily a defect.

### 11.1 Pre-cleaning checks

Examples are M custom-column expressions for a nullable text column called name. Null is not a text-cleaning issue; handle it in the Missing Values topic. Source errors should remain visible or be counted separately, not silently treated as clean. Use full-data scope rather than relying on the top-1,000-row preview.

| Check | Example M expression |
| --- | --- |
| Edge whitespace | `[name] <> Text.Trim([name])` |
| Repeated ordinary spaces | `if [name] = null then false else Text.Contains([name], "  ")` |
| Control characters removed by Clean | `[name] <> Text.Clean([name])` |
| Special whitespace / invisible Unicode | List.ContainsAny with Unicode code lists (see below) |
| Field-specific prohibited characters | `if [name] = null then false else List.ContainsAny(Text.ToList([name]), {"*", "~"})` |
| Invalid characters in a digits-only text field | `[code] <> Text.Select([code], {"0".."9"})` |
| Unwanted substring | `if [name] = null then false else Text.Contains([name], "(Imported)")` |
| Unwanted prefix/suffix | Text.StartsWith / Text.EndsWith with a null guard |

Unicode example (special whitespace):

```powerquery
if [name] = null then false
else List.ContainsAny(
    List.Transform(Text.ToList([name]), Character.ToNumber),
    {160, 5760, 8192..8202, 8239, 8287, 12288}
)
```

For invisible formatting characters use an appropriate list, e.g. {173, 8203..8207, 8288, 65279}. These are non-exhaustive diagnostic sets and not a removal policy. Detection results may overlap (e.g. a special space at the edge or multiple ordinary spaces at the edge). Flag counts are not additive counts of unique faulty rows. The repeated-space test checks any position; restrict to trimmed input if edge runs should be excluded. Column Quality/Distribution/Profile do not directly flag these specific text anomalies.

### 11.2 Proposed reusable Text_Quality_Check

A future diagnostic Reference in Quality Control can scan declared text columns of a chosen query (initially YC_Companies) and return one row per column, with counts for edge whitespace, repeated spaces, special whitespace, control characters and selected invisible characters. Use Table.ColumnsOfType for eligible typed columns, apply appropriate field-specific rules, report null/error counts separately and retain access to exception rows for diagnosis. Evaluate performance for large data sets; avoid silently changing data. Pre-cleaning and post-cleaning checks must reference the corresponding input/output stage; a downstream Reference alone cannot describe original anomalies.

This is a design proposal only: no query was created, no M implementation tested in Excel and no issue counts measured. Existing 16-query inventory remains the latest verified baseline. Illustrative counts from lessons are not actual YC results.

### 11.3 Post-cleaning validation

| Check | Purpose |
| --- | --- |
| Row count | Text-only transformation should preserve row count and grain |
| Null and empty-string counts | Detect new missing or blank values; distinguish null from "" and whitespace |
| Error count | Detect introduced errors; investigate existing errors |
| Residual issues | Rerun the applicable diagnostic on the cleaned output |
| Changed values | Compare original and result for affected rows, using a stable identifier and matching input snapshot |
| Distinct count | Identify changes in cardinality; reduction can be intended for labels |
| Key uniqueness / collisions | Identify different original values mapping to the same cleaned key |
| Referential integrity | Recheck dependent joins if key values were changed |

For collision diagnosis, group original/cleaned pairs by cleaned value, count distinct original values, and inspect groups with more than one distinct original value. A-123, A#123 and A123 can collide after symbol removal. For categories this may be intended; for identifiers it can corrupt relationships. Distinct count alone does not establish that transformations are correct. Check affected examples and business rules. For composite keys validate the entire key combination. Do not automatically deduplicate to hide collisions.

Select tests proportionately: Trim -> residual edge whitespace, blanks, key uniqueness if applicable; Clean -> inspect lost word separators; Unicode/character removal and replacements -> changed values, empties, collisions and relevant joins. Always preserve original values in staging and document the rule. No text-cleaning PASS or FAIL is claimed without actual execution.

Next learning topic: Text Standardization.
