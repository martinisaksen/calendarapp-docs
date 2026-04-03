# Autonomous Source Onboarding Runbook

Use this runbook when an AI agent can execute the full source onboarding loop from a starting URL to a validated Draft SourceSchema.

## Scope

- In scope: source onboarding, `test-fetch`, and contributor Draft submission via `POST /api/source-schemas/community-submissions`
- Out of scope: admin-only approval operations

## Non-Interactive Execution Rules

- Do not pause for manual confirmation during execution.
- If one extraction strategy fails, automatically try the next strategy.
- Stop only on terminal failure conditions.
- Always emit a final machine-readable report.
- Do not propose or implement source-specific runtime code paths; prefer schema changes or generic reusable platform improvements.

## Required Reading

Before execution, read:
- [llms.txt](https://martinisaksen.github.io/calendarapp-docs/llms.txt) to discover canonical docs and schema files
- [Choose A Source Type](https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/choose-source-type.md)
- [Submission API And Validation](https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/submission-api-and-validation.md)
- [Submission Request JSON Schema](https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/submission-request.schema.json)
- the type-specific page that matches the chosen source type (use raw GitHub URL from llms.txt)

Type-specific schemaDefinition JSON Schemas:
- [ICS schemaDefinition JSON Schema](https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/ics.schema.json)
- [RSS schemaDefinition JSON Schema](https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/rss.schema.json)
- [JsonApi V2 schemaDefinition JSON Schema](https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/json-api-v2.schema.json)
- [HtmlLite schemaDefinition JSON Schema](https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/html-lite.schema.json)

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
- `sourceType`
- `schemaId`
- `status` (`Success` or `Failed`)
- `paginationModeUsed`
- `detailEnrichmentUsed`
- `validationTotalEventsParsed`
- `validationIsSuccess`
- `handoffReady`
- `blockers` (array)
- `nextActions` (array)

## Step 1: Identify The Best Source Type

Prefer the most structured option available:
1. `Ics`
2. `Rss`
3. `JsonApi`
4. `HtmlLite`

Decision loop:
1. Check whether `feedUrl` is already an ICS or RSS feed.
2. If not, inspect the page or network calls for a JSON endpoint that returns event objects.
3. Use HtmlLite only when the events are present in static HTML and no better structured feed exists.

Evidence requirements before locking source type:

1. Endpoint or page response can be fetched reliably at run time.
2. Repeated event records are visible in payload (not only configuration wrappers).
3. At least one sample record contains `title` and time/date evidence.
4. Selected type is still highest-priority among usable options.

Terminal failure:
- no usable ICS, RSS, JsonApi, or HtmlLite source can be found

## Step 2: Build The Smallest Valid Schema For The Chosen Type

Structured-source bootstrap rules:
- `Ics`: use a minimal validation-only schemaDefinition unless the source needs stricter required fields
- `Rss`: define `extractionRules` and map at least `title` and parseable `startTime`
- `JsonApi`: start new sources at `schemaVersion = 2`, define `eventArrayPath` plus `mappings`, then add pagination or auth only if needed

JsonApi bootstrap guardrails:

1. Start with `schemaVersion = 2` for new sources.
2. Start with minimal mappings (`title`, `startTime`) and requiredFields aligned to those mappings.
3. Validate event path first, then expand field coverage.
4. If the source splits date and time across fields, use `responseTransforms` to compose parser-ready `startTime` and `endTime` values.
5. If the source provides timezone, map `timeZone` from source data.
6. If zero events parse, adjust one variable at a time (path, mapping scope, transform shape, headers, pagination).
7. Capture parser-compatibility notes when a path syntax assumption fails.

Geolocation decision loop:

1. Detect coordinate candidates in source payload fields using common key variants: `latitude`, `lat`, `longitude`, `lng`, `lon`, `coordinates`, `geo`, `geometry`.
2. Validate coordinate values: latitude numeric and between `-90` and `90`, longitude numeric and between `-180` and `180`.
3. If valid source coordinates exist, preserve them and treat geocoding as not required for those records.
4. If coordinates are missing or invalid, require high-quality `location` and `venueAddress` mappings so geocoding fallback has useful input.
5. Confirm test-fetch sample events contain either valid source coordinates or strong text location evidence for fallback geocoding.
6. Never generate synthetic coordinates from guesses.

HtmlLite bootstrap rules remain below.

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

## Step 3: Apply Type-Specific Contract Rules

For every type:
- `schemaDefinition` must serialize as a JSON string in the request body
- prefer the contributor-safe workflow: `test-fetch` first, then `community-submissions`
- never call admin approval endpoints

Request envelope preflight checks (mandatory before POST):
1. Validate request JSON against Submission Request JSON Schema.
2. Parse `schemaDefinition` string into JSON and validate against the selected type's schemaDefinition JSON Schema.
3. Reject payloads that use cross-contract field names:
  - `sourceName` -> use `name`
  - `sourceUrl` or `url` -> use `feedUrl`
  - `sourceType` -> use `type`
  - top-level `contactEmail` -> use `metadata.contactEmail`
4. For `Ics`, keep `schemaDefinition` as `{}` or validation-only; do not generate `eventMapping`, `mappings`, `transforms`, or parser-specific extraction blocks.

For `JsonApi`:
- new sources should default to `schemaVersion = 2`
- `schemaVersion` must be `1` or `2` when provided
- advanced features require `schemaVersion = 2`
- `requestWorkflow` and `pagination` cannot both be configured

For `Rss`:
- `extractionRules` values should resolve against each feed item

For `Ics`:
- rely on the feed for title, start, end, location, and UID when present
- send top-level `feedUrl` (not `url`) in test-fetch/submission payloads
- keep `schemaDefinition` minimal (`{}` or validation-only) unless stricter checks are needed
- do not add `eventMapping` for standard ICS feeds
- `eventUrl` maps from the VEVENT `URL` field; relative URLs are resolved against `feedUrl`, and if the resolved URL points back to the source feed, the parser ignores it and extracts the first http/https URL from `DESCRIPTION` instead — no schema configuration needed
- if a source keeps full event details only on HTML event pages, optional detail enrichment can be enabled with `schemaDefinition.eventEnrichment` (`enabled`, `parser`, `maxFetchesPerRun`, `sameHostOnly`)
- if `eventEnrichment.parser = HtmlLite`, require `eventEnrichment.htmlLiteMappings`
- for test-fetch/runtime parity, use the same `eventEnrichment` block in the payload used for `POST /api/source-schemas/test-fetch`

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

Only `JsonApi` and `HtmlLite` commonly need pagination.

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

## Step 5: Test, Submit, Evaluate, Iterate

1. Run `POST /api/source-schemas/test-fetch`.
2. If test-fetch succeeds, submit Draft via `POST /api/source-schemas/community-submissions`.
3. Read `response.validation`.
4. If failed, adjust and resubmit.

For every iteration, record:

1. attempted change
2. `validation.isSuccess`
3. `validation.totalEventsParsed`
4. any `validation.errorMessage`

Do not batch multiple schema changes in a single retry unless blocked by request-shape validity.

When `validation.totalEventsParsed = 0`, retry in this order:
1. verify event path or `responseTransforms.flattenPath` matches repeated event nodes
2. verify `title` and parseable `startTime`
3. verify transformed `startTime` and `endTime` are actually populated after mappings/transforms
4. map `timeZone` when the source provides it
5. add or correct pagination

For JsonApi specifically, remember that a zero-event result can mean either:
- event path selection matched no event nodes
- required fields were empty after mappings/transforms and every candidate event was filtered out

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
- geolocation note: `source-coordinates-used` or `geocode-fallback-expected`

## Final Report Examples

Success:

```json
{
  "sourceName": "Example Events",
  "feedUrl": "https://www.example.org/events",
  "sourceType": "HtmlLite",
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
  "sourceType": "HtmlLite",
  "schemaId": "7d88c4a3-c9d4-4c4d-beb1-b9d71e590fa3",
  "status": "Failed",
  "paginationModeUsed": "queryIncrement",
  "detailEnrichmentUsed": false,
  "validationTotalEventsParsed": 0,
  "validationIsSuccess": false,
  "handoffReady": false,
  "blockers": [
    "validation.errorMessage: Selector returned zero event cards"
  ],
  "nextActions": [
    "Adjust schemaDefinition and resubmit",
    "If failure persists after max attempts, escalate with payload and error details"
  ]
}
```
