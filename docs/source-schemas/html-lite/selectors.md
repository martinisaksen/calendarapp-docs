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
