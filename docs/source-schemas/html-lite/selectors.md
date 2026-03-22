# Selectors

## Discovery Workflow

1. Identify the repeating event card container.
2. Map title, time, url first.
3. Add description, location, image after submit validation succeeds.

Good card candidates:
- div.views-row
- li.event-card
- article.event
- div.event-teaser

## Supported Selector Patterns

- CSS-like selection:
  - tag, class, id, descendant by space
- XPath:
  - prefix with xpath=
- Attribute extraction:
  - selector::attr(name)
  - selector@attrName
- Literal values:
  - literal:Some fixed value

Examples:
- div.event-card
- .event-title a
- xpath=.//time[@datetime]
- a::attr(href)
- img@src

## Selector Scope Rules

- `eventCardSelector` runs against the listing page document.
- `mappings` selectors run relative to each matched event card node.
- `detailPage.linkSelector` runs relative to the matched event card node.
- `detailPage.detailMappings` selectors run against the fetched detail page document.

Scope example:

- If `eventCardSelector` is `h2.event__title` then mapping `title: "a"` selects the anchor inside that `<h2>`.
- In `detailMappings`, `title: "h1"` selects from the detail page root, not the list page card.

## CSS-like vs XPath

HtmlLite is not XPath-only. It supports a limited CSS-like subset and raw XPath.

Use XPath when:
- you need child combinators like `>`
- you need selector lists like `.a, .b`
- you need sibling logic, nth/first filters, or attribute predicates beyond the simple subset
- you need conditional logic or explicit node tests

XPath examples:
- xpath=.//time[@datetime]
- xpath=.//img[1]
- xpath=.//h2[@class='event-title']/a

XPath selectors with attribute predicates (for example `[@class='...']`) are supported.

Use CSS-like selectors when:
- matching simple class/id/tag structures
- using descendant-by-space selectors like `h2 a`
- extracting direct attributes with ::attr(...)

CSS-like examples:
- img::attr(src)
- a::attr(href)
- h2.event__title a

Avoid unsupported CSS syntax in HtmlLite:

- `>` child combinator
- selector lists separated by commas
- pseudo-selectors like `:first-child`
- adjacent and general sibling operators

If you need any of those, switch that selector to `xpath=` instead of trying to force full CSS syntax.

## Invalid vs Valid Examples

Chat models often try to hedge with multiple fallback selectors in a single string. That does not work in HtmlLite.

Invalid:

```json
{
  "eventCardSelector": ".event-listing, .events-listing-item, .event-item",
  "mappings": {
    "title": ".event-title, h3, h2"
  }
}
```

Why invalid:
- selector lists separated by commas are unsupported

Valid:

```json
{
  "eventCardSelector": "xpath=.//*[contains(@class,'event-listing') or contains(@class,'events-listing-item') or contains(@class,'event-item')]",
  "mappings": {
    "title": "xpath=.//*[contains(@class,'event-title') or self::h3 or self::h2][1]"
  }
}
```

Also avoid invented selectors that are not proven by the page source. A weaker but evidenced selector is better than a broader guessed one.
