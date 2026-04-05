# Squarespace-Specific Patterns

## When To Use This Page

Use this page when the source is a Squarespace events listing and the initial HTML contains repeatable event cards.

This page is especially relevant when the visible card text splits date and time across separate nodes, but the card also includes a Google Calendar export link with canonical UTC timestamps.

## Platform Signatures

Common Squarespace indicators:
- page markup includes `eventlist` class names such as `eventlist-event` or `eventlist-column-info`
- event cards are rendered in the initial HTML response
- each card links to a detail page under a path like `/events/<slug>`
- each card may expose a Google Calendar export URL containing a `dates=` query parameter

## Recommended Source Type By Scenario

- Use `HtmlLite` when the initial HTML already contains repeatable event cards.
- Prefer `Ics`, `Rss`, or `JsonApi` if the platform exposes a real structured feed with event occurrence dates.
- Do not choose `Rss` just because Squarespace exposes an RSS endpoint; validate that the feed contains actual event occurrence dates rather than publish dates.

## Common Extraction Patterns

### Card Selector

Typical repeating card selector:

```json
"eventCardSelector": "article.eventlist-event.eventlist-event--upcoming"
```

### Basic Mappings

Typical card mappings:

```json
{
  "mappings": {
    "title": ".eventlist-title-link",
    "url": ".eventlist-title-link@href",
    "eventUrl": ".eventlist-title-link@href",
    "description": ".eventlist-column-info",
    "imageUrl": ".eventlist-column-thumbnail img@src"
  }
}
```

### Canonical Datetime From Google Calendar Link

When the visible time text is incomplete or split, map the export link first and derive `startTime` and `endTime` from the `dates=` query value.

Example pattern:

```json
{
  "mappings": {
    "startTime": ".eventlist-meta-export-google@href",
    "endTime": ".eventlist-meta-export-google@href"
  },
  "fieldTransforms": {
    "startTime": [
      { "type": "RegexReplace", "value": "^.*[?&]dates=([0-9]{8}T[0-9]{6}Z)/([0-9]{8}T[0-9]{6}Z).*$", "replacement": "$1" },
      { "type": "RegexReplace", "value": "^([0-9]{4})([0-9]{2})([0-9]{2})T([0-9]{2})([0-9]{2})([0-9]{2})Z$", "replacement": "$1-$2-$3T$4:$5:$6Z" }
    ],
    "endTime": [
      { "type": "RegexReplace", "value": "^.*[?&]dates=([0-9]{8}T[0-9]{6}Z)/([0-9]{8}T[0-9]{6}Z).*$", "replacement": "$2" },
      { "type": "RegexReplace", "value": "^([0-9]{4})([0-9]{2})([0-9]{2})T([0-9]{2})([0-9]{2})([0-9]{2})Z$", "replacement": "$1-$2-$3T$4:$5:$6Z" }
    ]
  }
}
```

Use this only when the export link is present consistently and the timestamps are clearly canonical.

## Datetime Pitfalls And Preferred Source Of Truth

Preferred extraction order for Squarespace events:
1. Google Calendar export URL timestamps
2. `time[datetime]` or structured datetime attributes
3. visible date/time text on the card

Common pitfalls:
- RSS `pubDate` can reflect publish time rather than event occurrence time
- visible card text may split month, day, and time into separate elements
- list pages may omit end time even when the export link contains both timestamps

## Anti-Patterns To Avoid

- Do not reconstruct datetimes by stitching together unrelated marketing text fragments if the export link already provides the canonical timestamp.
- Do not use a broad regex against the entire card HTML when a single `href` attribute is available.
- Do not assume the Squarespace RSS feed is event-safe without checking real sample items.

## Validation Checklist

- verify `validation.totalEventsParsed` is close to the visible upcoming card count
- confirm `sampleEvents` show the correct event occurrence date, not article publish date
- confirm the exported URLs resolve to the same host or an expected detail host
- confirm timezone assumptions match the venue locale when UTC timestamps are converted downstream

## Troubleshooting And Fallbacks

- If the Google export link is missing on some cards, fall back to `time[datetime]` or visible card time.
- If the listing cards are teaser-only, keep the list-page schema minimal and use detail-page enrichment for description, venue, or image.
- If the feed exposes paginated older events, add pagination only after the first page is parsing correctly.

## Minimal Working Example

```json
{
  "eventCardSelector": "article.eventlist-event.eventlist-event--upcoming",
  "mappings": {
    "title": ".eventlist-title-link",
    "url": ".eventlist-title-link@href",
    "startTime": ".eventlist-meta-export-google@href",
    "endTime": ".eventlist-meta-export-google@href"
  },
  "fieldTransforms": {
    "startTime": [
      { "type": "RegexReplace", "value": "^.*[?&]dates=([0-9]{8}T[0-9]{6}Z)/([0-9]{8}T[0-9]{6}Z).*$", "replacement": "$1" },
      { "type": "RegexReplace", "value": "^([0-9]{4})([0-9]{2})([0-9]{2})T([0-9]{2})([0-9]{2})([0-9]{2})Z$", "replacement": "$1-$2-$3T$4:$5:$6Z" }
    ]
  },
  "validation": {
    "requiredFields": ["title", "startTime", "url"]
  }
}
```

## AI Runbook Notes

When an AI agent is onboarding a Squarespace source:
1. Prove the cards exist in the initial HTML before committing to `HtmlLite`.
2. Probe RSS and ICS endpoints, but reject them if they do not carry event occurrence dates.
3. Check for export links on the card before attempting visible-text reconstruction.
4. Validate a minimal schema with `title`, `url`, and parseable `startTime` first.
5. Add `endTime`, detail-page enrichment, and pagination only after the initial parse is correct.