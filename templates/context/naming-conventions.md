---
type: Context
created: YYYY-MM-DD
---

# Conventions de nommage

## Liste FERMÉE des tags autorisés

> [!warning] Ne jamais inventer un tag à la volée — l'ajouter d'abord ici.

- `#tag-1`
- `#tag-2`
- `#tag-3`

## Règles de nommage des fichiers

| Dossier | Format | Exemple |
|---------|--------|---------|
| `wiki/` | kebab-case descriptif | `feedback-loop-model.md` |
| `wiki/decisions/` | verbe + objet | `choisir-x-plutot-que-y.md` |
| `raw/` | `YYYY-MM-DD_description.md` (date de récupération) | `2026-05-21_nom-source.md` |
| `daily/` | `YYYY-MM-DD.md` | `2026-07-20.md` |

## Liens internes Obsidian

- `[[nom-de-page]]` — sans extension `.md`
- `[[dossier/nom-de-page]]` — avec chemin si ambiguïté
- Toujours tisser des liens entre pages connexes
