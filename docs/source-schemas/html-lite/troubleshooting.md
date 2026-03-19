# Troubleshooting

## Invalid request payload

Usually means schemaDefinition was sent as an object instead of a JSON string.

Fix:
- serialize schemaDefinition as a compact JSON string before submission
- verify request body uses `schemaDefinition: "{...}"` rather than nested object

## validation.totalEventsParsed = 0

- verify eventCardSelector first
- verify startTime extraction and date format
- prefer XPath for complex nested nodes
- confirm source is static HTML and not JS-only

If still zero, apply this fallback sequence:

1. Reduce `eventCardSelector` to a very stable repeated node (often title wrapper).
2. Use minimal mappings only: `title`, `url`, parseable `startTime`.
3. If list time text is unreliable, set temporary `startTime` to a `literal:` datetime.
4. Enable `detailPage` and map `startDate` + `startTime` there.
5. Add pagination after non-zero validation is achieved.

Common causes of false zero results:

- selector syntax uses unsupported CSS (`>`, `.a, .b`, pseudo-selectors)
- `linkSelector` does not produce a detail URL
- `startTime` cannot be parsed from list page
- mappings are objects instead of strings

Additional causes:
- `validation.requiredFields` includes fields not reliably mapped yet
- selector scope mismatch (list-card selector used against detail DOM or vice versa)

Quality anti-pattern:
- non-zero parse counts can still hide missing description/address/media fields
- if important fields are present on detail pages but absent in sample events, continue schema refinement

## validation.isSuccess = false

- inspect validation.errorMessage in submit response
- correct schemaDefinition and resubmit Draft

Common recovery loop:
1. fix one failure class at a time (selector, time parsing, pagination, detail link)
2. re-submit same Draft feedUrl
3. compare sampleEvents output to previous run
4. stop when parse count and sample quality are stable

If blocked by platform visibility limitations:
- prefer generic platform improvements (for example, richer sample event output) over source-specific runtime branches

## Expression must evaluate to a node-set

XPath is returning a scalar where a node query is required. Move attribute access into ::attr(...) after selecting a node.

If using HtmlLite custom attribute syntax, prefer `selector@attribute` or `selector::attr(attribute)`.

## Approved source cannot be edited via submit

If submit returns a 400 indicating source is already approved:
- do not continue retrying submit for that feedUrl
- hand off to admins for approved-source change process
