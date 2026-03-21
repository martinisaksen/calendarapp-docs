# JsonApi Source Guide

Use `JsonApi` when event data comes from a JSON endpoint instead of ICS, RSS, or static HTML.

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

See also: [Submission API And Validation](submission-api-and-validation.md)