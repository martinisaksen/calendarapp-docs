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

## Hard Rules

Use these rules before you commit to a source type:

1. If a real event JSON endpoint exists, choose `JsonApi`, not `HtmlLite`.
2. If a real ICS feed exists, choose `Ics`, not `Rss`, `JsonApi`, or `HtmlLite`.
3. Use `HtmlLite` only when the initial HTML response already contains repeated event cards and no better structured feed exists.
4. Do not choose a type based only on what the page looks like in the browser. Choose it based on the best underlying source that can be validated.

## Decision Table

| Type | Use It When | Do Not Use It When | Required `schemaDefinition` Shape |
|---|---|---|---|
| `Ics` | The source publishes a valid `.ics` or iCalendar feed | You only have HTML or JSON | Validation-only object or `{}` |
| `Rss` | The source publishes RSS or Atom and each item acts like an event | The feed is a news/blog feed without event dates | v3 `pipeline.calendar.type = "Rss"` with `pipeline.calendar.parser.mappings` plus optional parser `filters` and top-level `validation` |
| `JsonApi` | The site exposes events through JSON responses | A better ICS or RSS feed already contains the same data | `schemaVersion = 3` with `pipeline.calendar` and `pipeline.event` |
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

!!! warning "JSON response with HTML body — use HtmlLite instead"
    Some endpoints return `Content-Type: application/json` but the payload body is a raw HTML string rather than structured event objects. WordPress `wp-json` wrappers sometimes behave this way.

    If the JSON body's event data is not parseable as structured JSON objects (for example, if the value is an embedded HTML document), treat the source as `HtmlLite` and parse the embedded HTML. Use `JsonApi` only when the response contains traversable structured event objects.

### HtmlLite

Use HtmlLite only when:
- event cards are present in View Source or the first HTML response
- titles, URLs, and date/time can be selected from DOM nodes
- you do not need JavaScript execution to reveal the event list

Do not use HtmlLite when:
- the page is only a rendered shell over a JSON endpoint
- the most reliable event data comes from XHR or fetch responses
- the page shows cards visually, but the initial HTML does not contain repeatable event nodes

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

## Generic Source Discovery Workflow

Use this sequence for any starting URL before writing a schema:

1. Capture evidence from the initial response first.
2. Check for structured feed links (`.ics`, RSS/Atom) in page HTML and metadata.
3. If no feed is obvious, inspect network activity and script-referenced endpoints for JSON payloads that contain event objects.
4. Validate that the discovered payload includes repeated event records (not only UI config or shell data).
5. Choose the highest-priority type that is truly usable from validated payload evidence.

Evidence checklist before committing to a type:

1. The candidate endpoint returns parseable content (`text/calendar`, XML feed, or JSON object/array).
2. At least one sample event contains `title` and `start`/time evidence.
3. The path from root to event collection can be described deterministically.
4. Event count is plausible for the source (not just one test row unless expected).

If any checklist item fails, continue discovery before picking a type.

### Discovery Example: Next.js Public Bundle -> JsonApi

Some event pages are Next.js shells backed by a public content API.

1. Check the initial HTML and linked scripts for `/_next/` assets or serialized app data.
2. Inspect public bundle code for fetch targets, hostnames, and required headers such as `x-api-key`.
3. Verify the discovered endpoint returns structured event JSON rather than HTML wrapped in JSON.
4. If the bundle reveals both a stable JSON endpoint and the required public API key/header, choose `JsonApi` and carry those request requirements into the schema.

This avoids unnecessary `HtmlLite` attempts against a page that only renders event cards after client-side data loading.

## Proof Before You Pick HtmlLite

Before you choose `HtmlLite`, you should be able to cite all of these from the initial HTML response:

1. one repeated event card container
2. one title sample from that container
3. one date or time sample from that container
4. one stable URL or detail link from that container

If you cannot prove those four things, keep looking for a structured source.

## Where To Go Next

- Shared payload and validation rules: [Submission API And Validation](submission-api-and-validation.md)
- `Ics` details: [Ics Source Guide](ics.md)
- `Rss` details: [Rss Source Guide](rss.md)
- `JsonApi` details: [JsonApi Source Guide](json-api.md)
- `HtmlLite` details: [HtmlLite Overview](html-lite/overview.md)