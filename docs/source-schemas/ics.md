# Ics Source Guide

Use `Ics` when the source publishes a valid iCalendar feed. This is usually the best option when available because the parser reads standard calendar fields directly.

For a machine-readable authoring contract, use the contributor-facing [Ics JSON Schema](ics.schema.json).

## When To Use It

Use `Ics` when:

- the URL returns a `.ics` file or `text/calendar`
- the feed already includes title, start, end, location, description, or event URL
- the source has stable event UIDs

Do not use `Ics` when you only have HTML or JSON.

## Minimal `schemaDefinition`

`Ics` does not require field mappings. A validation-only schema is enough for most sources.

The submission envelope is still required at the top level:
- `type`
- `feedUrl`
- `schemaDefinition` (JSON string)

Do not submit ICS payloads with top-level `url` or `eventMapping` fields.

```json
{
  "validation": {
    "requiredFields": ["title", "startTime"],
    "minEventsPerFetch": 1,
    "maxEventsPerFetch": 10000
  }
}
```

## Test-Fetch Example

```json
{
  "type": "Ics",
  "feedUrl": "https://example.org/events.ics",
  "schemaDefinition": "{\"validation\":{\"requiredFields\":[\"title\",\"startTime\"]}}"
}
```

## Wrong Vs Right Payload Shape

The `Ics` `schemaDefinition` may only contain `validation`. **No other property is supported** — the parser reads RFC 5545 fields (`UID`, `SUMMARY`, `DTSTART`, `DTEND`, `DESCRIPTION`, `LOCATION`, `URL`) directly from the feed. Do not add `mappings`, `eventMapping`, or any other field.

Wrong — top-level `url` instead of `feedUrl`, `eventMapping` inside `schemaDefinition`:

```json
{
  "type": "ics",
  "url": "https://calendar.google.com/calendar/ical/lacentercalendar%40gmail.com/public/basic.ics",
  "eventMapping": {
    "id": "uid",
    "title": "summary",
    "description": "description",
    "location": "location",
    "startTime": "dtstart",
    "endTime": "dtend",
    "url": "url"
  }
}
```

Also wrong — `mappings` is equally unsupported in `schemaDefinition`:

```json
{
  "type": "ics",
  "url": "https://calendar.google.com/calendar/ical/lacentercalendar%40gmail.com/public/basic.ics",
  "mappings": {
    "id": "uid",
    "title": "summary",
    "description": "description",
    "location": "location",
    "startTime": "dtstart",
    "endTime": "dtend",
    "allDay": "allDay"
  }
}
```

Right for this API contract (validation-only `schemaDefinition`):

```json
{
  "type": "Ics",
  "feedUrl": "https://calendar.google.com/calendar/ical/lacentercalendar%40gmail.com/public/basic.ics",
  "schemaDefinition": "{}"
}
```

Also valid when you want explicit validation guards:

```json
{
  "type": "Ics",
  "feedUrl": "https://calendar.google.com/calendar/ical/lacentercalendar%40gmail.com/public/basic.ics",
  "schemaDefinition": "{\"validation\":{\"requiredFields\":[\"title\",\"startTime\"],\"minEventsPerFetch\":1}}"
}
```

Summary of rules:

| Field | Correct location | Notes |
|---|---|---|
| `feedUrl` | top-level request field | not `url` |
| `type` | top-level request field | value is `Ics` (capital I) |
| `schemaDefinition` | top-level request field | compact JSON string |
| `validation` | inside `schemaDefinition` only | only allowed property |
| `mappings` | **nowhere** | unsupported for `Ics` |
| `eventMapping` | **nowhere** | unsupported for `Ics` |
| `url` | **nowhere** | use `feedUrl` at top level |

## Community Submission Example

```json
{
  "name": "Example Events ICS",
  "description": "Public iCalendar feed for Example Events",
  "type": "Ics",
  "feedUrl": "https://example.org/events.ics",
  "schemaDefinition": "{\"validation\":{\"requiredFields\":[\"title\",\"startTime\"]}}",
  "metadata": {
    "location": "Example City",
    "category": "community",
    "region": "WA"
  }
}
```

## Requirements And Validation Notes

- `feedUrl` must return a valid ICS feed
- `title` and `startTime` should be present on real events
- `UID` is strongly preferred because it improves identity stability
- missing `DtEnd` values may be normalized to a default duration by the parser
- all-day and timezone data should be checked in `sampleEvents`

## Recommended Contributor Workflow

1. Confirm the feed is really ICS.
2. Start with the minimal validation object above.
3. Run `test-fetch`.
4. Check that titles and start times are populated and plausible.
5. Submit with `community-submissions`.

## Common Failure Modes

- The URL is not actually an ICS response
- The feed is empty or stale
- Titles are blank or missing on some events
- Timezone handling in the source is inconsistent

See also: [Submission API And Validation](submission-api-and-validation.md)