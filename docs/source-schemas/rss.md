# Rss Source Guide

Use `Rss` when the source publishes RSS or Atom and each item behaves like an event record.

## When To Use It

Use `Rss` when:

- the response is XML with `<rss>`, `<channel>`, `<item>`, or Atom `<entry>` nodes
- titles and dates can be extracted from feed elements
- the feed is event-oriented rather than general news content

Do not use `Rss` when the feed is missing event dates or when a better ICS feed already exists.

## Required `schemaDefinition` Shape

`Rss` requires `extractionRules`.

```json
{
  "extractionRules": {
    "title": "title",
    "description": "description",
    "startTime": "pubDate",
    "link": "link",
    "guid": "guid",
    "location": "location"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

## How To Structure The Payload

- `type` must be `Rss`
- `feedUrl` points at the XML feed
- `schemaDefinition.extractionRules` maps output fields to XML elements or attributes per item
- `validation.requiredFields` should almost always include `title` and `startTime`

## Requirements And Validation Notes

- `extractionRules` is required
- each extraction rule value should resolve against each feed item
- `startTime` must parse as a real date/time
- `guid` is recommended for stable identity
- `link` should point to the event detail page when available

## Recommended Contributor Workflow

1. Inspect the feed structure and confirm that each item is an event.
2. Map `title` and `startTime` first.
3. Add `description`, `link`, `guid`, and `location`.
4. Run `test-fetch` and verify that the sample items parse correctly.
5. Submit the draft with `community-submissions`.

## Common Failure Modes

- using a general-purpose blog/news feed instead of an event feed
- `startTime` points to a non-date field
- missing `guid` leads to unstable identity
- element names differ between RSS and Atom variants

See also: [Submission API And Validation](submission-api-and-validation.md)