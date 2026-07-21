---
type: Context
created: YYYY-MM-DD
---

# Naming conventions

## CLOSED list of allowed tags

> [!warning] Never invent a tag on the fly — add it here first.

- `#tag-1`
- `#tag-2`
- `#tag-3`

## File naming rules

| Folder | Format | Example |
|--------|--------|---------|
| `wiki/` | descriptive kebab-case | `feedback-loop-model.md` |
| `wiki/decisions/` | verb + object | `choose-x-over-y.md` |
| `raw/` | `YYYY-MM-DD_description.md` (retrieval date) | `2026-05-21_source-name.md` |
| `daily/` | `YYYY-MM-DD.md` | `2026-07-20.md` |

## Internal Obsidian links

- `[[page-name]]` — without the `.md` extension
- `[[folder/page-name]]` — with the path if there's ambiguity
- Always weave links between related pages
