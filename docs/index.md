# CalendarApp Source Onboarding Docs

This site is the public source-of-truth for onboarding event sources into CalendarApp.

It is intentionally split into two execution tracks:
- Human source contributors
- Autonomous AI agents

> Scope boundary: these docs cover source onboarding and source schema submission only. They do not cover code contribution to private CalendarApp application repositories.

## Choose A Track

### Human Source Contributors

Start here:
1. [HtmlLite Overview](source-schemas/html-lite/overview.md)
2. [HtmlLite API Workflow](source-schemas/html-lite/api-workflow.md)
3. [Validation Checklist](source-schemas/html-lite/validation-checklist.md)
4. [Troubleshooting](source-schemas/html-lite/troubleshooting.md)

Use this track when a person is inspecting pages, refining selectors, and submitting drafts.

### Autonomous AI Agents

Start here:
1. [Autonomous Runbook](ai/source-onboarding/runbook.md)
2. [HtmlLite Overview](source-schemas/html-lite/overview.md)
3. [HtmlLite API Workflow](source-schemas/html-lite/api-workflow.md)
4. [ChatGPT Prompt (No HTTP tool use)](ai/source-onboarding/prompt-template.md)

Use this track when an agent can browse pages and execute API calls end-to-end, or when a chat model must generate a human-runnable submission script.

## Source Onboarding Lifecycle

1. Discover a starting URL and verify static-HTML viability.
2. Build a minimal schema (`eventCardSelector`, `title`, parseable `startTime`).
3. Submit Draft and inspect validation (`isSuccess`, `totalEventsParsed`, `sampleEvents`).
4. Add identity, pagination, and detail enrichment.
5. Re-submit until validation is stable and non-zero.
6. Hand off schema ID and validation notes for admin review.

## Core Reference Pages

| Area | Page | Purpose |
|---|---|---|
| Reliability | [Validation Checklist](source-schemas/html-lite/validation-checklist.md) | Field-completeness and handoff readiness gates |
| Reliability | [Troubleshooting](source-schemas/html-lite/troubleshooting.md) | Recovery order for zero-parse and validation failures |
| HtmlLite | [Overview](source-schemas/html-lite/overview.md) | Contract summary and deterministic build order |
| HtmlLite | [API Workflow](source-schemas/html-lite/api-workflow.md) | Draft submission behavior and response interpretation |
| HtmlLite | [Schema Basics](source-schemas/html-lite/schema-basics.md) | Minimal schema and progressive enrichment pattern |
| HtmlLite | [Selectors](source-schemas/html-lite/selectors.md) | CSS-like and XPath usage rules |
| HtmlLite | [Event Identity](source-schemas/html-lite/event-identity.md) | Stable ID strategy and fallback behavior |
| HtmlLite | [Time Handling](source-schemas/html-lite/time-handling.md) | Date/time parsing and timezone assumptions |
| HtmlLite | [Pagination](source-schemas/html-lite/pagination.md) | Multi-page extraction strategies |
| HtmlLite | [Detail Pages](source-schemas/html-lite/detail-pages.md) | Detail-page enrichment and merge rules |
| HtmlLite | [Validation Checklist](source-schemas/html-lite/validation-checklist.md) | Quality gates before admin handoff |
| HtmlLite | [Troubleshooting](source-schemas/html-lite/troubleshooting.md) | Fast diagnosis for zero-parse and validation failures |
