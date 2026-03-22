# Submission API And Validation

This page describes the shared contributor workflow for every source type.

Use these endpoints:

- `GET /api/source-schemas/templates`
- `POST /api/source-schemas/test-fetch`
- `POST /api/source-schemas/community-submissions`

Do not use admin-only approval or trigger endpoints as a community contributor.

## Shared Request Envelope

Every submission uses the same top-level fields:

```json
{
  "name": "Example Source",
  "description": "What this source publishes",
  "type": "JsonApi",
  "feedUrl": "https://example.org/events",
  "schemaDefinition": "{\"eventArrayPath\":\"$.events[*]\",\"mappings\":{\"title\":\"$.title\",\"startTime\":\"$.start\"}}",
  "metadata": {
    "location": "Example City",
    "category": "community",
    "region": "WA",
    "language": "en",
    "estimatedEventCount": 20
  },
  "calendarStrategyOverride": "Auto"
}
```

Important:

- `type` must be one of `Ics`, `Rss`, `JsonApi`, `HtmlLite`
- `feedUrl` is required and must be a valid absolute URL
- `schemaDefinition` must be sent as a JSON string, not a nested object
- `schemaDefinition` can be up to 200000 characters

`schemaDefinition` must contain only fields supported by the selected source type. Do not include wrapper or note fields such as:

- `sourceName`
- `baseUrl`
- `transforms`

Those do not belong in the submitted schema body.

## Contributor-Safe Workflow

### 1. Load A Starter Template

Use `GET /api/source-schemas/templates` when you want a known-good starting point.

The templates endpoint currently publishes starter schemas for:
- `Ics`
- `Rss`
- several `JsonApi` patterns

If the templates endpoint does not include the exact type or shape you need, start from the type guide on this site.

### 2. Test Fetch Before Submitting

Use `POST /api/source-schemas/test-fetch` while refining the schema.

Example:

```json
{
  "type": "Rss",
  "feedUrl": "https://example.org/events/feed.rss",
  "schemaDefinition": "{\"extractionRules\":{\"title\":\"title\",\"startTime\":\"pubDate\"},\"validation\":{\"requiredFields\":[\"title\",\"startTime\"]}}"
}
```

### 3. Submit The Draft

Use `POST /api/source-schemas/community-submissions` once test-fetch is stable.

The response returns:
- `sourceSchema`
- `validation.isSuccess`
- `validation.totalEventsParsed`
- `validation.sampleEvents`
- `validation.errorMessage`
- `validation.validationErrors`

## Shared Validation Rules

The backend enforces these common rules for every source type:

- `schemaDefinition` must be valid JSON
- `schemaDefinition` cannot exceed the max depth limit of 30
- `mappings` cannot exceed 300 entries when present
- `requestWorkflow.steps` cannot exceed 25 entries when present
- `requestHeaders` cannot exceed 50 entries when present
- outbound URLs must be `http` or `https`
- localhost, private-network, and restricted hosts are blocked

## JsonApi Additional Contract Rules

`JsonApi` has extra schema-contract validation:

- `schemaVersion` must be `1` or `2` when provided
- advanced features require `schemaVersion = 2`
- `validationProfile = Advanced` requires `schemaVersion = 2`
- `validationProfile = Basic` cannot be used with advanced features
- `requestWorkflow` and `pagination` cannot both be configured
- `pagination.maxRequestsPerRun` must be greater than `0` when provided
- dynamic token acquisition requires `tokenConfig.dynamicAcquisition.endpoint`

## Type-Specific Minimum Shapes

### Ics

```json
{
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

### Rss

```json
{
  "extractionRules": {
    "title": "title",
    "startTime": "pubDate"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

### JsonApi

```json
{
  "eventArrayPath": "$.events[*]",
  "mappings": {
    "title": "$.title",
    "startTime": "$.start"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

### HtmlLite

```json
{
  "eventCardSelector": "div.event-card",
  "mappings": {
    "title": "h3",
    "startTime": "time::attr(datetime)"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

## Common Invalid Patterns

These mistakes are common when generating schemas with chat models.

### Invalid: unsupported HtmlLite selector lists

```json
{
  "eventCardSelector": ".event-card, .featured-event",
  "mappings": {
    "title": ".event-title, h3, h2"
  }
}
```

Why invalid:
- HtmlLite does not support comma-separated selector lists.

Valid rewrite:

```json
{
  "eventCardSelector": "xpath=.//*[contains(@class,'event-card') or contains(@class,'featured-event')]",
  "mappings": {
    "title": "xpath=.//*[contains(@class,'event-title') or self::h3 or self::h2][1]"
  },
  "validation": {
    "requiredFields": ["title", "startTime"]
  }
}
```

### Invalid: unsupported HtmlLite pagination shape

```json
{
  "pagination": {
    "enabled": true,
    "nextPageSelector": "a.next"
  }
}
```

Valid rewrite:

```json
{
  "pagination": {
    "type": "nextLink",
    "nextPageSelector": "a.next@href",
    "maxPages": 5
  }
}
```

### Invalid: unrecognized HtmlLite fields

```json
{
  "baseUrl": "https://example.org",
  "transforms": [
    {
      "field": "url",
      "type": "absoluteUrl"
    }
  ]
}
```

Why invalid:
- `baseUrl` and `transforms` are not part of the HtmlLite contract.
- `url` and `imageUrl` are already resolved to absolute URLs downstream when valid relative links are captured.

### Invalid: wrong field name

```json
{
  "detailPage": {
    "enabled": true,
    "detailMappings": {
      "address": ".event-address"
    }
  }
}
```

Valid rewrite:

```json
{
  "detailPage": {
    "enabled": true,
    "detailMappings": {
      "venueAddress": ".event-address"
    }
  }
}
```

## Validation Pass Criteria

Treat a draft as ready for admin review only when:

1. `validation.isSuccess` is `true`
2. `validation.totalEventsParsed` is non-zero and plausible
3. `validation.sampleEvents` show good coverage for title, time, URL, and location when available
4. the source type is still the best available source after testing

## Contributor Handoff Checklist

Include:

- `sourceSchema.id`
- `feedUrl`
- final `schemaDefinition`
- validation summary
- any known limitations such as missing description, incomplete location, or pagination constraints