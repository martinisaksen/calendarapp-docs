# Submission API And Validation

This page describes the shared contributor workflow for every source type.

Use these endpoints:

- `GET /api/source-schemas/templates`
- `POST /api/source-schemas/test-fetch`
- `POST /api/source-schemas/community-submissions`

Do not use admin-only approval or trigger endpoints as a community contributor.

## Shared Request Envelope

Every submission uses the same top-level fields:

```json
{
  "name": "Example Source",
  "description": "What this source publishes",
  "type": "JsonApi",
  "feedUrl": "https://example.org/events",
  "schemaDefinition": "{\"eventArrayPath\":\"$.events[*]\",\"mappings\":{\"title\":\"$.title\",\"startTime\":\"$.start\"}}",
  "metadata": {
    "location": "Example City",
    "category": "community",
    "region": "WA",
    "language": "en",
    "estimatedEventCount": 20
  },
  "calendarStrategyOverride": "Auto"
}
```

Important:

- `type` must be one of `Ics`, `Rss`, `JsonApi`, `HtmlLite`
- `feedUrl` is required and must be a valid absolute URL
- `schemaDefinition` must be sent as a JSON string, not a nested object
- `schemaDefinition` can be up to 200000 characters

`schemaDefinition` must contain only fields supported by the selected source type. Do not include wrapper or note fields such as:

- `sourceName`
- `baseUrl`
- `transforms`

Those do not belong in the submitted schema body.

## Contributor-Safe Workflow

### 1. Load A Starter Template

Use `GET /api/source-schemas/templates` when you want a known-good starting point.

The templates endpoint currently publishes starter schemas for:
- `Ics`
- `Rss`
- several `JsonApi` patterns

If the templates endpoint does not include the exact type or shape you need, start from the type guide on this site.

### 2. Test Fetch Before Submitting

Use `POST /api/source-schemas/test-fetch` while refining the schema.

Example:

```json
{
  "type": "Rss",
  "feedUrl": "https://example.org/events/feed.rss",
  "schemaDefinition": "{\"extractionRules\":{\"title\":\"title\",\"startTime\":\"pubDate\"},\"validation\":{\"requiredFields\":[\"title\",\"startTime\"]}}"
}
```

### 3. Submit The Draft

Use `POST /api/source-schemas/community-submissions` once test-fetch is stable.

The response returns:
- `sourceSchema`
- `validation.isSuccess`
- `validation.totalEventsParsed`
- `validation.sampleEvents`
- `validation.errorMessage`
- `validation.validationErrors`

## Shared Validation Rules

The backend enforces these common rules for every source type:

- `schemaDefinition` must be valid JSON
- `schemaDefinition` cannot exceed the max depth limit of 30
- `mappings` cannot exceed 300 entries when present
- `requestWorkflow.steps` cannot exceed 25 entries when present
- `requestHeaders` cannot exceed 50 entries when present
- outbound URLs must be `http` or `https`
- localhost, private-network, and restricted hosts are blocked

## Geolocation Mapping And Fallback Expectations

Use this guidance when authoring schemas for any source type.

- If the source provides valid coordinates, map and preserve that source geolocation evidence.
- If source coordinates are missing or invalid, ingestion falls back to geocoding from text fields such as `location` and `venueAddress`.
- Address query normalization is applied before geocoding, so cleaner `location` and `venueAddress` mappings improve results.
- On updates, if an incoming event has no geometry, existing persisted geometry is retained rather than cleared.

Coordinate validation guidance for contributors and AI:

- Latitude must parse as a number in range `-90` to `90`.
- Longitude must parse as a number in range `-180` to `180`.
- Coordinates with both values missing, non-numeric, or out of range should be treated as invalid and should trigger fallback geocoding.
- If source payload uses GeoJSON-style coordinate arrays, remember array order is usually longitude, latitude.

Common source key patterns to look for during mapping:

- latitude-like keys: `lat`, `latitude`, `geo.lat`, `location.latitude`, `geometry.location.lat`
- longitude-like keys: `lng`, `lon`, `long`, `longitude`, `geo.lng`, `location.longitude`, `geometry.location.lng`
- coordinate objects or arrays: `coordinates`, `geo`, `geometry`, `point`

Authoring note:

- Do not invent coordinates.
- If explicit coordinate mapping is not supported for a source shape, provide high-quality `location` and `venueAddress` fields for geocoding fallback.

## JsonApi Additional Contract Rules

`JsonApi` has extra schema-contract validation:

- new JsonApi sources should default to `schemaVersion = 2`
- `schemaVersion` must be `1` or `2` when provided
- advanced features require `schemaVersion = 2`
- `validationProfile = Advanced` requires `schemaVersion = 2`
- `validationProfile = Basic` cannot be used with advanced features
- `requestWorkflow` and `pagination` cannot both be configured
- `pagination.maxRequestsPerRun` must be greater than `0` when provided
- dynamic token acquisition requires `tokenConfig.dynamicAcquisition.endpoint`

## Type-Specific Minimum Shapes

### Ics

```json
{
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

### Rss

```json
{
  "extractionRules": {
    "title": "title",
    "startTime": "pubDate"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

### JsonApi

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

When the source splits date/time fields or exposes source timezone, a practical v2 shape often looks more like this:

```json
{
  "schemaVersion": 2,
  "eventArrayPath": "$.events[*]",
  "mappings": {
    "title": "$.title",
    "startTime": "$.startTime",
    "endTime": "$.endTime",
    "timeZone": "$.timeZone"
  },
  "responseTransforms": {
    "flattenPath": "$.events[*]",
    "fields": {
      "startTime": {
        "type": "Composite",
        "template": "{{startDate}} {{startClock}}",
        "inputs": {
          "startDate": "$.start.date",
          "startClock": "$.start.time"
        }
      }
    }
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

### HtmlLite

```json
{
  "eventCardSelector": "div.event-card",
  "mappings": {
    "title": "h3",
    "startTime": "time::attr(datetime)"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

## Common Invalid Patterns

These mistakes are common when generating schemas with chat models.

### Invalid: unsupported HtmlLite selector lists

```json
{
  "eventCardSelector": ".event-card, .featured-event",
  "mappings": {
    "title": ".event-title, h3, h2"
  }
}
```

Why invalid:
- HtmlLite does not support comma-separated selector lists.

Valid rewrite:

```json
{
  "eventCardSelector": "xpath=.//*[contains(@class,'event-card') or contains(@class,'featured-event')]",
  "mappings": {
    "title": "xpath=.//*[contains(@class,'event-title') or self::h3 or self::h2][1]"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

### Invalid: unsupported HtmlLite pagination shape

```json
{
  "pagination": {
    "enabled": true,
    "nextPageSelector": "a.next"
  }
}
```

Valid rewrite:

```json
{
  "pagination": {
    "type": "nextLink",
    "nextPageSelector": "a.next@href",
    "maxPages": 5
  }
}
```

### Invalid: unrecognized HtmlLite fields

```json
{
  "baseUrl": "https://example.org",
  "transforms": [
    {
      "field": "url",
      "type": "absoluteUrl"
    }
  ]
}
```

Why invalid:
- `baseUrl` and `transforms` are not part of the HtmlLite contract.
- `url` and `imageUrl` are already resolved to absolute URLs downstream when valid relative links are captured.

### Invalid: unsupported `regex:` mapping syntax

```json
{
  "mappings": {
    "startTime": "regex:(\\w+ \\d{1,2}, \\d{4})"
  }
}
```

Why invalid:
- HtmlLite mappings expect supported selector strings, XPath selectors, attribute extraction syntax, or `literal:` values.
- `regex:` is not part of the HtmlLite mapping DSL.

Use selector-based extraction instead (for example `time::attr(datetime)` or split `startDate` and `startTime`).

### Invalid: wrong field name

```json
{
  "detailPage": {
    "enabled": true,
    "detailMappings": {
      "address": ".event-address"
    }
  }
}
```

Valid rewrite:

```json
{
  "detailPage": {
    "enabled": true,
    "detailMappings": {
      "venueAddress": ".event-address"
    }
  }
}
```

## Validation Pass Criteria

Treat a draft as ready for admin review only when:

1. `validation.isSuccess` is `true`
2. `validation.totalEventsParsed` is non-zero and plausible
3. `validation.sampleEvents` show good coverage for title, time, URL, and location when available
4. the source type is still the best available source after testing

## Iteration Strategy For Any Source Type

Use a strict one-change-at-a-time loop:

1. Start with the smallest valid schema shape for the chosen type.
2. Run `test-fetch` and capture `isSuccess`, `totalEventsParsed`, `sampleEvents`, and `errorMessage`.
3. Change only one dimension per retry (selector/path, mapping, pagination, headers, or auth).
4. Re-run `test-fetch` immediately after each change.
5. Stop iterating when results are stable across repeated runs.

This avoids masking root causes and makes failures easier to explain in handoff.

## JsonApi: Zero-Event Diagnostic Order

When `validation.isSuccess = true` but `totalEventsParsed = 0` (or implausibly low), check in this order:

1. `feedUrl` returns the expected JSON payload at request time.
2. `eventArrayPath` points to repeated event items, not wrapper/config nodes.
3. If `responseTransforms` are used, `responseTransforms.flattenPath` points to the intended repeated event nodes.
4. Mapping paths are relative to each event item.
5. Required fields remain populated after mappings/transforms; zero parsed events can mean every candidate event was filtered out because `title` or `startTime` ended up empty.
6. If the source provides timezone, `timeZone` is mapped from source rather than assumed from runtime environment.
7. Required headers are present (for example, browser-like `User-Agent` when needed).
8. Pagination/workflow settings are not limiting extraction unexpectedly.

If a payload clearly contains events but extraction is empty, simplify `eventArrayPath` and mappings until one sample event parses, then build back up.

## Formal Schema Definition Status

CalendarApp now publishes contributor-facing JSON Schema documents for all supported `schemaDefinition` types.

See [Source Schema JSON Schema Files](json-schema-files.md).

Use these as the source of truth instead:

- the contributor-facing JsonApi v2 JSON Schema file for machine-readable authoring help
- the contributor-facing source-schema JSON Schema files for machine-readable authoring help
- the type-specific documentation on this site
- backend contract validation and error messages
- the backend source models and tests for supported fields and combinations

That means contributors and AI should treat the JSON Schema plus docs as authoring guidance, while backend validation remains the authoritative final contract.

## Contributor Handoff Checklist

Include:

- `sourceSchema.id`
- `feedUrl`
- final `schemaDefinition`
- validation summary
- any known limitations such as missing description, incomplete location, or pagination constraints

For `JsonApi`, also include:

- confirmed event collection path used in production schema
- any required headers or token dependencies
- parser or mapping limitations discovered during iteration
- geolocation note: `source-coordinates-used` or `geocode-fallback-expected`