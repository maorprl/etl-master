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

## 2. Text Cleaning — next learning topic

Planned sequence: Trim; Clean; Replace Values; Whitespace; Special Characters. Apply only where justified by the actual data and field semantics. No text-cleaning steps have been reported as completed.

## 3. Later topics

Text Standardization; Missing Values; Duplicates; Errors; Numerical Validity; Date Validity; Categorical Validity; Logical Consistency; Referential Integrity; Data Validation. Some checks were already implemented during structural modeling; avoid repeating them unnecessarily. See DATA_QUALITY.md.

## 4. Verification boundary

The verified baseline remains yc-data(1)(1).xlsx (16 saved queries). Subsequent header and date work is a learning/user-reported checkpoint, not a newly inspected workbook. No workbook or query code was remotely modified by this documentation update. Full refresh, remaining connection-only QC outputs, final schema and load settings remain open.
