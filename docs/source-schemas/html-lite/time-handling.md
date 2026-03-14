# Time Handling

Time parsing is a common failure point.

## Preferred Extraction Order

1. time datetime attribute
2. schema.org or meta startDate
3. data-start or similar data attributes
4. visible text parsing

Examples:
- Preferred: xpath=.//time[@datetime]::attr(datetime)
- Riskier: .event-time
- Risky: visible text like 10:30 am - 11:30 am

## Split Date And Time Support

When the page separates date and time into different nodes, map:

- `startDate` to the visible date node
- `startTime` to either a time-only value or a visible range like `10:30 AM - 11:30 AM`
- optionally `endDate` and `endTime` if the end date is exposed separately

Example:

```json
{
	"mappings": {
		"title": "h2 a",
		"startDate": ".event__date",
		"startTime": ".event__time",
		"url": "h2 a@href"
	}
}
```

Behavior:

- If `startTime` is already a full datetime, HtmlLite uses it directly.
- If `startDate` is present and `startTime` is time-only, HtmlLite combines them.
- If `startTime` is a visible range like `10:30 AM - 11:30 AM`, HtmlLite derives both `StartTime` and `EndTime`.
- If no end can be derived, `EndTime` still defaults to `StartTime + 1 hour`.

## Validation Tips

- verify submit response validation.sampleEvents have parseable datetime values
- verify timezone assumptions
- verify AM/PM and all-day handling

## Default Behavior

- EndTime defaults to StartTime + 1 hour when no endTime mapping is provided
- IsAllDay defaults to false
