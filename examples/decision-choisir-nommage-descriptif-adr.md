---
tags: [meta, methodologie]
type: Decision
created: 2026-07-20
decision_date: 2026-07-20
status: acceptée
supersedes: ""
superseded_by: ""
related: ["[[vault-methodology-template]]"]
source: "[[sources/architecture-decision-record-repo]]"
---

# Choisir un nommage descriptif plutôt que numéroté pour les ADR

## Contexte

Le template ajoute une couche `wiki/decisions/` (ADR). Il faut fixer une convention de nommage des fichiers. Deux contraintes du vault pèsent sur le choix :

1. la navigation se fait par wikilinks `[[...]]` et par Graphify, **pas** par un index séquentiel ;
2. la convention existante du wiki est déjà le kebab-case descriptif (`feedback-loop-model.md`).

## Options envisagées

- **Numérotation séquentielle** (`adr-0001-*.md`, convention classique de beaucoup d'équipes) → **rejetée** : `[[adr-0007]]` est illisible dans un graphe de wikilinks, et l'ordre chronologique est déjà porté par `decision_date`. Le numéro n'apporte rien qu'Obsidian/Graphify ne donnent déjà.
- **Nommage descriptif impératif** (`choisir-x-plutot-que-y.md`, convention du repo de référence de Joel Parker Henderson) → **retenue** : cohérent avec le kebab-case existant, auto-descriptif dans les backlinks, requêtable en langage naturel.
- **Hybride** (`0001-choisir-x.md`, préfixe + description) → **gardée en option, pas par défaut** : cumule les deux mais alourdit le nom ; réservé aux vaults qui tiennent à un ordre chronologique visible dans l'explorateur.

## Décision

Nommage **descriptif et impératif** en kebab-case, sans numéro. Le préfixe numérique reste une option documentée pour qui veut un ordre visible.

## Conséquences

- ➕ Cohérence totale avec la convention wiki existante — une seule règle de nommage à retenir.
- ➕ Backlinks et requêtes Graphify lisibles (`[[choisir-postgres-plutot-que-mongo]]`).
- ➖ Pas d'ordre chronologique visible dans l'explorateur de fichiers → assumé, `decision_date` le porte.
- ➖ Renommer une décision casse ses wikilinks → à traiter comme n'importe quelle page wiki (mettre à jour les backlinks via `/lint`).
