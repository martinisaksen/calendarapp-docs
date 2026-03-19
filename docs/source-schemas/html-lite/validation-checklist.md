# Validation Checklist

## Quality Gates Before Submission

Require all of the following:
- submit response validation.isSuccess is true
- submit response validation.totalEventsParsed is non-zero and reasonable
- submit response validation.sampleEvents title, startTime, and url values are valid
- event IDs are stable and not title-derived
- time values are parseable and timezone assumptions are sane
- pagination is configured when multiple pages exist
- detail enrichment is enabled when list pages are incomplete

Field completeness gate:
- compare parsed sample events against at least 3 known detail URLs
- ensure expected detail fields are present when the source page provides them

Recommended additional checks:
- sample events include expected city/region context
- URLs resolve to canonical event pages (not list-only links)
- detailPage.linkSelector resolves within the card scope

## Submission Checklist

- compatibility check completed
- schema submitted as Draft
- schema ID and validation notes handed off to admins

For autonomous execution, also require:
- final machine-readable report emitted
- blockers and nextActions populated when status is Failed
- retry attempts are bounded and documented

## Schema Quality Levels

| Level | Expected Fields |
|---|---|
| Basic | title + startTime |
| Standard | title + startTime + url + location |
| Rich | title + startTime + endTime + url + location + description |
| Premium | Rich + imageUrl + venueName + venueAddress |

## Minimum Acceptance Criteria

Approve handoff readiness only when:
1. `validation.isSuccess` is true
2. `validation.totalEventsParsed` is greater than zero
3. sampled events look semantically correct for title/time/url
4. ID and time handling rules are stable across at least one pagination step (if paging exists)

## Field Completeness Matrix (Recommended)

Use this quick matrix against known detail URLs before handoff:

| Field | Expected On Source | Present In Parsed Sample |
|---|---|---|
| title | yes/no | yes/no |
| startTime | yes/no | yes/no |
| endTime | yes/no | yes/no |
| description | yes/no | yes/no |
| location | yes/no | yes/no |
| venueName | yes/no | yes/no |
| venueAddress | yes/no | yes/no |
| imageUrl | yes/no | yes/no |
| url | yes/no | yes/no |

If parse count is non-zero but expected fields are still missing, treat as a quality failure and continue refinement.
