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
