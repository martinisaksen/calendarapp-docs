# Source Schema v3 Pipeline Examples

## Ics -> None

```json
{
  "schemaVersion": 3,
  "pipeline": {
    "calendar": {
      "type": "Ics",
      "parser": {}
    },
    "event": {
      "type": "None",
      "input": {
        "mode": "none"
      },
      "parser": {}
    }
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

## Ics -> Html

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
          "description": ".event-details",
          "venueAddress": ".venue-address",
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

## Json -> Json

```json
{
  "schemaVersion": 3,
  "pipeline": {
    "calendar": {
      "type": "Json",
      "parser": {
        "eventArrayPath": "$.events[*]",
        "mappings": {
          "title": "$.name",
          "startTime": "$.start",
          "detailUrl": "$.detailUrl"
        }
      }
    },
    "event": {
      "type": "Json",
      "input": {
        "mode": "calendarFieldUrl",
        "field": "detailUrl"
      },
      "parser": {
        "eventArrayPath": "$.items[*]",
        "mappings": {
          "description": "$.description",
          "venueName": "$.venue.name",
          "venueAddress": "$.venue.address"
        }
      },
      "limits": {
        "sameHostOnly": true,
        "maxRequestsPerRun": 10
      }
    }
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```
