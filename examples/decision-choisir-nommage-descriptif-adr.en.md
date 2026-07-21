---
tags: [meta, methodology]
type: Decision
created: 2026-07-20
decision_date: 2026-07-20
status: accepted
supersedes: ""
superseded_by: ""
related: ["[[vault-methodology-template]]"]
source: "[[sources/architecture-decision-record-repo]]"
---

# Choose descriptive naming over numbering for ADRs

## Context

The template adds a `wiki/decisions/` layer (ADR). A file naming convention needs to be fixed. Two vault constraints weigh on this choice:

1. navigation happens via `[[wikilinks]]` and Graphify, **not** via a sequential index;
2. the wiki's existing convention is already descriptive kebab-case (`feedback-loop-model.md`).

## Options considered

- **Sequential numbering** (`adr-0001-*.md`, the classic convention used by many teams) → **rejected**: `[[adr-0007]]` is unreadable in a wikilink graph, and chronological order is already carried by `decision_date`. The number adds nothing that Obsidian/Graphify don't already provide.
- **Descriptive, imperative naming** (`choose-x-over-y.md`, the convention used in Joel Parker Henderson's reference repo) → **chosen**: consistent with the existing kebab-case convention, self-descriptive in backlinks, queryable in natural language.
- **Hybrid** (`0001-choose-x.md`, prefix + description) → **kept as an option, not the default**: combines both but makes the name heavier; reserved for vaults that care about a visible chronological order in the file explorer.

## Decision

**Descriptive, imperative** kebab-case naming, no numbering. The numeric prefix stays a documented option for anyone who wants a visible order.

## Consequences

- ➕ Full consistency with the existing wiki convention — a single naming rule to remember.
- ➕ Readable backlinks and Graphify queries (`[[choose-postgres-over-mongo]]`).
- ➖ No visible chronological order in the file explorer → accepted trade-off, `decision_date` carries it.
- ➖ Renaming a decision breaks its wikilinks → handle it like any other wiki page (update backlinks via `/lint`).
