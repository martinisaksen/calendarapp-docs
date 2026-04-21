# Wheneber Docs

Public documentation for onboarding new calendar sources into Wheneber.

## Scope

This repository documents source contribution workflows only:
- schema-first onboarding for external event sources
- source type selection for Ics, Rss, JsonApi, and HtmlLite
- payload design, validation, and contributor-safe submission patterns
- autonomous AI onboarding runbooks and prompt templates

This repository does not document code contribution for private Wheneber application repositories.

## Audience Tracks

- Human source contributors: start with source type selection and the shared submission workflow docs.
- AI agents: use the autonomous runbook and deterministic output contract.

## Start Points

- Site landing page: `docs/index.md`
- Human onboarding workflow: `docs/source-schemas/choose-source-type.md`
- Shared submission workflow: `docs/source-schemas/submission-api-and-validation.md`
- AI autonomous workflow: `docs/ai/source-onboarding/runbook.md`
- Chat-only fallback workflow: `docs/ai/source-onboarding/prompt-template.md`
