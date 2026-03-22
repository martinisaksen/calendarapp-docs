# Schema Basics

## Minimal Required Fields

- eventCardSelector
- mappings.title
- mappings.startTime
- validation.requiredFields includes title and startTime

## Supported Mapping Keys

| Key | Description |
|---|---|
| `id` | Stable external ID for deduplication. If absent, falls back to `url`, then a random GUID. **Always map this.** |
| `title` | Event title |
| `startTime` | Event start datetime |
| `endTime` | Event end datetime. Defaults to startTime + 1 hour if absent. |
| `startDate` | Optional helper field for pages that split the calendar date from the visible time. When present, `startTime` may be a time-only value or a range like `10:30 AM - 11:30 AM`. |
| `endDate` | Optional helper field used with `endTime` when the end date differs from the start date or the page exposes end date separately. |
| `description` | Event description |
| `location` | Location string |
| `venueName` | Venue name |
| `venueAddress` | Venue address |
| `url` | Event URL (resolved to absolute) |
| `imageUrl` | Image URL (resolved to absolute) |
| `recurrenceRule` | RFC-5545 RRULE string (e.g. `FREQ=WEEKLY;BYDAY=TH`) or human-readable text (e.g. `Recurring weekly on Thursday`). If omitted, the event is stored as a single instance. |

## Minimal Example

```json
{
  "eventCardSelector": "div.event-teaser",
  "mappings": {
    "title": "h3",
    "startTime": "xpath=.//time[@datetime]::attr(datetime)",
    "url": "a::attr(href)"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

## Minimal Progression (Recommended)

Start small and validate each step.

Important:
- `literal:` time values are bootstrap aids, not final handoff values
- replace or override them with real extracted times before final contributor handoff whenever possible
- do not add fields that are not part of the HtmlLite contract

Step 1:

```json
{
  "eventCardSelector": "h2.event__title",
  "mappings": {
    "title": "a",
    "url": "a@href",
    "startTime": "literal:2026-01-01T00:00:00-08:00"
  },
  "validation": {
    "requiredFields": ["title", "startTime", "url"]
  }
}
```

Step 2: add `id`, pagination, and detail-page mappings after Step 1 returns non-zero events.

## Common Invalid Shapes

### Do not add unsupported top-level fields

Invalid:

```json
{
  "sourceName": "Example Events",
  "baseUrl": "https://example.org",
  "eventCardSelector": "div.event-card",
  "mappings": {
    "title": "h3",
    "startTime": ".date"
  }
}
```

Why invalid:
- `sourceName` and `baseUrl` are not part of `schemaDefinition`

### Do not add `transforms`

Invalid:

```json
{
  "eventCardSelector": "div.event-card",
  "mappings": {
    "url": "a@href"
  },
  "transforms": [
    {
      "field": "url",
      "type": "absoluteUrl"
    }
  ]
}
```

Why invalid:
- `transforms` is not part of the HtmlLite schema

### Use `venueAddress`, not `address`

Invalid:

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

Valid:

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

## Split Date And Time Is Usually Better Than Guessing

If a page shows date and time separately, prefer `startDate` plus `startTime` over guessing one combined selector.

Example:

```json
{
  "eventCardSelector": "article.event",
  "mappings": {
    "id": "h2 a@href",
    "title": "h2 a",
    "startDate": ".event__date",
    "startTime": ".event__time",
    "url": "h2 a@href"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

Do not map `startTime` to a date-only node and `endTime` to a time-only node unless you have proven that structure is valid for the page.

## Full Example

```json
{
  "eventCardSelector": "div.event-teaser",
  "mappings": {
    "id": "a::attr(href)",
    "title": "h3",
    "startTime": "xpath=.//time[@datetime]::attr(datetime)",
    "description": ".event-teaser__description",
    "location": ".event-teaser__address",
    "venueName": "literal:My Venue",
    "venueAddress": ".event-teaser__address",
    "url": "a::attr(href)",
    "imageUrl": "img::attr(src)",
    "recurrenceRule": ".event-recurrence::attr(data-rrule)"
  },
  "detailPage": {
    "enabled": false
  },
  "pagination": {
    "type": "nextLink",
    "nextPageSelector": ".pager-next a::attr(href)",
    "maxPages": 10
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

## Split Date And Time Example

Use helper date mappings when the page exposes date and time in separate elements.

```json
{
  "eventCardSelector": "article.event",
  "mappings": {
    "id": "h2 a@href",
    "title": "h2 a",
    "startDate": ".event__date",
    "startTime": ".event__time",
    "location": ".event__location",
    "url": "h2 a@href"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

In this mode, HtmlLite combines `startDate` with `startTime`. If `startTime` contains a visible range such as `10:30 AM - 11:30 AM`, the parser uses the first time for `StartTime` and the second time for `EndTime`.

## Known-Good Bootstrap Pattern (Multnomah Example)

Use this pattern for Drupal list pages where real time precision is on detail pages:

```json
{
  "eventCardSelector": "h2.event__title",
  "mappings": {
    "id": "a@href",
    "title": "a",
    "url": "a@href",
    "startTime": "literal:2026-01-01T00:00:00-08:00"
  },
  "detailPage": {
    "enabled": true,
    "linkSelector": "a@href",
    "detailMappings": {
      "title": "h1",
      "url": "link[rel='canonical']@href",
      "startDate": ".event__date",
      "startTime": ".event__time",
      "location": ".event__location",
      "description": ".event__body",
      "imageUrl": ".field--name-field-event-image img@src"
    }
  },
  "pagination": {
    "type": "queryIncrement",
    "parameter": "page",
    "start": 0,
    "step": 1,
    "maxPages": 3
  },
  "validation": {
    "requiredFields": ["title", "startTime", "url"]
  }
}
```

This schema is a validated bootstrap pattern for non-zero event parsing on the Multnomah source.
