# 🧠 Obsidian Vault Methodology Template

A structured methodology for building a **persistent knowledge base** in Obsidian, powered by Claude Code agents.

The core idea: instead of using AI to answer questions from scratch each time, you compile knowledge permanently into a wiki. Each conversation enriches the base — nothing disappears into chat history.

---

## What's inside

- **Architecture** — a layered folder structure (`raw/`, `wiki/`, `context/`, `daily/`, `projects/`)
- **5 slash commands** — `/ingest`, `/query`, `/lint`, `/save`, `/purge`, `/obsidian-challenge`
- **5 specialized agents** — each agent receives only the context it needs (~40% token savings)
- **Wiki page structure** — frontmatter, TL;DR, limits & counter-examples, relations
- **Naming conventions** — closed tag lists, kebab-case, ISO date prefixes
- **Initialization checklist** — step-by-step from zero to first ingestion
- **Split vault rules** — when to create a new vault (thematic + 500-page hard cap)

---

## How to use

1. Copy `vault-methodology-template.md` into your Obsidian vault root
2. Start with `context/ligne-rouge.md` — write your non-negotiable constraint for this vault
3. Follow the initialization checklist at the bottom of the template
4. Adapt the domain examples to your field

> The template is intentionally domain-agnostic. The examples in the original vault were analytics engineering — they've been replaced with generic placeholders.

---

## Philosophy

```
Source → Ingest → Wiki → Query → Synthesize → Wiki (enriched)
```

Knowledge compounds. A vault used for 6 months becomes exponentially more useful than one used for 6 days — because every query builds on prior ingestion.

---

## License

[CC BY-NC 4.0](LICENSE) — free to use and adapt, attribution required, commercial use prohibited.
