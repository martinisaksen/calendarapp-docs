# HtmlLite Platform Patterns

Use this page when the source is built on a well-known site builder or CMS that tends to produce repeatable DOM and link patterns.

Platform pages are intended to reduce repeated trial-and-error for common sources while keeping the core HtmlLite guidance generic.

## When To Use This Page

Use the platform-specific pages when:
- the site clearly matches a known builder such as Squarespace or Wix
- repeated onboarding work has shown stable extraction patterns for that platform
- you need recognition guidance before choosing selectors or source type details

Do not use platform pages as a substitute for source-type selection. Start with [Choose A Source Type](../choose-source-type.md) if you have not yet confirmed that `HtmlLite` is the right choice.

## Platform Page Structure

Each platform page should cover the same topics:
- when to use the page
- platform signatures and how to recognize it
- recommended source type by scenario
- common extraction patterns
- datetime pitfalls and preferred source-of-truth fields
- anti-patterns to avoid
- validation checklist
- troubleshooting and fallbacks
- minimal working example
- AI runbook guidance when an agent is performing the onboarding

## Available Platform Pages

- [Squarespace-Specific Patterns](squarespace-specific-patterns.md)
- [Wix-Specific Selector Patterns](wix-specific-patterns.md)

## Suggested Next Candidates

Add new platform pages only after repeated onboarding work shows a reusable pattern. Current likely candidates:
- WordPress event plugins
- Eventbrite-hosted or embedded event pages

## Shared Rules Across Platforms

- Prefer structured feeds over HtmlLite when a real ICS, RSS, or JSON source exists.
- Prefer canonical machine-readable fields over visible marketing copy.
- Keep transforms deterministic and bounded.
- Validate against parsed samples, not just selector matches.
- Document platform-specific assumptions in handoff notes when they affect long-term maintenance.