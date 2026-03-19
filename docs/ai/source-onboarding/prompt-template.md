# ChatGPT Schema Generation Prompt

Use this prompt with **standard ChatGPT or any plain chat interface** to generate a schema and a ready-to-run submission command.

> If you are using an AI agent with HTTP tool use (custom GPT with API Actions, etc.), use the [Autonomous Runbook](runbook.md) instead.

---

## How it works

ChatGPT cannot call APIs on your behalf. Instead, this prompt instructs it to:

1. Browse the feed URL and inspect the HTML structure
2. Build the complete `schemaDefinition`
3. Output a copy-pasteable PowerShell command you run yourself to submit it

The generated output is successful only if it is deterministic and directly executable with no placeholders.

---

## Prompt

Copy the block below, fill in the inputs, and paste it into ChatGPT.

---

```
Before answering, read the docs under **HtmlLite Source Submission** at https://martinisaksen.github.io/calendarapp-docs/.

Build a complete HtmlLite SourceSchema submission for the following source.

Inputs:
- sourceName: <SOURCE_NAME>
- feedUrl: <FEED_URL>
- expectedLocation: <LOCATION> (optional — infer from site if omitted)
- expectedCategory: <CATEGORY> (optional — infer from site if omitted)
- expectedRegion: <REGION> (optional — infer from site if omitted)
- expectedLanguage: <LANGUAGE> (optional — default "en" if omitted)

Instructions:
1. Browse <FEED_URL> and identify: event card selector, field mappings
   (title, startTime, endTime, location, url, id), and pagination mode.
2. Build schema in phases:
   - Phase A (bootstrap): `eventCardSelector` + minimal mappings (`title`, `url`, parseable `startTime`) + validation.requiredFields.
   - Phase B: add `id`, pagination, and detailPage mappings.
3. If list-page time is unreliable, use `literal:` for bootstrap `startTime` and extract real `startDate` + `startTime` from detail pages.
4. Construct the final schemaDefinition JSON object and serialize as a compact JSON string (single line).
5. Use only values that can be inferred from page content. Do not invent fields not evidenced by the page.

Schema contract (must follow exactly):
- mappings values must be strings, not objects.
- HtmlLite supports simple CSS-like selectors and XPath.
- Prefer simple CSS-like selectors for tag/class/id and descendant-by-space patterns such as `h2 a`.
- Use `xpath=` when you need unsupported CSS features like `>`, comma-separated selector lists, sibling logic, or positional filters.
- For text or datetime extraction: use "selector".
- For attribute extraction: use "selector@attribute".
- Do not output mapping objects like {"selector":"...","type":"...","attribute":"..."}.
- Do not use unsupported CSS operators such as `>` or `.a, .b` unless you rewrite that selector as `xpath=`.
- If the page splits date and time into separate nodes, map `startDate` and `startTime`. `startTime` may be a time-only value or a visible range like `10:30 AM - 11:30 AM`.
- If detail page extraction is used, use detailPage.linkSelector and detailPage.detailMappings with string selectors only.
- `detailPage.enabled` must be explicitly set to true when detail enrichment is required.
- `detailPage.linkSelector` must resolve to a URL from the list-card scope.
- Pagination uses `pagination.type` (not `pagination.mode`).
- Allowed pagination.type values: "nextLink", "queryIncrement", "pathIncrement", "fixedUrls".
- Do not use "query" as a pagination type. Use "queryIncrement" with parameter/start/step.

Output exactly two things and nothing else:

### 1. schemaDefinition JSON (pretty-printed for review)
Print only valid JSON for schemaDefinition.

### 2. Ready-to-run PowerShell submission command
Produce a complete PowerShell script targeting
http://localhost:5047/api/source-schemas that submits all fields
(name, description, type, feedUrl, schemaDefinition, metadata).
schemaDefinition must be the compact single-line JSON string.
The command must be copy-pasteable with no placeholders remaining.

The script must:
- assign schemaDefinition from the pretty JSON via `ConvertTo-Json -Compress`
- submit with `Invoke-RestMethod`
- print schema ID
- print validation.isSuccess
- print validation.totalEventsParsed
- print validation.errorMessage when failed

After the Invoke-RestMethod call, the script must also print the validation result:
- "$($resp.validation.isSuccess)" — true or false
- "$($resp.validation.totalEventsParsed) events parsed"
- If validation.isSuccess is false: "$($resp.validation.errorMessage)"

Before finalizing output, self-check:
- all mappings are strings
- selectors avoid unsupported CSS operators unless rewritten with `xpath=`
- requiredFields align with mapped fields
- linkSelector returns a URL
- pagination.type is valid
- schemaDefinition compact string is single-line
```

---

## After running the command

The script will print the validation result directly. Check:

- `True` + event count → ready for admin handoff (note the `sourceSchema.id` printed by the script)
- `False` + error message → copy the error message back into ChatGPT and ask it to fix the schema, then resubmit

If you want to inspect the full response, run `$resp | ConvertTo-Json -Depth 8` after the script completes.


