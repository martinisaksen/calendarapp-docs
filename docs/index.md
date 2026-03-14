# CalendarApp Documentation

Welcome to CalendarApp public documentation.

This site is organized for two audiences:
- **Human source contributors** onboarding new event sources
- **AI agents** performing automated source onboarding

> **Scope:** This documentation covers source onboarding and submission workflows only. It is not a code contribution guide for CalendarApp application repositories.

---

## User Guides

### HtmlLite Source Submission

| Page | Description |
|---|---|
| [Overview](source-schemas/html-lite/overview.md) | What HtmlLite is and when to use it |
| [API Workflow](source-schemas/html-lite/api-workflow.md) | Submit a source and interpret the validation response |
| [Schema Basics](source-schemas/html-lite/schema-basics.md) | Schema structure and required fields |
| [Selectors](source-schemas/html-lite/selectors.md) | CSS and XPath selector syntax |
| [Event Identity](source-schemas/html-lite/event-identity.md) | How event IDs are derived |
| [Time Handling](source-schemas/html-lite/time-handling.md) | Timezone and date format rules |
| [Pagination](source-schemas/html-lite/pagination.md) | Multi-page source configuration |
| [Detail Pages](source-schemas/html-lite/detail-pages.md) | Fetching event detail from linked pages |
| [Validation Checklist](source-schemas/html-lite/validation-checklist.md) | Pre-submission quality gates |
| [Troubleshooting](source-schemas/html-lite/troubleshooting.md) | Common errors and fixes |

---

## AI Guides

### Autonomous Source Onboarding

| Page | Description |
|---|---|
| [Autonomous Runbook](ai/source-onboarding/runbook.md) | Step-by-step workflow for AI agents submitting sources |
| [Prompt Template](ai/source-onboarding/prompt-template.md) | Reusable prompt for sourcing and submitting via the API |
