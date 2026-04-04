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

For Html calendar parsers, calendar parser options can also include `multiDateExpansion`.
This allows one calendar card to emit multiple occurrence events from a date-list field.
When combined with `detailPage.occurrenceSelector`, expansion determines event count and detail occurrence matching only refines emitted events.

Event stage input modes:
- `none`
- `calendarFieldUrl`
- `fixedUrl`

Current implementation note:
- In current phase-2 scaffolding, `pipeline.event.input.field` is restricted to `eventUrl` when `event.type` is `Html` or `Json`.

## Event Input Modes Reference

The `pipeline.event.input.mode` determines how per-event details are obtained. Choose based on your source structure:

| Mode | Use Case | Example | Input Config |
|------|----------|---------|---------------|
| `none` | Events are already complete from calendar stage; no detail page needed | ICS feed with full event data | `{ "mode": "none" }` |
| `calendarFieldUrl` | Calendar stage outputs a URL field (e.g., `eventUrl`); fetch detail page per event | Calendar list has link to detail page in each item | `{ "mode": "calendarFieldUrl", "field": "eventUrl" }` |
| `fixedUrl` | All events use the same detail page URL pattern; substitute event ID into template | Detail URL is predictable: `https://example.com/events/{eventId}` | `{ "mode": "fixedUrl", "url": "https://example.com/events/{id}" }` |

### Decision Tree

1. **Does your calendar stage output complete event data?**
  - Yes → Keep `event` lightweight, but still choose an allowed pair for your calendar type
  - For `Ics` and `Rss`, `event.type: "None"` is valid
  - For `Html`, choose `event.type: "Html"` or `"Json"` per the allowlist
  - No → Continue to step 2

2. **Does each calendar item have a link to its detail page?**
  - Yes → Use `mode: "calendarFieldUrl"`; in current phase-2 scaffolding map and use `eventUrl`
   - No → Continue to step 3

3. **Can you construct detail URLs from event IDs?**
  - Yes → Use `mode: "fixedUrl"`; provide URL template with `{id}` placeholder and validate behavior in your target environment
   - No → Check if detail page enrichment is needed; if not, use `mode: "none"`

### Examples

**Ics -> None (Complete calendar feed)**
```json
{
  "pipeline": {
    "calendar": { "type": "Ics" },
    "event": { "type": "None", "input": { "mode": "none" } }
  }
}
```

**Html -> Html with detail enrichment (Calendar has detail links)**
```json
{
  "pipeline": {
    "calendar": { "type": "Html", "parser": { "mappings": { "eventUrl": "a.event-link@href" } } },
    "event": { "type": "Html", "input": { "mode": "calendarFieldUrl", "field": "eventUrl" } }
  }
}
```

**Json -> Json with fixed template (Predictable detail URLs)**
```json
{
  "pipeline": {
    "calendar": { "type": "Json" },
    "event": { "type": "Json", "input": { "mode": "fixedUrl", "url": "https://api.example.com/events/{id}" } }
  }
}
```

## Security defaults

- `event.limits.sameHostOnly` defaults to `true`.
- `event.limits.maxRequestsPerRun` is required when event type is `Html` or `Json`.
- Event URL modes must use `http` or `https` only.

## Related files

- [v3 JSON Schema](source-schema-v3.schema.json)
- [v3 examples](pipeline-examples.md)
