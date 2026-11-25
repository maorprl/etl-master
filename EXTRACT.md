# Extract

Living documentation of data extraction methods and their practical application in ETL Master.

## Extraction principles

- Choose the connector according to the source access method and data format.
- Separate extraction from subsequent structuring and transformation.
- Record the procedure, generated code, observed output and validation status.

## Connector knowledge

| Source | Format | Power Query connector |
| --- | --- | --- |
| Web endpoint | JSON | From Web |
| Local file | JSON | From JSON |

## EXT-001 — Y Combinator Companies

| Property | Value |
| --- | --- |
| Source ID | SRC-001 (see DATA_SOURCES.md) |
| Endpoint | https://yc-oss.github.io/api/companies/all.json |
| Format | JSON |
| Tool | Excel Power Query |
| Connector | From Web |
| Authentication | Anonymous if prompted (actual selection not independently observed) |
| Date | 2026-11-25 |
| Status | Initial extraction successful, evidenced by user screenshot |

### Procedure

1. Open Excel.
2. Select Data > Get Data > From Other Sources > From Web.
3. Select Basic and enter the endpoint URL.
4. Click OK.
5. If prompted, select Anonymous and click Connect.
6. Inspect the resulting Power Query preview.

### Generated M code

```powerquery
= Json.Document(Web.Contents("https://yc-oss.github.io/api/companies/all.json"))
```

- `Web.Contents`: retrieves the web response.
- `Json.Document`: parses the response as JSON.

### Observed result

The Power Query screenshot shows a top-level List containing Record values. Each record is expected to represent a company, based on the source purpose; its fields have not yet been inspected.

### Extraction validation

- [x] Query returned a parsed JSON preview with no visible error.
- [x] Top-level structure observed: List of Records.
- [ ] Verify full record count and completeness.
- [ ] Verify refresh behavior.

No table conversion, record expansion, cleaning or loading has been verified yet.

### Next step

Inspect the JSON structure and begin Data Structuring. Avoid expanding nested lists before establishing the intended grain.
