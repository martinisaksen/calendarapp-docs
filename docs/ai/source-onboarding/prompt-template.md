# ChatGPT Schema Generation Prompt

Copy the template below, replace `<FEED_URL>` with your calendar feed URL, and paste into ChatGPT. The AI can use it to read the schema documentation, generate a valid payload, and, in tool-enabled environments, test and submit it.

In a tool-enabled AI session, the model can use this prompt to read the schema documentation, generate a valid payload, and, if it has HTTP access to the correct endpoint, test and submit it. In text-only sessions, use the same prompt to generate the payload and PowerShell commands, then run them yourself.

> **For AI agents with HTTP tool use** (custom GPTs with API actions, etc.): use the [Autonomous Runbook](runbook.md) instead.

---

## Prompt Template

Just copy, change the URL, and paste:

```
Generate a calendar schema for this feed:
Feed URL: <FEED_URL>

REQUIRED PROCESS — NO SHORTCUTS

Step 1 — Retrieve Documentation

You MUST fetch and read all of these before doing anything else:
- https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/llms.txt
- From that index, fetch the raw URL for: choose-source-type.md
- From that index, fetch the raw URL for: submission-api-and-validation.md

If you cannot access any of these documents, STOP immediately and return:
ERROR: Unable to retrieve required documentation
DO NOT continue.

Step 2 — Doc Evidence (MANDATORY)

Print a section called "Doc Evidence" containing direct verbatim quotes (not summaries) from the docs you fetched for:
- Source type selection rules
- ICS-specific rules (if the feed is ICS)
- RSS-specific default validation and filtering rules (if the feed is RSS)
- Submission constraints (schemaDefinition format, metadata field restrictions)

Step 3 — Retrieve Schema Contract

Fetch and read:
https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/submission-request.schema.json

Then fetch the type-specific contract for the selected source type:
- Ics: https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/ics.schema.json
- Rss: https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/rss.schema.json
- JsonApi: https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/v3/source-schema-v3.schema.json
- HtmlLite: https://raw.githubusercontent.com/martinisaksen/calendarapp-docs/main/docs/source-schemas/html-lite.schema.json

List all constraints that apply to: schemaDefinition, metadata, and required fields. Validate both the outer submission payload and the inner type-specific schemaDefinition contract.

Step 4 — Determine Source Type

Using ONLY the rules quoted in Step 2:
- State the chosen type (Ics, Rss, JsonApi, or HtmlLite)
- Quote the exact rule from the docs that applies
- Explain why the feed matches it

Step 5 — Build Schema

Build the complete JSON payload.
It must validate against the submission request schema and follow all constraints quoted in Step 2.

If the selected type is `Rss`, the payload MUST also follow these generation defaults unless the docs explicitly justify a tighter variant:
- include `pipeline.calendar.parser.mappings.title`
- include parseable `pipeline.calendar.parser.mappings.startTime`
- include `validation.minEventsPerFetch = 1`
- include `validation.requiredFields = ["title", "startTime"]`
- add `pipeline.calendar.parser.mappings.id` when the feed exposes stable identity (often mapped from `guid`)
- add `pipeline.calendar.parser.mappings.eventUrl` when the feed exposes detail pages (often mapped from `link` or `url`)
- prefer source-side filtering first (event-only feed URL or category-specific feed URL)
- when source-side filtering is insufficient, use supported schema filtering with `pipeline.calendar.parser.filters.includeAny` / `excludeAny`
- if the feed mixes non-event items and the source offers no narrower event-only variant, stop and explain that the feed is not suitable for RSS onboarding as-is

Step 6 — Validation Gate (MANDATORY before output)

Explicitly confirm:
- schemaDefinition is a JSON STRING (not an object)
- metadata contains ONLY allowed fields: location, region, category, language, contactEmail, or estimatedEventCount
- For Ics: schemaDefinition is {} or validation-only (no mappings, eventMapping, transforms, sourceType, url)
- For Rss: validation includes `minEventsPerFetch = 1` and `requiredFields = ["title", "startTime"]` unless the docs support a stricter field gate
- For Rss: any filters are under `pipeline.calendar.parser.filters` and only use supported fields (`title`, `description`, `eventUrl`, `category`)

Step 7 — Test + Submit

If you have HTTP tool access and can reach the correct API endpoint for the current environment, POST to the test-fetch endpoint first, then submit the exact same payload to the submission endpoint if successful.

Environment note:
- Local development commonly uses http://localhost:5047/api/source-schemas
- Hosted or production contributors may need a different documented endpoint

If you do not have HTTP tool access or cannot reach the endpoint, do not pretend to test or submit. Instead, output the payload and PowerShell commands for the operator to run.

OUTPUT FORMAT (in this order):
1. Source Type
2. Doc Evidence — verbatim quoted excerpts from fetched docs
3. Constraint Checklist — each item from Step 6 confirmed
4. Final JSON Payload
5. PowerShell commands for test-fetch and community-submissions
6. Test Result: actual result if HTTP execution was possible, otherwise "Not executed in this chat session"
```

---

## How It Works

When you paste this template with your URL:

1. **AI fetches the index** — `llms.txt` provides raw GitHub URLs to the docs (plain text, no JavaScript required)
2. **AI quotes the rules** — The "Doc Evidence" section adds a visible retrieval check and makes it easier to audit which documentation the model used
3. **AI loads the matching contract** — After choosing the source type, it reads the matching type-specific `schemaDefinition` schema as well as the outer submission schema
4. **AI inspects your feed** — It determines whether the source is Ics, Rss, JsonApi, or HtmlLite using the documented rules
5. **AI validates the schema** — It runs through the constraint checklist before producing output
6. **AI tests and submits when possible** — In tool-enabled sessions with reachable endpoints, it can call the API; otherwise it outputs the payload and commands for a human to run
7. **AI prints results** — Schema ID, event count, and any errors when execution was possible

The output is ready to review and run. In non-executing chat sessions, use the generated PowerShell commands yourself.

---

## Quick Reference: What Goes in the Prompt

| Field | What to Enter |
|-------|---------------|
| `<FEED_URL>` | The URL of the calendar feed (ICS file, RSS feed, JSON endpoint, or HTML page) |

That's it. The prompt guides the rest of the workflow.

---

## Troubleshooting

**AI skips the Doc Evidence section:**
- Paste the prompt again and add: "You must include verbatim quotes from the docs in the Doc Evidence section before generating the schema."

**AI returns `ERROR: Unable to retrieve required documentation`:**
- Check that you have internet access in the chat session
- If using a custom GPT or restricted environment, make sure browsing/fetch tools are enabled

**AI says it tested or submitted, but your environment does not support HTTP tools or localhost access:**
- Treat that as a bad response
- Ask it to regenerate the answer with the payload plus PowerShell commands only, and to mark the test result as not executed

**AI generates unsupported fields:**
- Copy the error message from test-fetch back into the chat
- Ask the AI: "Fix this error in the schema and resubmit"

**Test returns 0 events:**
- Share the error message with the AI
- Ask: "What should we debug? Should we try a different source type?"

---

## After Submission

The AI will print:
- `sourceSchema.id` — The ID of your submitted schema (save this for reference)
- `validation.isSuccess: true` — Schema passed validation
- `validation.totalEventsParsed` — Number of events extracted from the feed

If `isSuccess: false`, an error message explains what to fix. Copy it back to the AI and ask for a corrected schema.

---

## Tips for Success

- **Keep the URL in the prompt.** Change only the feed URL; leave everything else.
- **If the AI skips the Doc Evidence section,** stop it and ask it to restart from Step 1.
- **If the AI cannot execute HTTP requests,** it should still generate the payload and commands, but it should not claim to have tested or submitted anything.
- **If the AI breaks the rules,** share the error message and ask it to fix the schema.
- **If it generates 0 events,** ask the AI what debugging step it recommends (source type, selector, timezone, etc.).
- **Save the schema ID** after successful submission — you'll need it if you want to update the schema later.
- **Test with a simple source first** (like a public ICS calendar) to verify your setup before trying complex JSON or HTML sources.


