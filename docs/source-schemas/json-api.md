# JsonApi Source Guide

Use `JsonApi` when event data comes from a JSON endpoint instead of ICS, RSS, or static HTML.

If a usable event JSON endpoint exists, `JsonApi` is the correct type even when the page visually renders event cards.

For new JsonApi sources, default to `schemaVersion = 2`.

## When To Use It

Use `JsonApi` when:

- the page or widget fetches JSON responses containing events
- events live in arrays or nested data structures
- pagination, headers, tokens, or request workflows are required to reach the event payload

Do not use `JsonApi` when an equivalent ICS or RSS feed already provides the same data more simply.

## Minimum `schemaDefinition`

`JsonApi` requires `eventArrayPath` and `mappings`.

```json
{
  "eventArrayPath": "$.events[*]",
  "mappings": {
    "id": "$.id",
    "title": "$.title",
    "startTime": "$.start",
    "endTime": "$.end",
    "location": "$.location",
    "url": "$.url"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

For new sources, prefer this minimal v2 shape instead of omitting `schemaVersion`:

```json
{
  "schemaVersion": 2,
  "eventArrayPath": "$.events[*]",
  "mappings": {
    "title": "$.title",
    "startTime": "$.start"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

## Optional Features

Add these only when needed:

- `eventArrayPaths` for alternate payload shapes
- `pagination` for offset, cursor, or page-based APIs
- `authentication` for API key, bearer, or basic auth flows
- `requestHeaders` for required headers such as `User-Agent`
- `requestWorkflow` for multi-step request chains
- `schemaVersion = 2` for advanced features such as tokens, query templates, response transforms, or windowed pagination

## Default Version Guidance

- New JsonApi sources should start at `schemaVersion = 2`.
- Keep `schemaVersion = 1` only for legacy schemas that are already working and do not need v2-only features.
- If the source needs `responseTransforms`, `flattenPath`, token acquisition, query templates, or windowed pagination, `schemaVersion = 2` is required.

Use the contributor-facing [JsonApi V2 JSON Schema](json-api-v2-json-schema.md) when you want a machine-readable contract for authoring or AI validation.

## Requirements And Validation Notes

- `eventArrayPath` is required
- `mappings` is required and should be non-empty
- `schemaVersion` must be `1` or `2` when provided
- advanced features require `schemaVersion = 2`
- `requestWorkflow` and `pagination` cannot both be configured
- `pagination.maxRequestsPerRun` must be greater than `0` when provided
- dynamic token acquisition requires `tokenConfig.dynamicAcquisition.endpoint`

## Starter Patterns

The templates endpoint includes starter patterns for:

- flat event lists
- offset pagination
- bearer-authenticated APIs
- Bibliocommons-style keyed collections

Use the closest template first, then trim it down if the source is simpler.

## Worked Example: Landing Page To Advanced JsonApi Schema

Some event pages look like normal listing pages in the browser but are actually driven by JSON API calls. When that happens, do not fall back to HtmlLite just because visible cards exist.

Typical discovery flow:

1. Start from the public events page.
2. Inspect network calls or page scripts.
3. Identify the JSON endpoint that returns real event objects.
4. Decide whether auth headers, dynamic token acquisition, query templates, or windowed pagination are required.
5. Use `schemaVersion = 2` when those advanced features are needed.

Example advanced shape:

```json
{
  "schemaVersion": 2,
  "eventArrayPath": "$.docs.docs[*]",
  "mappings": {
    "id": "$.recid",
    "url": "$.eventUrl",
    "title": "$.title",
    "startTime": "$.startDate",
    "endTime": "$.endDate",
    "location": "$.location",
    "venueName": "$.listing.title",
    "venueAddress": "$.address1",
    "description": "$.description"
  },
  "queryTemplates": {
    "urlTemplate": "https://example.org/api/events?start={{windowStartUtc}}&limit={{pageSize}}&skip={{offset}}&token={{authToken}}"
  },
  "tokenConfig": {
    "acquisitionStrategy": "Dynamic",
    "dynamicAcquisition": {
      "endpoint": "https://example.org/api/token",
      "method": "Get",
      "extractPath": "$.token",
      "bindingKey": "authToken"
    }
  },
  "pagination": {
    "strategy": "Offset",
    "offsetParamName": "skip",
    "pageSizeParamName": "limit",
    "defaultPageSize": 12,
    "maxRequestsPerRun": 48
  },
  "validation": {
    "requiredFields": ["title", "startTime", "url"],
    "minEventsPerFetch": 1
  },
  "validationProfile": "Advanced"
}
```

Use this kind of pattern when the source has an event API with token acquisition, query templates, or transformed response fields. Do not try to recreate the same source with HtmlLite selectors.

## Split Date/Time And Source Timezone Pattern

Use this pattern when the payload stores date and time in separate fields and also exposes timezone in the source data.

```json
{
  "schemaVersion": 2,
  "eventArrayPath": "$.data.widgets[*].data.settings.events[*]",
  "mappings": {
    "id": "$.id",
    "title": "$.name",
    "startTime": "$.startTime",
    "endTime": "$.endTime",
    "timeZone": "$.timeZone",
    "url": "$.buttonLink.value"
  },
  "responseTransforms": {
    "flattenPath": "$.data.widgets[*].data.settings.events[*]",
    "fields": {
      "startTime": {
        "type": "Composite",
        "template": "{{startDate}} {{startClock}}",
        "inputs": {
          "startDate": "$.start.date",
          "startClock": "$.start.time"
        }
      },
      "endTime": {
        "type": "Composite",
        "template": "{{endDate}} {{endClock}}",
        "inputs": {
          "endDate": "$.end.date",
          "endClock": "$.end.time"
        }
      }
    }
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

Guidance:

- If the source provides timezone, map `timeZone` from the source instead of relying on runtime or server-local timezone assumptions.
- When response transforms reshape or reframe the effective event payload, set `responseTransforms.flattenPath` explicitly.
- If the source already provides explicit timezone offsets in `startTime` or `endTime`, preserve them.

## Recommended Contributor Workflow

1. Find the JSON endpoint that returns real event objects.
2. Start from the closest template or the minimal shape above.
3. Map `title` and `startTime` first, then add URL, ID, and location.
4. Add pagination or auth only when the source actually requires it.
5. Run `test-fetch`.
6. Inspect `sampleEvents` carefully for field completeness.
7. Submit the draft with `community-submissions`.

## Optional Geolocation Mapping Pattern

When the JSON source includes explicit coordinates, map them as source geolocation evidence and validate ranges before handoff.

Example source patterns you may encounter:

- flat fields: `latitude` and `longitude`
- abbreviated fields: `lat` and `lng`, `lat` and `lon`
- nested fields: `geo.lat` and `geo.lng`, `location.latitude` and `location.longitude`
- GeoJSON-like arrays: `geometry.coordinates`

Contributor guidance:

- Prefer direct source coordinates when valid.
- Keep `location` and `venueAddress` mappings even when coordinates exist, so fallback remains viable if some records omit coordinates.
- If coordinate fields are inconsistent across records, treat out-of-range or non-numeric values as invalid and rely on fallback geocoding for those records.
- Validate sample events for mixed data quality, not only one happy-path record.

## Endpoint Discovery (Generic)

When the visible events page is JS-rendered, use this repeatable approach:

1. Open developer tools and record requests triggered by initial page load.
2. Filter to JSON responses and inspect payload shape.
3. Prefer endpoints that already return event objects over endpoints that only return markup fragments.
4. Verify whether the endpoint needs required headers, query parameters, or token steps.
5. Confirm that the payload can be fetched consistently outside the browser session.

Do not assume the first JSON response is the correct event source. Always confirm that it contains repeated event items.

## Event Path Debug Loop

Use this loop when `validation.totalEventsParsed` is `0` or unexpectedly low:

1. Validate raw payload first and manually count a small sample of event items.
2. Start with the shortest plausible `eventArrayPath` to the repeated event array.
3. Test a minimal mapping set: only `title` and `startTime` required.
4. If `responseTransforms` are used, verify that transformed `startTime` and `endTime` values are actually populated and parseable.
5. If parsing fails, adjust only one variable per attempt (path scope, mapping path, transform shape, or headers).
6. Re-run `test-fetch` and inspect `sampleEvents` after each change.
7. Add optional fields (`description`, venue fields, `imageUrl`) only after core parsing is stable.

## Mapping Reliability Checks

Before final submission, verify:

1. `requiredFields` are actually mapped and consistently present.
2. Mappings are scoped per event item, not at root or unrelated siblings.
3. URL fields are valid or intentionally nullable when sources omit detail links.
4. Date and time fields are complete enough for downstream use; if split across fields, use `responseTransforms` to produce parser-ready datetime values.
5. If the source provides timezone, map `timeZone` and verify sample events reflect the intended source timezone.
6. `sampleEvents` include detail fields when the source provides them.

## Parser Compatibility Notes

Json path syntax support can vary by backend implementation. If a path looks valid but still parses zero events:

1. Prefer conservative path syntax and avoid advanced JSONPath features unless confirmed supported.
2. Replace complex path expressions with stepwise, explicit segments.
3. Validate wildcard behavior on arrays and keyed objects with targeted test-fetch runs.
4. Record any parser-specific limitations in your handoff notes.

## Common Failure Modes

- wrong `eventArrayPath`
- missing or incorrect `responseTransforms.flattenPath` when transforms reshape the effective event payload
- transformed `startTime` or `endTime` values that remain empty or unparsable after composition
- mappings that point to a scalar outside the current event item
- unnecessary advanced features when a simpler schema would work
- pagination configured together with requestWorkflow
- auth headers or tokens copied into the wrong part of the schema
- choosing HtmlLite even though a usable JSON endpoint exists

See also: [Submission API And Validation](submission-api-and-validation.md)