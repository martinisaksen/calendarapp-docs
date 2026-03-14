# Autonomous Prompt Template

Use this prompt with autonomous agents:

---

Add one new HTML event source to HtmlLite SourceSchema end-to-end without human interaction.

Inputs:
- sourceName: <SOURCE_NAME>
- feedUrl: <FEED_URL>
- expectedLocation: <LOCATION>
- expectedCategory: <CATEGORY>
- expectedRegion: <REGION>
- expectedLanguage: <LANGUAGE>

Before starting, read the following reference pages in order:

1. [API Workflow](../../source-schemas/html-lite/api-workflow.md) — POST endpoint, request body fields, response shape, and how to interpret `validation.*`
2. [Schema Basics](../../source-schemas/html-lite/schema-basics.md) — `schemaDefinition` structure: `eventCardSelector`, `mappings`, `validation`
3. [Selectors](../../source-schemas/html-lite/selectors.md) — CSS selector and XPath syntax for extracting field values
4. [Pagination](../../source-schemas/html-lite/pagination.md) — `nextLink`, `queryIncrement`, `pathIncrement`, and `fixedUrls` modes and their schema fields
5. [Detail Pages](../../source-schemas/html-lite/detail-pages.md) — when and how to enable `detailPage` enrichment
6. [Troubleshooting](../../source-schemas/html-lite/troubleshooting.md) — how to diagnose and fix validation failures before retrying

Use this link if you need additional information to navigate to other docs: [Home](../../index.md)

Constraints:
- use only SourceSchemas endpoints
- schemaDefinition must be submitted as a JSON string
- prefer machine-readable datetime fields
- use stable external IDs; never use title text alone for ID
- automatically choose pagination mode: nextLink, queryIncrement, pathIncrement, or fixedUrls
- enable detail page enrichment only when needed
- retry with selector and pagination adjustments when submit validation fails
- do not stop for user confirmation
- do not call admin-only approve or trigger endpoints

Required workflow:
1. viability detection
2. schema construction
3. pagination strategy selection
4. submit draft and evaluate response.validation
5. retry on validation failure
6. produce admin handoff packet
7. final structured report

Success criteria:
- response.validation.isSuccess = true
- draft submission isSuccess = true
- response.validation.totalEventsParsed is non-zero

Return only:
- final structured report
- final schemaDefinition JSON used in submission

Example success report:

```json
{
	"sourceName": "Example Events",
	"feedUrl": "https://www.example.org/events",
	"schemaId": "1acba9be-d70c-435a-8efb-123e350638e5",
	"status": "Success",
	"validationTotalEventsParsed": 12,
	"validationIsSuccess": true,
	"handoffReady": true,
	"blockers": [],
	"nextActions": ["Send to admin review"]
}
```

Example failure report:

```json
{
	"sourceName": "Example Events",
	"feedUrl": "https://www.example.org/events",
	"schemaId": "7d88c4a3-c9d4-4c4d-beb1-b9d71e590fa3",
	"status": "Failed",
	"validationTotalEventsParsed": 0,
	"validationIsSuccess": false,
	"handoffReady": false,
	"blockers": ["validation.errorMessage: Invalid schema definition"],
	"nextActions": ["Fix schema and resubmit"]
}
```
