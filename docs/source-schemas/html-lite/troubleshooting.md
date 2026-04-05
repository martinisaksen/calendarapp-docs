# Troubleshooting

## Invalid request payload

Usually means schemaDefinition was sent as an object instead of a JSON string.

Fix:
- serialize schemaDefinition as a compact JSON string before submission
- verify request body uses `schemaDefinition: "{...}"` rather than nested object

## schemaDefinition must be v3 with schemaVersion=3

This means the payload is using legacy flat schema format.

Fix:

- add `schemaVersion: 3`
- wrap parser config under `pipeline.calendar.parser`
- add `pipeline.event`
- follow [v3 migration guide](../v3/migration-guide.md)

## phase 2 scaffolding currently supports only pipeline.event.input.field = eventUrl

This means event-stage input is using a field name other than `eventUrl`.

Fix:

- map `eventUrl` in calendar mappings
- use `"field": "eventUrl"` in `pipeline.event.input`
- see [v3 event input constraints](../v3/event-input-constraints.md)

## pipeline.calendar.type must be "Html" for HtmlLite sources

A common schema validation failure occurs when contributors set `pipeline.calendar.type` to `"HtmlLite"` inside the `schemaDefinition`. The runtime rejects this value.

The correct distinction:

| Field | Correct value |
|---|---|
| Top-level source `type` (submission envelope) | `"HtmlLite"` |
| `pipeline.calendar.type` (inside `schemaDefinition`) | `"Html"` |

Fix: in your `schemaDefinition`, set `pipeline.calendar.type` to `"Html"`. The outer submission field `type` stays `"HtmlLite"`.

See the [v3 Pipeline Examples](../v3/pipeline-examples.md) for a minimum-shape reference.

## JSON endpoint returns HTML string instead of event objects

Some endpoints return `Content-Type: application/json` but the payload body is an HTML string rather than structured event objects. This is common with WordPress `wp-json` wrappers where the body field contains raw HTML markup.

Decision rule:

- If the JSON response contains **structured event objects** traversable with a path like `$.events[*]`, use `JsonApi`.
- If the JSON response contains an **HTML string** (event data is embedded HTML, not a JSON object array), treat the source as `HtmlLite` and parse the embedded HTML instead.

How to check: inspect the raw response body. If event records are HTML markup rather than JSON key/value objects, choose `HtmlLite`.

See [Choose A Source Type](../choose-source-type.md) for the full source type decision table.

## validation.totalEventsParsed = 0

**This is the #1 source creation issue.** Events silently drop when required fields fail to parse.

### Diagnostic Checklist (in order)

1. **Verify eventCardSelector matches DOM nodes**
   - Test in browser console: `document.querySelectorAll('YOUR_SELECTOR')`
   - If zero results, selector is wrong
   - **Common error:** CSS attribute selector `[id^="prefix"]` doesn't work in XPath
   - **Fix:** Use XPath: `.//*[starts-with(@id, "prefix")]`

2. **Check title mapping returns values**
   - Verify mapping selector works on one card element
   - If title is deeply nested, use `//` for any-depth matching instead of `/`

3. **Check if startTime parses**
   - Look at `sampleEvents` startTime values
   - Common failure: Date ranges like "May 8 - 24, 2026" cause silent drops
   - See [time-handling.md](time-handling.md#date-range-parsing--silent-failures) for date range solutions

4. **Check validation.requiredFields**
   - Ensure all required fields are actually mapped in `mappings`
   - Remove fields from `requiredFields` if source doesn't provide them

### Debug Workflow

1. Run `test-fetch` and compare `sampleEvents` output with visible events on page
2. For each missing field, verify selector in browser
3. Update selector and re-test until `totalEventsParsed` matches expected count
4. Common XPath patterns:
   - Any element with attribute starting with value: `.//*[starts-with(@id, "prefix")]`
   - Element with exact attribute: `.//div[@id="exact-id"]`
   - Any descendant: `.//span` or `.//*[contains(@class, "text")]`
   - Get attribute value: Always ends with `/@attributeName`

## validation.isSuccess = false

- inspect validation.errorMessage in submit response
- correct schemaDefinition and resubmit Draft

Common recovery loop:
1. fix one failure class at a time (selector, time parsing, pagination, detail link)
2. re-submit same Draft feedUrl
3. compare sampleEvents output to previous run
4. stop when parse count and sample quality are stable

If blocked by platform visibility limitations:
- prefer generic platform improvements (for example, richer sample event output) over source-specific runtime branches

## Expression must evaluate to a node-set

XPath is returning a scalar where a node query is required. Move attribute access into ::attr(...) after selecting a node.

If using HtmlLite custom attribute syntax, prefer `selector@attribute` or `selector::attr(attribute)`.

## Approved source cannot be edited via submit

If submit returns a 400 indicating source is already approved:
- do not continue retrying submit for that feedUrl
- hand off to admins for approved-source change process
