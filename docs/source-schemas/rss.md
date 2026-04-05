# Rss Source Guide

Use `Rss` when the source publishes RSS or Atom and each item behaves like an event record.

For a machine-readable authoring contract, use the contributor-facing [Rss JSON Schema](rss.schema.json).

## When To Use It

Use `Rss` when:

- the response is XML with `<rss>`, `<channel>`, `<item>`, or Atom `<entry>` nodes
- titles and dates can be extracted from feed elements
- the feed is event-oriented rather than general news content

Do not use `Rss` when the feed is missing event dates or when a better ICS feed already exists.

## Required `schemaDefinition` Shape

`Rss` uses v3 pipeline. The RSS-specific contract applies to `pipeline.calendar.parser` and requires `mappings`.

```json
{
  "mappings": {
    "title": "title",
    "description": "description",
    "startTime": "pubDate",
    "eventUrl": "link",
    "id": "guid",
    "location": "location"
  },
  "filters": {
    "includeAny": [
      { "field": "title", "value": "showcase", "matchType": "contains" }
    ],
    "excludeAny": [
      { "field": "description", "value": "recap", "matchType": "contains" }
    ]
  },
  "validation": {
    "minEventsPerFetch": 1,
    "requiredFields": ["title", "startTime"]
  }
}
```

## How To Structure The Payload

- `type` must be `Rss`
- `feedUrl` points at the XML feed
- `schemaDefinition.pipeline.calendar.parser.mappings` maps output fields to XML elements or attributes per item
- `validation.minEventsPerFetch` should default to `1`
- `validation.requiredFields` should default to `title` and `startTime`

## Requirements And Validation Notes

- `mappings` is required
- each extraction rule value should resolve against each feed item
- `startTime` must parse as a real date/time
- `id` is recommended for stable identity (map from `guid` when available)
- `eventUrl` should point to the event detail page when available (map from `link` or `url`)
- `location` or `endTime` should be added to `validation.requiredFields` only when the feed is consistently complete and dropping missing items is intentional

## Filtering Expectations

- Prefer an event-only or category-specific upstream feed URL when the publisher offers one.
- If upstream filtering is not sufficient, use `pipeline.calendar.parser.filters` with deterministic include/exclude rules.
- Keep filter rules explicit and reviewable (`field`, `value`, optional `matchType`, optional `caseSensitive`).
- If the feed mixes event items with blog/news/announcement content and cannot be narrowed upstream or with deterministic parser filters, choose a different source URL or source type.
- If you enable detail enrichment, make sure `link` or `url` is mapped because the detail stage depends on it even though it is not part of `requiredFields`.

## Recommended Contributor Workflow

1. Inspect the feed structure and confirm that each item is an event.
2. If the publisher offers multiple RSS endpoints, pick the narrowest event-oriented feed before authoring the schema.
3. Map `title` and `startTime` first.
4. Add `validation.minEventsPerFetch = 1` and `validation.requiredFields = ["title", "startTime"]`.
5. Add `description`, `eventUrl`, `id`, and `location`.
6. Run `test-fetch` and verify that the sample items parse correctly.
7. Tighten `requiredFields` only if the feed is consistently complete and you intentionally want incomplete items dropped.
8. Add deterministic `filters.includeAny`/`filters.excludeAny` rules if non-event items remain after source-side narrowing.
9. Submit the draft with `community-submissions`.

## Common Failure Modes

- using a general-purpose blog/news feed instead of an event feed
- `startTime` points to a non-date field
- missing `id` mapping leads to unstable identity
- element names differ between RSS and Atom variants
- using broad keyword filters without deterministic review criteria
- adding `location` or `endTime` to `requiredFields` too early and filtering out otherwise valid events

See also: [Submission API And Validation](submission-api-and-validation.md)