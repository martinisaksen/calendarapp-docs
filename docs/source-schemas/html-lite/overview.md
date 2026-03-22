# HtmlLite Source Submission Overview

Use HtmlLite when onboarding a new source that serves static HTML containing event cards.

If you are not sure the source should use HtmlLite, start with [Choose A Source Type](../choose-source-type.md) before building selectors.

This reference is shared by both tracks:
- human source contributor workflow
- autonomous AI source onboarding workflow

## HtmlLite Quick Reference

- `eventCardSelector`: selector for repeating event nodes
- `mappings` values: strings only
- Text extraction: `selector`
- Attribute extraction: `selector@attribute` or `selector::attr(attribute)`
- Constant value: `literal:...`
- Selector language: limited CSS-like selectors plus `xpath=`
- `detailPage.linkSelector` must resolve to a URL
- `detailPage.detailMappings` run against detail-page DOM
- Detail-page enrichment runs when both `detailPage.enabled=true` and `CALENDARAPP_HTMLLITE_DETAIL_ENRICHMENT_ENABLED` is not set to disable it
- `pagination.type` allowed values: `nextLink`, `queryIncrement`, `pathIncrement`, `fixedUrls`
- `schemaDefinition` must be sent as a JSON string in API payloads

## AI-First Workflow From A Starting URL

Use this deterministic order to avoid 0-event failures:

1. Find one stable repeating card selector.
2. Map only `title`, `url`, and a parseable `startTime` first.
3. Validate with `POST /api/source-schemas/test-fetch`.
4. Add `id` and pagination.
5. Add `detailPage` enrichment for time/location/description/image if list fields are incomplete.
6. Re-validate after each change.

When detail mappings are configured but no enrichment is visible in parsed samples, verify the runtime flag `CALENDARAPP_HTMLLITE_DETAIL_ENRICHMENT_ENABLED` is enabled in the target environment.

If list-page time is unreliable, use a temporary parseable `literal:` start time to keep cards from being dropped, then overwrite with detail-page `startDate` + `startTime` mappings.

## Human Workflow Variant

Use the same order as AI-first workflow, but validate after each manual selector change and keep notes on:
- why each selector was chosen
- what failed and how it was corrected
- final validation evidence for admin handoff

## Reliability Principles

- Parse success is necessary but not sufficient. Validate field completeness for product-facing fields.
- Prefer schema-first fixes over runtime code changes.
- Runtime/platform code changes are acceptable only when generic and reusable across all sources.
- If list cards are teaser-only, use detail-page enrichment instead of brittle list selectors.
- For API-backed sources, verify field projection is not suppressing rich fields such as long descriptions or address fields.

## End-to-End Flow

1. Confirm source compatibility with static HTML parsing.
2. Build a minimal schema.
3. Add identity, pagination, and detail-page rules.
4. Test fetch, then submit Draft and review backend validation result.
5. Handoff schema ID and validation notes for admin review.

## Endpoint Summary

- POST /api/source-schemas/test-fetch
- POST /api/source-schemas/community-submissions

## Request Contract Rule

schemaDefinition must be sent as a JSON string in request payloads.

## Submit Validation Result

The submit endpoint performs server-side fetch/parse validation and returns:
- sourceSchema: the created Draft source schema
- validation.isSuccess: pass/fail
- validation.totalEventsParsed: parsed event count
- validation.sampleEvents: sample parsed rows (up to 5)
- validation.errorMessage: failure reason when validation fails

Quick interpretation:
- Success: validation.isSuccess = true and validation.totalEventsParsed > 0
- Failure: validation.isSuccess = false and validation.errorMessage explains what to fix

See also:
- [API Workflow](api-workflow.md)
- [Validation Checklist](validation-checklist.md)
- [Troubleshooting](troubleshooting.md)
