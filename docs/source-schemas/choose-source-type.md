# Choose A Source Type

Use this page before writing `schemaDefinition`. CalendarApp supports four contributor-facing source types:

- `Ics`
- `Rss`
- `JsonApi`
- `HtmlLite`

Choose the most structured source that exposes the event data you need. In general, prefer this order:

1. `Ics`
2. `Rss`
3. `JsonApi`
4. `HtmlLite`

Structured feeds are easier to validate, less brittle to maintain, and less likely to break when a website redesigns its HTML.

## Decision Table

| Type | Use It When | Do Not Use It When | Required `schemaDefinition` Shape |
|---|---|---|---|
| `Ics` | The source publishes a valid `.ics` or iCalendar feed | You only have HTML or JSON | Validation-only object or `{}` |
| `Rss` | The source publishes RSS or Atom and each item acts like an event | The feed is a news/blog feed without event dates | `extractionRules` plus optional `validation` |
| `JsonApi` | The site exposes events through JSON responses | A better ICS or RSS feed already contains the same data | `eventArrayPath` and `mappings`, plus optional pagination/auth/workflow |
| `HtmlLite` | Event cards are present in static HTML without browser execution | The page is JS-rendered and the events are not present in initial HTML | `eventCardSelector` and `mappings`, plus optional detail/pagination |

## How To Recognize Each Type

### Ics

Look for:
- URLs ending in `.ics`
- `text/calendar` responses
- links labeled `iCal`, `Add to calendar`, or `Subscribe`

Typical example:

```text
https://example.org/events.ics
```

### Rss

Look for:
- RSS or Atom icons
- feed URLs ending in `.rss`, `.xml`, or `/feed`
- XML responses with `<rss>`, `<channel>`, `<item>`, or `<entry>`

Typical example:

```text
https://example.org/events/feed.rss
```

### JsonApi

Look for:
- browser network requests returning JSON arrays or event objects
- endpoints used by calendar widgets or SPA pages
- APIs with pagination, auth headers, or search parameters

Typical example:

```text
https://api.example.org/events
```

### HtmlLite

Use HtmlLite only when:
- event cards are present in View Source or the first HTML response
- titles, URLs, and date/time can be selected from DOM nodes
- you do not need JavaScript execution to reveal the event list

Typical example:

```text
https://example.org/events
```

## Common Contributor Workflow

1. Identify the best source type.
2. Build the smallest valid `schemaDefinition` for that type.
3. Validate with `POST /api/source-schemas/test-fetch`.
4. Submit with `POST /api/source-schemas/community-submissions`.
5. Re-test and re-submit until the validation result is stable.

## Where To Go Next

- Shared payload and validation rules: [Submission API And Validation](submission-api-and-validation.md)
- `Ics` details: [Ics Source Guide](ics.md)
- `Rss` details: [Rss Source Guide](rss.md)
- `JsonApi` details: [JsonApi Source Guide](json-api.md)
- `HtmlLite` details: [HtmlLite Overview](html-lite/overview.md)