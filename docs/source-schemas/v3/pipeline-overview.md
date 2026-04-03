# Source Schema v3 Pipeline Overview

Source Schema v3 introduces a clean two-stage schemaDefinition contract:

- `calendar`: fetch and parse the source calendar/list payload
- `event`: enrich or transform each calendar item into final event fields

This model replaces implicit parser branching with explicit stage contracts.

## Goals

- Support flexible combinations for unexpected sources.
- Keep one canonical execution path for test-fetch and ingestion.
- Make parser composition explicit and statically validated.

## v3 shape

```json
{
  "schemaVersion": 3,
  "pipeline": {
    "calendar": {
      "type": "Ics",
      "parser": {}
    },
    "event": {
      "type": "Html",
      "input": {
        "mode": "calendarFieldUrl",
        "field": "eventUrl"
      },
      "parser": {
        "mappings": {
          "description": ".event-description",
          "imageUrl": ".hero img@src"
        }
      },
      "limits": {
        "sameHostOnly": true,
        "maxRequestsPerRun": 5
      }
    }
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

## Allowed calendar -> event combinations

- `Ics -> None`
- `Ics -> Html`
- `Ics -> Json`
- `Html -> Html`
- `Html -> Json`
- `Json -> Html`
- `Json -> Json`
- `Json -> None`
- `Rss -> None`
- `Rss -> Html`
- `Rss -> Json`

Combinations outside this matrix are rejected by schema validation.

## Stage contract summary

Calendar stage output must include enough fields for event stage input when event stage is not `None`.

Calendar stage fetch options may include:
- `fetch.authentication` for bearer, API key, or basic auth
- `fetch.requestHeaders` for non-restricted custom headers
- `fetch.pagination` for page, offset, or cursor traversal

Event stage input modes:
- `none`
- `calendarFieldUrl`
- `fixedUrl`

## Security defaults

- `event.limits.sameHostOnly` defaults to `true`.
- `event.limits.maxRequestsPerRun` is required when event type is `Html` or `Json`.
- Event URL modes must use `http` or `https` only.

## Related files

- [v3 JSON Schema](source-schema-v3.schema.json)
- [v3 examples](pipeline-examples.md)
