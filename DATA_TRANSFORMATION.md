# Data Transformation

Living notes for cleaning and type conversion in ETL Master. Distinguish learning, applied steps and verified saved-workbook results.

## 1. Data Types — learning completed, 2026-11-25

| Topic | Purpose | Power Query tool |
| --- | --- | --- |
| Change Type | Define the target type and convert compatible values | Column type menu / Transform |
| Using Locale | Interpret culture-dependent text correctly | Change Type > Using Locale |
| Type Conversion | Convert representations requiring custom logic | M functions / Table.TransformColumns |

### 1.1 Change Type and schema

Common types: Text, Whole Number (Int64.Type), Decimal Number, Fixed Decimal Number, Date, Date/Time, Time, Duration, Logical and Any. Choose by meaning, not appearance. Phone numbers and formatted identifiers often belong in Text; counts generally belong in Whole Number. For many fields, use Detect Data Type as a starting point where available, select multiple columns of the same type, use Table.TransformColumnTypes, or maintain a schema-mapping table for larger metadata-driven ETL processes. A complete schema mapping was not built.

Proposed YC_Companies types: id and team_size Whole Number; name, website, slug, status and batch Text. Confirm actual fields and source semantics. Select relevant analytical fields, define types, audit automatically inferred types and verify conversions. Full final schema and field selection were not independently verified.

Risks: lost leading zeros, numeric identifier precision, ambiguous dates and decimal/group separators, rounding, mixed-type errors, sampling limitations and incompatible join keys. Column Quality detects conversion errors, not every semantic error. Review the original conversion step before trying to recover lost information.

### 1.2 Using Locale

Right-click column > Change Type > Using Locale. Choose target type and source culture. For example, text 03/05/2026 means March 5 under en-US and May 3 under de-DE; 1,250 can be 1250 in en-US or 1.25 in de-DE. Locale is based on source conventions, not the analyst's location. JSON numbers that are already numeric do not require locale parsing; text values may. No YC-specific locale conversion was required or verified.

### 1.3 Custom conversion — launched_at

User showed a numeric value 1322045532 in launched_at. It is consistent with a Unix timestamp in seconds since 1970-01-01 UTC. The precise business definition of launched_at has not been independently verified. Do not equate it with founding date or funding date. Direct Change Type to Date does not implement Unix epoch conversion.

Agreed approach: transform the existing analytical column, not add a redundant one, because the raw value remains in YC_All_Companies. Example step using the previously taught RenamedColumns step:

```powerquery
= Table.TransformColumns(
    RenamedColumns,
    {
        {"launched_at",
        each if _ = null then null
        else #date(1970, 1, 1)
        + #duration(Number.RoundDown(_ / 86400), 0, 0, 0),
        type date}
    }
)
```

Use the actual preceding step name. This example assumes numeric seconds, preserves nulls and returns the UTC calendar date without time. An alternative is epoch datetime plus a duration of seconds, then Date.From; type datetime retains the time. Converting to Date intentionally discards time-of-day in the analytical table. Check numeric unit (seconds vs milliseconds), unexpected values, nulls, date ranges and conversion errors. User acknowledged the lesson; exact final M code, final saved output and a new refresh were not inspected.

## 2. Text Cleaning — learning completed, 2026-11-26

The initial five-topic outline was expanded and then consolidated to focus on practical analyst work. Organize by problem rather than by individual functions. Text extraction, splitting/combining for analytical purposes, formatting and case/category standardization are distinct topics; use those tools here only when they resolve a cleaning issue. Encoding and HTML parsing are source-specific exceptions, not routine steps.

| Topic | Detection (before changes) | Treatment | Policy |
| --- | --- | --- | --- |
| Leading/trailing whitespace | Compare original with Text.Trim | Text.Trim; optionally Text.TrimStart / Text.TrimEnd | Routine for approved text fields |
| Repeated internal spaces | Text.Contains(value, "  ") | Replace or split/filter/combine | Conditional on field semantics |
| Special whitespace / invisible Unicode | Text.ToList + Character.ToNumber + List.ContainsAny | Replace special spaces with ordinary spaces; remove only confirmed unwanted invisible characters | Diagnose first |
| Control characters | Compare original with Text.Clean | Text.Clean or replace tabs/newlines with spaces | Diagnose first; preserve meaningful separators |
| Character cleaning | List.ContainsAny for prohibited characters; compare with Text.Select for allowed characters | Text.Remove / Text.Select; flag invalid values if correction is uncertain | Field-specific |
| Text replacement | Text.Contains / StartsWith / EndsWith / PositionOf | Text.Replace, Replace Values; positional extraction/replacement if justified | Targeted |
| Cleaning validation | Before/after comparisons and checks | See DATA_QUALITY.md | Required, proportionate to risk |

Pattern-based cleaning / Column From Examples and positional variants are optional techniques, not mandatory stand-alone lessons. No general pattern-based exercise was completed.

### 2.1 Trim and dynamic text-column selection

UI: select approved text columns > Transform > Format > Trim. Unlike Excel TRIM, Power Query Text.Trim does not collapse internal spaces. It is idempotent with the same settings. Apply only where edge whitespace is meaningless. Numeric/date/logical columns are not targets; text representations may be trimmed before type conversion. Any-type columns must be assessed rather than assumed to be text.

Example M step after type assignment (replace the previous step name as needed):

```powerquery
= Table.TransformColumns(
    #"Changed Type",
    List.Transform(
        Table.ColumnsOfType(#"Changed Type", {type text}),
        each {_, Text.Trim, type text}
    )
)
```

This dynamically targets columns with matching declared text types; it does not discover text values inside arbitrary Any columns. Review nullable/type variations as appropriate. In production, restrict the selection to approved fields or subtract an explicit exclusion list. New text columns may be included at refresh if their schema is exposed and typed. Do not change the raw staging query or independently clean related keys without checking consistency.

### 2.2 Internal spaces and Unicode whitespace

Examples below are custom-column expressions for a nullable text field named name (use actual field name). To collapse repeated ordinary spaces:

```powerquery
if [name] = null then null else
    Text.Combine(
        List.Select(Text.Split([name], " "), each _ <> ""),
        " "
    )
```

This also removes ordinary edge spaces. One Replace Values pass replacing two spaces with one may leave repeated spaces. Do not apply to fixed-width text, code or meaningful formatting.

Replace NBSP (U+00A0) with an ordinary space rather than concatenate words:

```powerquery
Text.Replace([name], Character.FromNumber(160), " ")
```

Other diagnostic codes: 5760, 8192–8202, 8239, 8287, 12288. Invisible formatting examples: soft hyphen 173, zero-width space 8203, ZWNJ/ZWJ 8204/8205, LRM/RLM 8206/8207, word joiner 8288, BOM 65279. These are illustrative lists, not exhaustive Unicode classifications. The presence of a character does not establish that it is invalid. Preserve legitimate directionality, script shaping, emoji sequences and word boundaries. Example targeted removal, only when confirmed unwanted:

```powerquery
Text.Remove([name], Character.FromNumber(8203))
```

### 2.3 Control characters

UI: Transform > Format > Clean; M: Text.Clean([name]). Clean removes control characters rather than replacing them with a separator; e.g. Startup followed by a newline and Nation may become StartupNation. In a single-line label where a newline is a separator, replace first:

```powerquery
Text.Replace([name], "#(lf)", " ")
```

M escape sequences #(tab), #(cr), #(lf), #(cr,lf) represent relevant characters. For mixed line endings replace CRLF first, then remaining CR/LF as needed. Preserve meaningful newlines in descriptions and other multiline text. Recheck for residual whitespace after replacement.

### 2.4 Character cleaning and text replacement

- Text.Remove([name], {"*", "~"}) removes individual characters wherever found.
- Text.Select([code], {"A".."Z", "a".."z", "0".."9", "-"}) retains the allowed characters only (assuming code is text). Prefer flagging invalid identifiers over silently turning them into other identifiers. Allowed characters alone do not establish format validity.
- Text.Replace([name], "(Imported)", "") replaces every exact occurrence of a substring; it is case-sensitive. UI Replace Values distinguishes substring replacement from matching entire cell contents. Replace may leave edge spaces.
- Positional variants: Text.StartsWith / Text.EndsWith with Text.Range or Text.Start; Text.ReplaceRange uses a zero-based offset. Example guarded removal of an exact prefix:

```powerquery
if [name] = null then null
else if Text.StartsWith([name], "Company: ")
then Text.Range([name], Text.Length("Company: "))
else [name]
```

Check length, delimiter or expected character before position-based changes. Text.TrimStart/End remove individual specified characters, not a literal whole prefix/suffix. Text.Remove differs from Text.Replace: characters vs exact substring. Do not blindly remove legal suffixes, punctuation, minus signs, decimal separators or symbols in names (AT&T, C++, C#). Keep raw values and check collisions; A-123, A#123 and A123 may become one identifier.

### 2.5 YC application boundary

The learning chapter is complete. YC_Companies is the practice target; no confirmed text-cleaning M steps, issue counts, new QC queries, or refreshed/saved workbook results were supplied for this chapter. Do not assume these transformations have been applied. Recommended workflow: detect, assess extent and business meaning, clean approved analytical fields only, validate. Staging remains unchanged. See DATA_QUALITY.md for the diagnostic and validation protocol.

## 3. Later topics

Text Standardization; Missing Values; Duplicates; Errors; Numerical Validity; Date Validity; Categorical Validity; Logical Consistency; Referential Integrity; Data Validation. Some checks were already implemented during structural modeling; avoid repeating them unnecessarily. See DATA_QUALITY.md.

## 4. Verification boundary

The verified baseline remains yc-data(1)(1).xlsx (16 saved queries). Subsequent header and date work is a learning/user-reported checkpoint, not a newly inspected workbook. Text Cleaning is a completed learning checkpoint; application and text-quality results are not verified. No workbook or query code was remotely modified by this documentation update. Full refresh, remaining connection-only QC outputs, final schema and load settings remain open.
