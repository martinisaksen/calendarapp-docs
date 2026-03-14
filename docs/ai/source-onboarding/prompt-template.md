# ChatGPT Schema Generation Prompt

Use this prompt with **standard ChatGPT or any plain chat interface** to generate a schema and a ready-to-run submission command.

> If you are using an AI agent with HTTP tool use (custom GPT with API Actions, etc.), use the [Autonomous Runbook](runbook.md) instead.

---

## How it works

ChatGPT cannot call APIs on your behalf. Instead, this prompt instructs it to:

1. Browse the feed URL and inspect the HTML structure
2. Build the complete `schemaDefinition`
3. Output a copy-pasteable PowerShell command you run yourself to submit it

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
2. Construct the schemaDefinition JSON object with eventCardSelector,
   mappings, pagination (if needed), and validation.requiredFields.
3. Serialize schemaDefinition as a compact JSON string (no line breaks).

Output exactly two things and nothing else:

### 1. schemaDefinition JSON (pretty-printed for review)
<paste the full schemaDefinition object here, formatted for readability>

### 2. Ready-to-run PowerShell submission command
Produce a complete PowerShell Invoke-RestMethod command targeting
http://localhost:5047/api/source-schemas that submits all fields
(name, description, type, feedUrl, schemaDefinition, metadata).
schemaDefinition must be the compact single-line JSON string.
The command must be copy-pasteable with no placeholders remaining.
```

---

## After running the command

The response will include a `validation` object. Check:

- `validation.isSuccess = true` and `validation.totalEventsParsed > 0` → ready for admin handoff
- `validation.isSuccess = false` → copy `validation.errorMessage` back into ChatGPT and ask it to fix the schema, then resubmit


