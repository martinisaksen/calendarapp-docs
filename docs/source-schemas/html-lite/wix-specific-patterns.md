# Wix-Specific Selector Patterns

## When To Use This Page

Use this page when the source is built on Wix and the event list is present in the initial HTML response.

This page is most useful when Wix component IDs are the only stable handles for repeated cards and child fields.

## Platform Signatures

Common Wix indicators:
- repeated DOM nodes with IDs like `comp-xxxxxxxx__item-...`
- card content identified by stable ID prefixes rather than semantic class names
- event detail links under paths such as `/event-details-registration/...`
- initial HTML may contain enough content for HtmlLite even when the page looks app-like in the browser

## Recommended Source Type By Scenario

- Use `HtmlLite` when repeated event cards are already present in the initial response.
- Prefer `JsonApi` if the page shell loads event data from a real structured JSON endpoint.
- Do not force CSS selectors when the stable structure is only expressible with XPath `starts-with()`.

## Common Extraction Patterns

### Event Card Selector

Wix uses stable ID prefixes. CSS selector `[id^="prefix"]` does not translate directly to the HtmlLite XPath-based parser path, so use XPath `starts-with()`.

Correct pattern:

```json
"eventCardSelector": ".//*[starts-with(@id, \"comp-mcsmgbbs__\")]"
```

### Title, Time, And URL Mappings

```json
{
  "mappings": {
    "title": ".//*[starts-with(@id, \"comp-mcsn738v__\")]//span",
    "startTime": ".//*[starts-with(@id, \"comp-jfx5gg4w__\")]/text()",
    "url": ".//a[1]/@href"
  }
}
```

### Typical Wix Card Shape

```html
<div id="comp-mcsmgbbs__item-abc123">
  <div id="comp-mcsn738v__item-abc123">
    <span>Event Title</span>
  </div>
  <div id="comp-jfx5gg4w__item-abc123">
    May 24, 2026 at 7:00pm
  </div>
  <a href="/event-detail?id=abc123">View Details</a>
</div>
```

## Datetime Pitfalls And Preferred Source Of Truth

Preferred extraction order for Wix sources:
1. structured datetime attributes or explicit event metadata
2. stable date/time text inside the repeated card
3. detail-page fields when list text is incomplete

Common pitfalls:
- visible date text may include ranges or marketing copy that does not parse cleanly
- `.text()` can produce multiple text nodes when the component has nested spans
- component IDs are stable by prefix but not by full generated suffix

## Anti-Patterns To Avoid

- Do not use raw CSS `[id^="..."]` syntax when the parser expects XPath.
- Do not lock selectors to the full generated component ID including the instance suffix.
- Do not assume the first text node is parseable without checking whitespace and range formatting.

## Validation Checklist

- verify parse count is close to the visible number of cards
- inspect `sampleEvents` for missing or duplicated titles caused by broad prefix matches
- verify `startTime` parses consistently across multiple cards, not just the first one
- compare parsed URLs against visible detail links

## Troubleshooting And Fallbacks

- If XPath matches too broadly, add child predicates or additional `starts-with()` constraints.
- If text extraction returns null or multiple fragments, target the inner `span` or the first text node explicitly.
- If list-page times are ranges such as `May 8 - 24, 2026`, use the guidance in [time-handling.md](time-handling.md#date-range-parsing--silent-failures).
- If the list is teaser-only, enable detail-page enrichment instead of overfitting card selectors.

## Minimal Working Example

```json
{
  "eventCardSelector": ".//*[starts-with(@id, \"comp-mcsmgbbs__\")]",
  "mappings": {
    "title": ".//*[starts-with(@id, \"comp-mcsn738v__\")]//span",
    "startTime": ".//*[starts-with(@id, \"comp-jfx5gg4w__\")]/text()",
    "url": ".//a[1]/@href"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

## AI Runbook Notes

When an AI agent is onboarding a Wix source:
1. Confirm the event cards exist in the first HTML response.
2. Identify stable component ID prefixes from repeated cards, not one-off wrapper IDs.
3. Build the minimal schema with `title`, `url`, and parseable `startTime`.
4. Validate against multiple sample cards before adding detail-page enrichment.
5. Escalate to `JsonApi` only if the real event data lives outside the initial HTML.
