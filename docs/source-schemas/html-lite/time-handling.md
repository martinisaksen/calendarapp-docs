# Time Handling

Time parsing is a common failure point.

## Preferred Extraction Order

1. time datetime attribute
2. schema.org or meta startDate
3. data-start or similar data attributes
4. visible text parsing

Examples:
- Preferred: xpath=.//time[@datetime]::attr(datetime)
- Riskier: .event-time
- Risky: visible text like 10:30 am - 11:30 am

## Split Date And Time Support

When the page separates date and time into different nodes, map:

- `startDate` to the visible date node
- `startTime` to either a time-only value or a visible range like `10:30 AM - 11:30 AM`
- optionally `endDate` and `endTime` if the end date is exposed separately

Example:

```json
{
	"mappings": {
		"title": "h2 a",
		"startDate": ".event__date",
		"startTime": ".event__time",
		"url": "h2 a@href"
	}
}
```

Behavior:

- If `startTime` is already a full datetime, HtmlLite uses it directly.
- If `startDate` is present and `startTime` is time-only, HtmlLite combines them.
- If `startTime` is a visible range like `10:30 AM - 11:30 AM`, HtmlLite derives both `StartTime` and `EndTime`.
- If no end can be derived, `EndTime` still defaults to `StartTime + 1 hour`.

## Validation Tips

- verify submit response validation.sampleEvents have parseable datetime values
- verify timezone assumptions
- verify AM/PM and all-day handling

## Date Range Parsing & Silent Failures

⚠️ **Important:** Unstructured date ranges are still not supported by default single-date parsing. Use `multiDateExpansion` for deterministic list-style multi-date strings.

### Examples of What Fails

**Fails (without `multiDateExpansion`):**
- "May 8 - 24, 2026" ← Date range
- "June 6 & 7, 2026" ← Multiple dates with ampersand
- "June 27 - July 19, 2026" ← Cross-month range
- "TBD" or "Call for dates" ← Unparseable text
- "Ongoing" ← Non-datetime text

**Succeeds (single dates parse cleanly):**
- "April 18, 2026" ✓
- "05/30/2026" ✓
- "2026-12-13" ✓
- "December 13, 2025" ✓

### Debug Strategy When Parse Count is Low

1. Run test-fetch and check `validation.totalEventsParsed` vs. expected count on source page
2. If discrepancy exists, compare `sampleEvents` against the source
3. **Common cause:** Date format uses ranges or special text that cannot parse
4. Manual verification: Does each visible event have a single, standard date format?

### Workarounds

**Option 1: Expand one card into multiple occurrences (recommended)**

Use `multiDateExpansion` in HtmlLite parser config:

```json
"multiDateExpansion": {
  "enabled": true,
  "sourceField": "startTime",
  "defaultTime": "7:30 PM",
  "defaultTimeZone": "America/Los_Angeles",
  "fallbackMode": "legacySingle",
  "limits": {
    "maxOccurrencesPerCard": 12,
    "maxExpandedEventsPerRun": 500,
    "maxParseFailuresPerCard": 3
  }
}
```

This enables one-card-to-many-events fan-out for supported multi-date list formats.

**Option 2: Extract only the first date from a range**
```json
"fieldTransforms": {
  "startTime": [
    { "type": "ReplaceLiteral", "value": " - ", "replacement": " RANGE_SEP " },
    { "type": "ReplaceLiteral", "value": " RANGE_SEP ", "replacement": "|" }
  ]
}
```
Then use regex or post-processing to extract text before the pipe.

**Option 3: Use detail page enrichment**
If list pages show date ranges but detail pages have specific start/end times:
- Set list `startTime` to a `literal:` placeholder
- Enable `detailPage.enabled: true`
- Map actual dates in `detailPage.detailMappings`

**Option 4: Create separate sources**
For venue+date combinations, create distinct sources:
- Source 1: Events happening June 6
- Source 2: Events happening June 7

(Use filters or pagination to isolate by date)

## Normalizing a.m./p.m. Abbreviations

Standard date parsing does not recognize `a.m.` or `p.m.` — it requires `AM`/`PM`. If the source uses dotted abbreviations, normalize them with `ReplaceLiteral` transforms before any date parsing occurs.

Also strip connector words such as `at` that appear between the date and time string.

```json
"fieldTransforms": {
  "startTime": [
    { "type": "ReplaceLiteral", "value": " at ", "replacement": " " },
    { "type": "ReplaceLiteral", "value": "p.m.", "replacement": "PM" },
    { "type": "ReplaceLiteral", "value": "a.m.", "replacement": "AM" },
    { "type": "CollapseWhitespace" }
  ]
}
```

## Multi-Date Strings — First Date vs Expansion

Some performing arts sites encode multiple performance dates in a single string, for example:

> `Saturday, April 11, 2026 at 7:00 p.m. & Sunday, April 12, 2026 at 3:00 p.m.`

There are two supported patterns:

1. **Preferred:** use `multiDateExpansion` to create one event per occurrence.
2. **Fallback:** keep first-date-only transforms when you intentionally want one representative occurrence.

First-date-only transform example:

```json
"fieldTransforms": {
  "startTime": [
    { "type": "RegexReplace", "value": " & .*$", "replacement": "" },
    { "type": "ReplaceLiteral", "value": " at ", "replacement": " " },
    { "type": "ReplaceLiteral", "value": "p.m.", "replacement": "PM" },
    { "type": "ReplaceLiteral", "value": "a.m.", "replacement": "AM" },
    { "type": "CollapseWhitespace" }
  ]
}
```

> **Note:** This chain captures only the first performance date. For one-occurrence-per-performance output inside HtmlLite, prefer `multiDateExpansion`.

## Composition With Detail Occurrence Matching

When both `multiDateExpansion` and `detailPage.occurrenceSelector` are configured:

1. `multiDateExpansion` determines event count
2. `detailPage.occurrenceSelector` refines the matching expanded event
3. Detail occurrence matching does not fan out additional events

This precedence keeps behavior deterministic and avoids duplicate ownership of fan-out logic.

## Default Behavior

- EndTime defaults to StartTime + 1 hour when no endTime mapping is provided
- IsAllDay defaults to false
