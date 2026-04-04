# v3 Event Input Constraints

This page documents current event-stage input constraints for contributor-facing v3 schemas.

## Input modes

Supported values for `pipeline.event.input.mode`:

- `none`
- `calendarFieldUrl`
- `fixedUrl`

## Current phase-2 constraint

For event types `Html` and `Json`, current phase-2 scaffolding accepts only:

```json
"input": {
  "mode": "calendarFieldUrl",
  "field": "eventUrl",
  "baseUrl": "https://example.org"
}
```

If `field` is not `eventUrl`, test-fetch fails with an error equivalent to:

- `phase 2 scaffolding currently supports only pipeline.event.input.field = eventUrl.`

`baseUrl` is optional and applies only to `calendarFieldUrl` mode.

Behavior:

- relative `eventUrl` values are resolved against `input.baseUrl` when present
- when `baseUrl` is omitted, relative links fall back to feed-based resolution behavior
- if a list payload URL is absolute but points to an API host, `baseUrl` can rebase host/path to the public content host
- if URL normalization fails, detail fetch is skipped for that event

## How to satisfy this constraint

1. Ensure calendar mappings include `eventUrl`.
2. Use `field: "eventUrl"` in event input.
3. Add `baseUrl` when list links are relative or host-mismatched.
4. Validate with `/api/source-schemas/test-fetch`.

Example calendar mapping:

```json
"mappings": {
  "title": "h2",
  "startTime": ".date",
  "eventUrl": "a@href"
}
```

## FixedUrl mode note

`fixedUrl` remains part of the v3 contract. For contributor workflows, validate behavior in your target environment before depending on it.
