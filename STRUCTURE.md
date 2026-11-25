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

Expected model: Companies (PK id) 1 to many Company_Tags (FK company ID, composite key company ID + tag). Regions and industries were subsequently implemented (see 3.6). former_names and tags_highlighted remain candidates, not yet implemented. The later workbook uses YC_All_Companies as parent. No final Data Model relationships have been created or verified.

## 3.5 — Key and structural validation

See [DATA_QUALITY.md](DATA_QUALITY.md) for the validation and remediation protocol, observed duplicate pairs, parent key profile and anti-join result.

## 3.6 — Query Architecture

### 3.6.1–3.6.3 — Groups and separation of concerns

Use the Queries pane, right-click > Move to Group > New Group (or select an existing group). Grouping changes organization, not M logic or query dependencies. The agreed structure is Staging, Data Tables and Quality Control. A Reference of YC_All_Companies named YC_Companies separates the analytical company table from staging. Keep staging fields intact so other child queries do not lose inputs. Remove nested fields from the analytical table only after deciding where their information belongs; final column selection is still pending.

Regions were also implemented via Reference, retaining Column1.id and Column1.regions and expanding the list into rows, following the tags procedure. The intended grains of tags and regions are distinct company/tag and company/region pairs. Names of actual workbook queries are used below.

### 3.6.4–3.6.5 — Industry modeling

The source includes industry (root text), subindustry (path text, e.g. Industrials -> Manufacturing and Robotics), and industries (List). The agreed v0.1 design uses only the list for construction. The order is accepted as a working assumption: index 0 is root, index 1 is child; one item denotes a root-only assignment. We intentionally did not add a separate semantic cross-column QC query. The two other fields remain available in staging. This is a modeling convention, not an independently verified claim about every future source record.

We distinguish a list of multiple independent assignments from a single hierarchical path. The model permits many-to-many company/industry relationships and a parent-child tree, but the current extraction produces at most one direct assignment per company. It does not infer multiple paths, more than two levels, or missing/invalid lists. These require explicit future decisions.

### Industry staging: reproducible procedure

1. Reference YC_All_Companies; retain Column1.id and Column1.industries; rename ID to company_id.
2. Add custom column industry_depth: List.Positions([Column1.industries]). A global Index Column is not a substitute: positions must restart within each list.
3. Add industry_with_depth: List.Zip({[Column1.industries], [industry_depth]}). This pairs each item with its position. Expanding both independent lists would risk a Cartesian product.
4. Expand industry_with_depth to new rows once. Do not expand each inner pair to rows.
5. Add industry_name = [industry_with_depth]{0}; depth = [industry_with_depth]{1}.
6. Add parent_name: if [depth] = 1 then [Column1.industries]{0} else null.
7. Add industry_id: root = industry_name; child = parent_name & " > " & industry_name; other depths = null.
8. Add parent_id: if [depth] = 1 then [parent_name] else null.
9. Add is_direct_assignment: [depth] = List.Count([Column1.industries]) - 1. Calculate while the original list is available.
10. Remove helper columns Column1.industries, industry_depth and industry_with_depth; assign company_id and depth Whole Number, names and IDs Text, is_direct_assignment Logical.
11. Name the query YC_Industry_Staging and place it in Staging.

Path IDs avoid unstable row indexes during refresh, but assume the separator is absent from component names and change if names/paths change. Direct assignment is the last item of the single path, not every ancestor. For unsupported depth the current logic could produce a null industry_id on the direct assignment; do not interpret this as valid. Inspect these exceptions before extending the model.

### Industry dictionary

Reference YC_Industry_Staging as YC_Industry_Hierarchy. Retain industry_id, industry_name, parent_id, depth; Remove Duplicates across all four fields. Verify uniqueness of industry_id separately (do not silently choose between conflicting rows for the same key).

Grain: one row per hierarchy node. PK: industry_id. parent_id references industry_id in the same logical table; null is expected for root nodes. Depth 0 = root; 1 = child.

### Company–industry bridge

Reference YC_Industry_Staging as YC_Company_Industry. Filter is_direct_assignment = true. Retain company_id and industry_id, Remove Duplicates on both. Grain: one distinct direct company/industry assignment; composite key company_id + industry_id.

Logical model:
- YC_Companies (Column1.id) 1:N YC_Company_Industry (company_id).
- YC_Industry_Hierarchy (industry_id) 1:N YC_Company_Industry (industry_id).
- YC_Industry_Hierarchy (industry_id) 1:N YC_Industry_Hierarchy (parent_id) for non-root nodes.
- The bridge supports M:N in principle, although the current single-path input convention yields one direct assignment per classified company.

Example (illustrative): company 5 -> Industrials > Manufacturing and Robotics; dictionary contains Industrials (root, null parent) and Industrials > Manufacturing and Robotics (child, parent Industrials). For future PivotTables, parent_id alone does not automatically produce a usable multi-level hierarchy in Excel; derived level columns or an appropriate model may be needed. Relationships have not yet been created in Excel Data Model.

### Query inventory: agreed target (16)

| Group | Query | Depends on / purpose |
| --- | --- | --- |
| Staging | YC_All_Companies | Web JSON, List to Table, Expand Records |
| Staging | YC_Industry_Staging | YC_All_Companies; indexed paths |
| Data Tables | YC_Companies | YC_All_Companies; analytical company rows |
| Data Tables | YC_Company_Tag | YC_All_Companies; company/tag |
| Data Tables | YC_Company_Region | YC_All_Companies; company/region |
| Data Tables | YC_Industry_Hierarchy | YC_Industry_Staging; node dictionary |
| Data Tables | YC_Company_Industry | YC_Industry_Staging; direct bridge |
| Quality Control | Companies_PK_Duplicates | YC_All_Companies |
| Quality Control | Cmpany_Tag_Duplicheck | YC_Company_Tag |
| Quality Control | Region_Duplicheck | YC_Company_Region |
| Quality Control | Company_Tags_Orphans | YC_Company_Tag vs YC_All_Companies |
| Quality Control | Company_Region_Orphans | YC_Company_Region vs YC_All_Companies |
| Quality Control | Industry_PK_Duplicates | YC_Industry_Hierarchy |
| Quality Control | Industry_Parent_Orphans | Non-null parent_id vs hierarchy IDs |
| Quality Control | Company_Industry_Duplicates | YC_Company_Industry |
| Quality Control | Companies_Without_Industry | YC_Companies vs YC_Company_Industry |

The two originally proposed bridge orphan QC queries were deferred, not implemented as required queries; the earlier 18-query target was revised to 16. See DATA_QUALITY.md for rationale and limitations.

## 3.7 — Workbook persistence and recovery

Close & Load / Close & Load To commits editor changes to the workbook; Only Create Connection preserves the query without a worksheet table. Then explicitly save Excel (Ctrl+S). Applied Steps alone and Git documentation are not backups of an unsaved workbook. Reopen the saved file and verify query code/counts after significant changes. Load settings can be changed via Queries & Connections > Load To. Preserve a versioned copy before automated edits. Reference defines logical dependency but not guaranteed one-time retrieval or caching.

A computer-use session rewrote the industry extraction beyond its requested scope; we restored the List.Positions/List.Zip sequence. Later the saved workbook contained an accidental Drill Down in YC_Companies (Source{0}[Column1.industries]), missing tag deduplication and unintended worksheet loads. The user subsequently reported deleting the Navigation step, applying tag deduplication, checking regions and reorganizing groups. These fixes are not present in the supplied older 12-query workbook. Do not claim the final workbook was verified. No workbook was edited by this documentation update.

For automated assistance restrict scope: do not modify existing queries, names, types, dependencies, load settings or M code unless specifically authorized. QC tasks detect and report; they do not remediate. Stop on an execution blocker rather than inventing workarounds. Report actual query names, sources, method, row count and PASS/FAIL/NOT TESTED. 

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
- [x] Model regions and industries; document the industry hierarchy and bridge.
- [ ] Verify saved final query inventory and remaining QC results.
- [ ] Decide whether other nested lists need analytical tables.

Parent ID was explicitly assigned Whole Number. Tag pair deduplication was reported completed; saved final verification remains open. No final load or full refresh verification has been established yet.
