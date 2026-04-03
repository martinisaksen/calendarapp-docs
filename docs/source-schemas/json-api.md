# JsonApi Source Guide

Use `JsonApi` when event data is delivered from a JSON endpoint.

## Current Contract

JsonApi source submissions now use the v3 pipeline contract only.

- `schemaVersion` must be `3`
- `pipeline.calendar` and `pipeline.event` are required
- JSON list parsing config belongs in `pipeline.calendar.parser`
- request auth/headers/pagination belong in stage `fetch` nodes

For historical v1/v2 reference material, see [JsonApi V2 JSON Schema](json-api-v2-json-schema.md).

## When To Use JsonApi

Use `JsonApi` when:

- the website or widget fetches JSON event payloads
- event objects are in arrays or nested containers
- authenticated or paginated API requests are needed to fetch events

Do not use `JsonApi` when equivalent ICS or RSS feeds already provide the same data more simply.

## Minimum v3 JsonApi Shape

```json
{
  "schemaVersion": 3,
  "pipeline": {
    "calendar": {
      "type": "Json",
      "parser": {
        "eventArrayPath": "$.events[*]",
        "mappings": {
          "title": "$.title",
          "startTime": "$.start"
        },
        "validation": {
          "requiredFields": ["title", "startTime"]
        }
      }
    },
    "event": {
      "type": "None",
      "input": { "mode": "none" },
      "parser": {}
    }
  }
}
```

## Common Optional Features

Add these only when needed:

- `pipeline.calendar.fetch.pagination` for page/offset/cursor APIs
- `pipeline.calendar.fetch.authentication` for bearer/api-key/basic auth
- `pipeline.calendar.fetch.requestHeaders` for required headers
- `pipeline.calendar.parser.responseTransforms` for shaping split date/time fields

## Practical Pattern: Split Date And Time

```json
{
  "schemaVersion": 3,
  "pipeline": {
    "calendar": {
      "type": "Json",
      "parser": {
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
        }
      }
    },
    "event": {
      "type": "None",
      "input": { "mode": "none" },
      "parser": {}
    }
  }
}
```

## Validation Notes

- `schemaVersion` must be `3`
- `pipeline.calendar.type` should be `Json` for JsonApi sources
- `pipeline.calendar.parser.eventArrayPath` is required
- `pipeline.calendar.parser.mappings` should be non-empty
- `requestWorkflow` and pagination cannot both be configured in the same stage

## Recommended Workflow

1. Find the JSON endpoint returning real event objects.
2. Start from the minimum v3 shape.
3. Map `title` and `startTime` first.
4. Add URL, location, and optional enrichment fields.
5. Run `test-fetch` and inspect sample events.
6. Submit draft via community submissions.
