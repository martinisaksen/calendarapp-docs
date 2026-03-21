# Ics Source Guide

Use `Ics` when the source publishes a valid iCalendar feed. This is usually the best option when available because the parser reads standard calendar fields directly.

## When To Use It

Use `Ics` when:

- the URL returns a `.ics` file or `text/calendar`
- the feed already includes title, start, end, location, description, or event URL
- the source has stable event UIDs

Do not use `Ics` when you only have HTML or JSON.

## Minimal `schemaDefinition`

`Ics` does not require field mappings. A validation-only schema is enough for most sources.

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