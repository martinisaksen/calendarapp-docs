# Ics Source Guide

Use `Ics` when the source publishes a valid iCalendar feed. This is usually the best option when available because the parser reads standard calendar fields directly.

For a machine-readable authoring contract, use the contributor-facing [Ics JSON Schema](ics.schema.json).

## ⚠️ ONLY Valid Format For ICS

The `Ics` source schema submission to CalendarApp **has exactly this structure and no other**:

```json
{
  "name": "<source name>",
  "description": "<source description>",
  "type": "Ics",
  "feedUrl": "<url-that-returns-ics-feed>",
  "schemaDefinition": "<json-string-with-only-validation>",
  "metadata": {
    "location": "<location>",
    "category": "<category>",
    "region": "<region>"
  }
}
```

The `schemaDefinition` is a compact JSON string containing `validation` and optional `eventEnrichment`:

```json
{
  "validation": {
    "requiredFields": ["title", "startTime"],
    "minEventsPerFetch": 1,
    "maxEventsPerFetch": 10000
  },
  "eventEnrichment": {
    "enabled": true,
    "parser": "GenericHtml",
    "maxFetchesPerRun": 5,
    "sameHostOnly": true
  }
}
```

or for test-fetch, simply:

```json
{}
```

**Forbidden at all levels:** `url`, `mappings`, `eventMapping`, `transforms`, `recurrence`, `sourceType`, `eventDefaults`, `allDay`.

## When To Use It

Use `Ics` when:

- the URL returns a `.ics` file or `text/calendar`
- the feed already includes title, start, end, location, description, or event URL
- the source has stable event UIDs

Do not use `Ics` when you only have HTML or JSON.

## ICS Field Mapping

The parser reads standard RFC 5545 VEVENT fields automatically. No `mappings` configuration is needed or supported.

| ICS VEVENT Field | Maps To | Notes |
|---|---|---|
| `SUMMARY` | `title` | Required. Events without SUMMARY are skipped. |
| `DTSTART` | `startTime` | Required. Events without DTSTART are skipped. |
| `DTEND` | `endTime` | Optional. Defaults to startTime + 1 hour when absent. |
| `DESCRIPTION` | `description` | Optional. If `URL` is absent or resolves back to the source feed, the first http/https URL found in DESCRIPTION is used as `eventUrl`. |
| `LOCATION` | `location` | Optional. Used for geocoding fallback when coordinates are absent. |
| `URL` | `eventUrl` | Optional. Relative URLs are resolved against the source feed. If the resolved URL points back to the source feed, it is ignored and DESCRIPTION fallback is used instead. |
| `UID` | identity key | Strongly recommended. Improves deduplication stability. |
| `TZID` (on DTSTART) | `timeZone` | Carried through from the timezone identifier on the start field. |

> **Note on `URL` vs `DESCRIPTION`:** Some sources use the `URL` VEVENT field for a shared category link rather than a per-event deep link. The parser resolves relative URLs against the source feed and ignores them when they normalize to the same target as the feed itself, even if query parameter order differs. In that case, if the `DESCRIPTION` field contains a bare absolute URL, the parser uses it as `eventUrl` automatically. You do not need to configure this — it is built-in fallback behavior.

## Test-Fetch Example

```json
{
  "type": "Ics",
  "feedUrl": "https://example.org/events.ics",
  "schemaDefinition": "{}"
}
```

## Optional: HTML Detail Enrichment For ICS

Some ICS feeds provide only partial event detail in VEVENT fields, while full details live on an event page linked by `eventUrl`.

CalendarApp supports optional HTML detail enrichment for `Ics` sources during ingestion and test-fetch.

- The feed remains the calendar source of truth.
- Detail pages are fetched only when enrichment is enabled.
- Enrichment fills weak or missing fields such as `description`, `venueAddress`, `imageUrl`, and (when missing) `location`.
- Strong ICS fields (`title`, `startTime`, `endTime`, `UID`) are not overridden.

Enable with schema-level configuration in `schemaDefinition`:

```json
{
  "eventEnrichment": {
    "enabled": true,
    "parser": "GenericHtml",
    "maxFetchesPerRun": 5,
    "sameHostOnly": true
  }
}
```

Use `parser: "HtmlLite"` only when you also provide `htmlLiteMappings`.

Recommended starting values:


- `enabled = true`
- `maxFetchesPerRun = 25` for ingestion
- `sameHostOnly = true`

For test-fetch parity with runtime ingestion behavior, send the same `eventEnrichment` block in the payload used for `POST /api/source-schemas/test-fetch`.

## Wrong Vs Right Payload Shape

The `Ics` `schemaDefinition` supports `validation` and optional `eventEnrichment`. The parser still reads RFC 5545 fields (`UID`, `SUMMARY`, `DTSTART`, `DTEND`, `DESCRIPTION`, `LOCATION`, `URL`) directly from the feed. Do not add `mappings`, `eventMapping`, or transform blocks.

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
| `validation` | inside `schemaDefinition` | optional validation constraints |
| `eventEnrichment` | inside `schemaDefinition` | optional detail-page enrichment settings |
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