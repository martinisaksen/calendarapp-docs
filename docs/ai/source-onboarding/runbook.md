# Autonomous Source Onboarding Runbook

Use this runbook when an AI agent can execute the full source onboarding loop from a starting URL to a validated Draft SourceSchema.

## Scope

- In scope: source onboarding and Draft submission via `POST /api/source-schemas`
- Out of scope: admin-only approval operations

## Non-Interactive Execution Rules

- Do not pause for manual confirmation during execution.
- If one extraction strategy fails, automatically try the next strategy.
- Stop only on terminal failure conditions.
- Always emit a final machine-readable report.
- Do not propose or implement source-specific runtime code paths; prefer schema changes or generic reusable platform improvements.

## Required Reading

Before execution, read the HtmlLite reference pages under [HtmlLite Source Submission](../../source-schemas/html-lite/overview.md).

## Input Contract

Required inputs:
- `sourceName`
- `feedUrl`

Optional inputs:
- `expectedLocation`
- `expectedCategory`
- `expectedRegion`
- `expectedLanguage`
- `knownEventCardSelector`
- `knownTimeSelector`
- `knownNextLinkSelector`

## Output Contract

The final output must be a single JSON object with:
- `sourceName`
- `feedUrl`
- `schemaId`
- `status` (`Success` or `Failed`)
- `paginationModeUsed`
- `detailEnrichmentUsed`
- `validationTotalEventsParsed`
- `validationIsSuccess`
- `handoffReady`
- `blockers` (array)
- `nextActions` (array)

## Step 1: Viability Detection

1. Fetch `feedUrl` HTML.
2. Confirm event cards are present in static HTML.
3. If static cards are absent, detect JS-rendered indicators.
4. If JS-rendered, try discovering JSON API or ICS alternatives.

Terminal failure:
- no static event HTML and no discoverable API/ICS alternative

## Step 2: Build Bootstrap Schema

Before finalizing mappings, define a field-completeness target against 3 to 5 known detail URLs:
- `title`
- `startTime`
- `endTime`
- `description`
- `location` or `venueName`
- `venueAddress`
- `imageUrl`
- `url`

Parsing success without these fields is not considered complete onboarding.

Build the smallest valid schema first:
- `eventCardSelector`
- `mappings.title`
- parseable `mappings.startTime`
- `validation.requiredFields = ["title", "startTime"]`

Bootstrap strategy:
1. Validate minimal schema quickly.
2. If zero events, simplify `eventCardSelector` to a stable repeated title wrapper.
3. If list-page time is unreliable, use temporary `literal:` `startTime`.
4. Move real date/time extraction to detail page (`startDate` + `startTime`).
5. Add pagination only after non-zero parse is achieved.

## Step 3: Apply Contract Rules

Mapping DSL rules:
- mapping values must be strings, never objects
- use CSS-like selectors for simple tag/class/id/descendant patterns
- use `xpath=` for unsupported CSS patterns (`>`, `.a, .b`, sibling logic, positional filters)
- attribute extraction uses `selector@attribute` or `selector::attr(attribute)`

Detail-page rules:
- set `detailPage.enabled = true` only when list fields are incomplete
- `detailPage.linkSelector` must resolve to a detail URL from list-card scope
- `detailMappings` run against detail-page DOM and override non-empty list values

Identity rules:
1. explicit external ID mapping when available
2. fallback to stable URL/permalink
3. avoid title-derived IDs

## Step 4: Choose Pagination Strategy

Decision order:
1. `nextLink`
2. `queryIncrement`
3. `pathIncrement`
4. `fixedUrls`
5. single-page fallback with explicit pagination risk note

Pagination contract:
- use `pagination.type` (not `pagination.mode`)
- allowed values: `nextLink`, `queryIncrement`, `pathIncrement`, `fixedUrls`
- `query` is invalid; use `queryIncrement` with `parameter`, `start`, `step`

## Step 5: Submit, Evaluate, Iterate

1. Submit Draft via `POST /api/source-schemas`.
2. Read `response.validation`.
3. If failed, adjust and resubmit.

When `validation.totalEventsParsed = 0`, retry in this order:
1. simplify `eventCardSelector`
2. verify `title` and parseable `startTime`
3. apply temporary `literal:` `startTime`
4. verify detail link and detail mappings
5. add or correct pagination

Max refinement attempts:
- 8 submissions

Terminal failure:
- still no successful validation after max attempts

## Step 6: Handoff Packet

If validation succeeds, produce handoff payload containing:
- `schemaId`
- `feedUrl`
- final `schemaDefinition`
- `validation.totalEventsParsed`
- `validation.sampleEvents` summary
- `validation.isSuccess`

## Final Report Examples

Success:

```json
{
  "sourceName": "Example Events",
  "feedUrl": "https://www.example.org/events",
  "schemaId": "1acba9be-d70c-435a-8efb-123e350638e5",
  "status": "Success",
  "paginationModeUsed": "nextLink",
  "detailEnrichmentUsed": false,
  "validationTotalEventsParsed": 12,
  "validationIsSuccess": true,
  "handoffReady": true,
  "blockers": [],
  "nextActions": [
    "Send schemaId and validation summary to admin review queue"
  ]
}
```

Failure:

```json
{
  "sourceName": "Example Events",
  "feedUrl": "https://www.example.org/events",
  "schemaId": "7d88c4a3-c9d4-4c4d-beb1-b9d71e590fa3",
  "status": "Failed",
  "paginationModeUsed": "queryIncrement",
  "detailEnrichmentUsed": false,
  "validationTotalEventsParsed": 0,
  "validationIsSuccess": false,
  "handoffReady": false,
  "blockers": [
    "validation.errorMessage: No parser registered for schema type HtmlLite"
  ],
  "nextActions": [
    "Adjust schemaDefinition and resubmit",
    "If failure persists after max attempts, escalate with payload and error details"
  ]
}
```
