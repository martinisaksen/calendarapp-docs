# Detail Pages

Use detail-page enrichment when list pages provide incomplete data.

## Enable Detail Pages When

- description is truncated
- image is missing on list page
- venue or address exists only on detail page

## Supported Fields

- detailPage.enabled
- detailPage.linkSelector
- detailPage.detailMappings

`detailMappings` supports the same keys as `mappings`, including `title`, `description`, `location`, `venueName`, `venueAddress`, `url`, `imageUrl`, `startDate`, `startTime`, `endDate`, `endTime`, and `recurrenceRule`.

Merge behavior:

- Listing-page values are parsed first.
- Detail-page values override listing values when non-empty.
- For split time extraction, detail `startDate` + `startTime` can replace a placeholder list `startTime`.

Rules:

- `detailPage.enabled` must be `true` to activate detail fetches.
- Runtime detail enrichment must also be enabled with environment variable `CALENDARAPP_HTMLLITE_DETAIL_ENRICHMENT_ENABLED`.
- If the environment variable is missing, detail enrichment defaults to enabled.
- `detailPage.linkSelector` must resolve to a detail URL from the list card scope.
- `detailMappings` selectors run against the detail-page DOM.

## Example

```json
{
  "detailPage": {
    "enabled": true,
    "linkSelector": "a@href",
    "detailMappings": {
      "title": "h1",
      "startDate": ".event__date",
      "startTime": ".event__time",
      "description": ".event-full-description",
      "imageUrl": "img.hero@src",
      "venueAddress": ".venue-address",
      "recurrenceRule": ".recurrence-info::attr(data-rrule)"
    }
  }
}
```
