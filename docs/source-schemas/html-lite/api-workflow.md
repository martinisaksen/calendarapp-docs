# HtmlLite API Workflow

This page defines the API-side contract for both manual and autonomous onboarding.

Use this page only after you have already chosen HtmlLite as the source type. For shared contributor rules that apply to every source type, see [Submission API And Validation](../submission-api-and-validation.md).

## Step 1: Test Fetch (No Draft Created)

Call:
- POST /api/source-schemas/test-fetch

Include:
- type = HtmlLite
- feedUrl
- schemaDefinition as JSON string

Use test-fetch while refining selectors so you do not create unnecessary draft revisions.

## Step 2: Submit Community Draft (With Built-in Validation)

Call:
- POST /api/source-schemas/community-submissions

Include:
- name
- description
- type = HtmlLite
- feedUrl
- schemaDefinition as JSON string
- metadata (location, category, region, language, estimatedEventCount)

The endpoint behaviour depends on whether the `feedUrl` has been seen before:

| Scenario | HTTP Status | Outcome |
|---|---|---|
| New feedUrl | 201 Created | Draft created, validation result returned |
| Existing Draft (same feedUrl) | 200 OK | Draft overwritten with new values, validation result returned |
| Existing Approved source | 400 Bad Request | Error — source cannot be edited via submit |

Response (201 or 200) includes:
- sourceSchema
- validation.isSuccess
- validation.totalEventsParsed
- validation.sampleEvents
- validation.errorMessage

Operational notes:
- Re-submitting the same Draft `feedUrl` is expected during refinement.
- A successful HTTP response does not imply parsing success. Always inspect `validation`.

## Response Examples

Success example:

```json
{
  "sourceSchema": {
    "id": "1acba9be-d70c-435a-8efb-123e350638e5",
    "name": "Example Events - HtmlLite",
    "type": "HtmlLite",
    "feedUrl": "https://www.example.org/events",
    "status": "Draft"
  },
  "validation": {
    "isSuccess": true,
    "totalEventsParsed": 12,
    "sampleEvents": [
      {
        "title": "Community Yoga",
        "startTime": "2026-04-20T18:00:00-07:00",
        "endTime": "2026-04-20T19:00:00-07:00",
        "location": "Riverfront Center",
        "eventUrl": "https://www.example.org/events/yoga"
      }
    ],
    "errorMessage": null
  }
}
```

Failure example:

```json
{
  "sourceSchema": {
    "id": "7d88c4a3-c9d4-4c4d-beb1-b9d71e590fa3",
    "name": "Example Events - HtmlLite",
    "type": "HtmlLite",
    "feedUrl": "https://www.example.org/events",
    "status": "Draft"
  },
  "validation": {
    "isSuccess": false,
    "totalEventsParsed": 0,
    "sampleEvents": [],
    "errorMessage": "Selector returned zero event cards"
  }
}
```

Approved source error (400):

```json
{
  "error": "This source is already approved and cannot be edited via submit."
}
```

## Step 3: Review Validation Result

Validation pass:
- validation.isSuccess = true
- validation.totalEventsParsed is non-zero and reasonable

Validation failure:
- validation.isSuccess = false
- review validation.errorMessage
- fix schema and re-submit the same feedUrl (returns 200 and overwrites the Draft)

Recommended interpretation gates:
1. `validation.isSuccess` is `true`
2. `validation.totalEventsParsed` is non-zero and plausible
3. `validation.sampleEvents` show valid title/startTime/url values

## Step 4: Handoff To Admin Review

After Draft submission, hand off schema ID and validation notes to admins.

Community and external AI contributors should not call admin-only approval or trigger endpoints.

Handoff packet should include:
- sourceSchema ID
- feedUrl
- final schemaDefinition
- validation summary (`isSuccess`, `totalEventsParsed`, sample event quality)

## PowerShell Example

```powershell
$base = 'http://localhost:5047/api/source-schemas'

$schemaObject = @{
  eventCardSelector = 'div.event-teaser'
  mappings = @{
    id = 'a::attr(href)'
    title = 'h3'
    startTime = 'xpath=.//time[@datetime]::attr(datetime)'
    url = 'a::attr(href)'
  }
  validation = @{
    requiredFields = @('title','startTime')
  }
}

$schemaDefinition = $schemaObject | ConvertTo-Json -Depth 20 -Compress

$submitBody = @{
  name = 'Example Events - HtmlLite'
  description = 'Community-submitted HtmlLite source'
  type = 'HtmlLite'
  feedUrl = 'https://www.example.org/events'
  schemaDefinition = $schemaDefinition
  metadata = @{
    location = 'Example City'
    category = 'community'
    region = 'Example Region'
    language = 'en'
    estimatedEventCount = 10
  }
} | ConvertTo-Json -Depth 12

$created = Invoke-RestMethod -Uri "$base/community-submissions" -Method Post -ContentType 'application/json' -Body $submitBody
$created | ConvertTo-Json -Depth 8

# Save schema ID for admin handoff
$id = $created.sourceSchema.id
"Submitted Draft SourceSchema ID: $id"

# Validation result from submit response
$validation = $created.validation
"Validation success: $($validation.isSuccess)"
"Parsed events: $($validation.totalEventsParsed)"
if (-not $validation.isSuccess) {
  "Validation error: $($validation.errorMessage)"
}
```

Related references:
- [Choose A Source Type](../choose-source-type.md)
- [Submission API And Validation](../submission-api-and-validation.md)
- [Schema Basics](schema-basics.md)
- [Validation Checklist](validation-checklist.md)
- [Troubleshooting](troubleshooting.md)
