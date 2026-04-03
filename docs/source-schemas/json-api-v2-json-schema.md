# JsonApi V2 JSON Schema

CalendarApp now publishes a contributor-facing JSON Schema file for `JsonApi` `schemaDefinition` v2.

Use it to improve accuracy when authoring schemas manually or generating them with AI.

## Files

- Source schema catalog: [Source Schema JSON Schema Files](json-schema-files.md)
- Raw schema: [json-api-v2.schema.json](json-api-v2.schema.json)
- Primary usage guide: [JsonApi Source Guide](json-api.md)
- Submission and validation rules: [Submission API And Validation](submission-api-and-validation.md)

## Scope

This schema is intended to help contributors and AI produce valid `JsonApi` v2 source schemas more consistently.

It covers:

- required core shape for `schemaVersion = 2`
- supported top-level fields
- enums for pagination, auth, token, query-template, and response-transform settings
- structural limits that are practical to express in JSON Schema

It does not replace backend validation.

The backend remains authoritative for:

- network and host restrictions
- secret/header redaction behavior
- some cross-field runtime validations
- parser behavior and downstream parsing expectations

## Practical Guidance

- Use this v2 schema only for historical reference and legacy schema audits.
- For new `JsonApi` sources, use the v3 pipeline contract (`schemaVersion = 3`).
- If the source splits date and time across fields, use `responseTransforms` to compose parser-ready `startTime` and `endTime` values.
- If the source provides timezone, map `timeZone` from the source.
- If transforms reshape the effective event payload, set `responseTransforms.flattenPath` explicitly.

## Validation Example

If you use a JSON Schema validator such as `ajv`, validate the `schemaDefinition` object itself against this file.

Example validator input:

```json
{
  "schemaVersion": 2,
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

Remember that the API request still sends `schemaDefinition` as a JSON string in the outer request body. The JSON Schema file validates the inner object before you serialize it.