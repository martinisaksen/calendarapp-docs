# ChatGPT Schema Generation Prompt

Use this prompt with **standard ChatGPT or any plain chat interface** to choose the right source type, generate a schema, and output a ready-to-run submission command.

> If you are using an AI agent with HTTP tool use (custom GPT with API Actions, etc.), use the [Autonomous Runbook](runbook.md) instead.

---

## How it works

ChatGPT cannot call APIs on your behalf. Instead, this prompt instructs it to:

1. Browse the feed URL and determine whether the best source type is `Ics`, `Rss`, `JsonApi`, or `HtmlLite`
2. Build the complete `schemaDefinition`
3. Output copy-pasteable PowerShell commands for test-fetch and community submission

The generated output is successful only if it is deterministic and directly executable with no placeholders.

---

## Prompt

Copy the block below, fill in the inputs, and paste it into ChatGPT.

---

```
Before answering, read the docs under **Choose A Source Type** and **Submission API And Validation** at https://martinisaksen.github.io/calendarapp-docs/.

Build a complete contributor-safe SourceSchema submission for the following source.

Inputs:
- sourceName: <SOURCE_NAME>
- feedUrl: <FEED_URL>
- expectedLocation: <LOCATION> (optional — infer from site if omitted)
- expectedCategory: <CATEGORY> (optional — infer from site if omitted)
- expectedRegion: <REGION> (optional — infer from site if omitted)
- expectedLanguage: <LANGUAGE> (optional — default "en" if omitted)

Instructions:
1. Browse <FEED_URL> and choose the most stable source type in this order: `Ics`, `Rss`, `JsonApi`, `HtmlLite`.
2. Prove the chosen source type with evidence from the page, feed, or network calls.
3. Explain the chosen source type in one short sentence.
4. Build the smallest valid schema for that type, then enrich it only if needed.
5. Construct the final schemaDefinition JSON object and serialize as a compact JSON string (single line).
6. Use only values that can be inferred from feed or page content. Do not invent fields not evidenced by the source.
7. If JsonApi parsing returns zero events, debug in this order: eventArrayPath scope, mapping scope, required headers, then pagination/workflow.
8. Change one variable per retry and explain what changed.

Schema contract (must follow exactly):
- `schemaDefinition` must be valid JSON and will be sent as a compact JSON string.
- For `Ics`, use a validation-only schema unless stricter checks are needed.
- For `Rss`, use `extractionRules` and map at least `title` and `startTime`.
- For `JsonApi`, include `eventArrayPath` and `mappings`; use `schemaVersion` 1 or 2 when needed.
- For `JsonApi`, advanced features require `schemaVersion = 2`.
- For `JsonApi`, do not configure both `requestWorkflow` and `pagination`.
- If a usable event JSON endpoint exists, choose `JsonApi`, not `HtmlLite`.
- For `HtmlLite`, mappings values must be strings, not objects.
- For `HtmlLite`, do not use comma-separated selector lists. Rewrite them as `xpath=` if needed.
- For `HtmlLite`, prefer simple CSS-like selectors for tag/class/id and descendant-by-space patterns such as `h2 a`.
- For `HtmlLite`, use `xpath=` when you need unsupported CSS features like `>`, comma-separated selector lists, sibling logic, or positional filters.
- For `HtmlLite`, attribute extraction uses `selector@attribute` or `selector::attr(attribute)`.
- For `HtmlLite`, do not use `regex:` mapping strings; they are unsupported.
- For `HtmlLite`, if detail page extraction is used, use detailPage.linkSelector and detailPage.detailMappings with string selectors only.
- For `HtmlLite`, allowed pagination.type values are `nextLink`, `queryIncrement`, `pathIncrement`, `fixedUrls`.
- Do not include unsupported fields such as `sourceName`, `baseUrl`, or `transforms` inside `schemaDefinition`.
- Do not use placeholder `literal:` time values in final handoff output unless no real time can be extracted from list or detail pages.

Output exactly three things and nothing else:

### 1. Chosen source type
Print only one of: `Ics`, `Rss`, `JsonApi`, `HtmlLite`.

### 2. Evidence and schemaDefinition JSON (pretty-printed for review)
Start with a short evidence block containing:
- chosen type reason
- discovered endpoint or repeated card selector
- for HtmlLite: one quoted example title, one time sample, and one link sample from the initial HTML

Then print only valid JSON for schemaDefinition.

### 3. Ready-to-run PowerShell test and submission commands
Produce a complete PowerShell script targeting
http://localhost:5047/api/source-schemas that first calls `/test-fetch` and then `/community-submissions` with all fields
(name, description, type, feedUrl, schemaDefinition, metadata).
schemaDefinition must be the compact single-line JSON string.
The command must be copy-pasteable with no placeholders remaining.

The script must:
- assign schemaDefinition from the pretty JSON via `ConvertTo-Json -Compress`
- call test-fetch with `Invoke-RestMethod`
- submit to `community-submissions` with `Invoke-RestMethod`
- print schema ID
- print validation.isSuccess
- print validation.totalEventsParsed
- print validation.errorMessage when failed

After the Invoke-RestMethod call, the script must also print the validation result:
- "$($resp.validation.isSuccess)" — true or false
- "$($resp.validation.totalEventsParsed) events parsed"
- If validation.isSuccess is false: "$($resp.validation.errorMessage)"

Before finalizing output, self-check:
- schema matches the chosen source type
- a higher-priority source type was not incorrectly skipped
- all required fields for that source type are present
- event path points to repeated event records, not wrapper/config nodes
- requiredFields are present in mapped output
- HtmlLite selectors avoid unsupported CSS operators unless rewritten with `xpath=`
- HtmlLite selectors do not use commas
- requiredFields align with mapped fields
- linkSelector returns a URL when detailPage is used
- pagination.type is valid when used
- schemaDefinition does not contain unsupported wrapper fields
- schemaDefinition compact string is single-line
```

---

## After running the command

The script will print the validation result directly. Check:

- `True` + event count → ready for admin handoff (note the `sourceSchema.id` printed by the script)
- `False` + error message → copy the error message back into ChatGPT and ask it to fix the schema, then resubmit

If you want to inspect the full response, run `$resp | ConvertTo-Json -Depth 8` after the script completes.


