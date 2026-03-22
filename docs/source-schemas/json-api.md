# JsonApi Source Guide

Use `JsonApi` when event data comes from a JSON endpoint instead of ICS, RSS, or static HTML.

If a usable event JSON endpoint exists, `JsonApi` is the correct type even when the page visually renders event cards.

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

## Optional Features

Add these only when needed:

- `eventArrayPaths` for alternate payload shapes
- `pagination` for offset, cursor, or page-based APIs
- `authentication` for API key, bearer, or basic auth flows
- `requestHeaders` for required headers such as `User-Agent`
- `requestWorkflow` for multi-step request chains
- `schemaVersion = 2` for advanced features such as tokens, query templates, response transforms, or windowed pagination

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

## Recommended Contributor Workflow

1. Find the JSON endpoint that returns real event objects.
2. Start from the closest template or the minimal shape above.
3. Map `title` and `startTime` first, then add URL, ID, and location.
4. Add pagination or auth only when the source actually requires it.
5. Run `test-fetch`.
6. Inspect `sampleEvents` carefully for field completeness.
7. Submit the draft with `community-submissions`.

## Common Failure Modes

- wrong `eventArrayPath`
- mappings that point to a scalar outside the current event item
- unnecessary advanced features when a simpler schema would work
- pagination configured together with requestWorkflow
- auth headers or tokens copied into the wrong part of the schema
- choosing HtmlLite even though a usable JSON endpoint exists

See also: [Submission API And Validation](submission-api-and-validation.md)