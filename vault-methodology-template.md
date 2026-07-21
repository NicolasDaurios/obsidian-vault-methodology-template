# Vault Methodology Template

*[English version](vault-methodology-template.en.md)*

**Version**: 4.1
**Auteur**: Nicolas / Datashiru
**Créé le**: 2026-04-24
**Mis à jour le**: 2026-07-20
**Inspiration**: Andrej Karpathy (mémoire LLM) + Pillitteri/Second cerveau + agricidaniel/claude-obsidian + Carpati/oubli-stratégique + Joel Parker Henderson (ADR)
**Licence**: [CC BY-NC 4.0](LICENSE) — Nicolas / Datashiru.
**Usage**: Adapte ce template à ton domaine. Commence par `context/ligne-rouge.md`.

---

## PRINCIPE FONDATEUR

**Rôle de l'IA** : Tu es l'Agent LLM Wiki. Tu es le programmeur, le wiki est ta base de code (Markdown), et Obsidian est ton IDE.

**Objectif** : Compiler la connaissance de manière persistante — pas simplement piocher des extraits (RAG classique), mais construire un savoir structuré, traçable et réutilisable qui devient plus utile exponentiellement.

**Philosophie structurelle (principe MECE)** : chaque fichier doit avoir **une place claire et une seule**. Les orphelins signalent une architecture défaillante. Si un contenu ne rentre nulle part, créer une catégorie plutôt que forcer.

La boucle d'apprentissage continue :

```
Source brute → Nettoyage → Ingestion → Wiki structuré → Query → Synthèse → Wiki enrichi
     ↑                                                                            |
     └────────────────────── Daily → Projects → Context ←───────────────────────┘
```

> **Acte fondateur** : avant de créer la moindre structure, répondre à cette question : *quelle est mon exigence irréductible pour ce vault ?* Écrire la réponse dans `context/ligne-rouge.md`. Tout le reste — l'ontologie, le scope, les conventions — en découle. Un vault sans ligne rouge est un système sans colonne vertébrale.

---

## ARCHITECTURE COMPLÈTE

```
[RACINE DU VAULT]
│
├── raw/                        → Sources brutes immuables (articles, docs, transcriptions, PDF→MD)
│
├── wiki/                       → Connaissance permanente compilée par les agents
│   ├── concepts/               → Idées abstraites, modèles mentaux, théories, frameworks
│   ├── entities/               → Personnes, entreprises, projets, technologies spécifiques
│   ├── patterns/               → Templates réutilisables, recettes, méthodes step-by-step
│   ├── case-studies/           → Histoires réelles, projets concrets, retours d'expérience
│   ├── sources/                → Métadonnées et résumés des fichiers présents dans raw/
│   ├── syntheses/              → Synthèses complexes générées lors des sessions /query
│   ├── tools/                  → Setup, configurations, intégrations d'outils
│   ├── lessons/                → Erreurs, gotchas, pièges à éviter
│   └── decisions/              → recommandé — journal de décisions (ADR) : le "pourquoi" (voir plus bas)
│
├── context/                    → Identité du vault (qui tu es, ton domaine, tes conventions)
│   ├── identity.md             → Ton positionnement, valeurs, mission, cible
│   ├── naming-conventions.md   → Liste FERMÉE des tags autorisés + règles de nommage
│   └── scope.md                → Domaine du vault, sous-domaines, hors scope
│
├── daily/                      → Notes quotidiennes (opérationnel, non permanent)
│   └── YYYY-MM-DD.md           → Objectifs du jour, décisions, tâches ouvertes
│
├── projects/                   → Initiatives à durée limitée (≠ connaissance permanente)
│   └── [nom-projet]/
│       ├── brief.md
│       └── log.md
│
├── .claude/
│   ├── commands/               → Slash commandes (ingest, query, lint, save)
│   ├── agents/                 → Définition des sous-agents spécialisés
│   └── settings.local.json     → Config MCP locale
│
├── graphify-out/                → recommandé — graphe de connaissances queryable (voir plus bas)
│   ├── graph.json
│   ├── manifest.json
│   └── cache/                   → à exclure du versioning (régénérable)
│
├── cloud.md                    → Le "contrat" : règles, conventions (≤ 500 lignes)
├── index.md                    → Page d'entrée + navigation + stats du vault
├── log.md                      → Historique global de toutes les actions des agents
└── AGENTS.md                   → optionnel — protocole multi-agents (voir plus bas)
```

### Règles des couches

| Couche | Modifiable ? | Par qui ? | Rôle |
|--------|-------------|-----------|------|
| `raw/` | ❌ Jamais | — | Archive immuable des sources |
| `wiki/` | ✅ Oui | Agents | Connaissance permanente |
| `context/` | ✅ Oui | Humain | Identité et conventions du vault |
| `daily/` | ✅ Oui | Humain + Agents | Notes opérationnelles quotidiennes (rétention : 30 jours après clôture) |
| `projects/` | ✅ Oui | Humain + Agents | Projets actifs à durée limitée |
| `.claude/` | ✅ Oui | Humain | Comportements et agents |
| `cloud.md` | ✅ Oui | Humain | Contrat & conventions (≤ 500 lignes) |

---

## SOUS-AGENTS SPÉCIALISÉS

Pour éviter la saturation du contexte et réduire les coûts (~40% de tokens économisés), chaque tâche est déléguée à un **sous-agent isolé**. Chaque agent reçoit uniquement le contexte nécessaire à sa tâche.

> **Note pratique** : en l'absence de support natif des sous-agents dans Claude Code, chaque commande doit explicitement lister les fichiers à charger pour limiter la fenêtre de contexte.

### Wiki Ingestor (`.claude/agents/ingestor.md`)
- **Déclencheur** : `/ingest`
- **Rôle** : Lit `raw/`, nettoie les distracteurs, extrait les faits saillants, crée les pages wiki
- **Contexte reçu** : Fichier source + `cloud.md` + `context/naming-conventions.md`
- **Output** : Pages créées/mises à jour dans `wiki/` + entrée dans `log.md`

### Wiki Librarian (`.claude/agents/librarian.md`)
- **Déclencheur** : `/lint`
- **Rôle** : Indexation, création de liens, santé globale du vault
- **Contexte reçu** : Liste de toutes les pages wiki + `cloud.md`
- **Output** : Rapport de santé + corrections proposées + `log.md` mis à jour

### Wiki Query (`.claude/agents/query.md`)
- **Déclencheur** : `/query`
- **Rôle** : Interroge la base et retourne uniquement la réponse finale
- **Contexte reçu** : La question + pages wiki pertinentes uniquement (pas tout le vault)
- **Output** : Réponse sourcée avec `[[liens]]` + offre de `/save`

---

## COUCHE GRAPHE DE CONNAISSANCES (GRAPHIFY) — recommandé

Au-delà d'un certain volume, un agent qui doit relire l'intégralité du wiki pour répondre à une question devient lent et coûteux en tokens. [Graphify](https://github.com/Graphify-Labs/graphify) résout ça en indexant le vault dans un graphe queryable (`graphify-out/graph.json`) : nœuds (pages, concepts) + arêtes (relations explicites et sémantiques déduites) + communautés détectées.

### Skill "Read Graph"

Protocole en 3 étapes, à documenter dans un `AGENTS.md` à la racine (fichier neutre, lisible par n'importe quel agent — Claude Code, GPT, ou un agent tournant sur un autre serveur — pas seulement `CLAUDE.md` qui est spécifique à Claude Code) :

1. **Consultation prioritaire du graphe** — lire `graphify-out/graph.json` avant de parcourir les fichiers Markdown.
2. **Identification ciblée des fichiers** — repérer les chemins et métadonnées pertinents depuis le graphe, sans ouvrir chaque document.
3. **Lecture finale sélective** — n'ouvrir le contenu complet d'un fichier que si c'est strictement nécessaire.

**Pourquoi** : cette technique diminue la consommation de tokens par rapport à une lecture directe de l'ensemble des fichiers. Et si plusieurs agents suivent le même protocole sur le même graphe, ils répondent depuis la même structure — cohérence multi-agents plutôt que chaque agent qui relit le vault à sa façon.

### Multi-agents et distribution

Si plusieurs agents doivent consulter le même vault depuis des environnements différents (ex: un agent local + un agent sur VPS), privilégier une synchro **Git en lecture seule** (l'agent distant fait un `git pull`, jamais de push direct) plutôt qu'exposer le vault sur le réseau — évite l'exposition réseau d'une machine perso et les conflits de merge multi-writer. Le vault reste géré par un seul point d'écriture (l'humain + son agent principal) ; un retour d'un agent distant est remonté et intégré manuellement, jamais mergé automatiquement.

> **Mise en œuvre de référence (auteur)** : l'architecture est bâtie autour de **Claude Code** (agent principal, en écriture) + **Hermès** (harnais agentique) tournant sur un **VPS** en tâche de fond, le vault synchronisé en Git lecture seule. Ce n'est qu'une instanciation possible — la méthode, en Markdown pur, s'applique à de multiples situations : base de connaissances perso, wiki d'équipe ou d'organisation, documentation de projet, veille, travail client, recherche.

### Hygiène git (leçon apprise)

`graphify-out/cache/` (cache d'extraction, régénérable) et les dossiers de backup datés créés avant chaque re-clustering ne doivent **jamais** être versionnés — ils gonflent l'historique git sans apporter de valeur à un agent consommateur, qui n'a besoin que de `graph.json`/`manifest.json`. À ajouter au `.gitignore` :

```
graphify-out/cache/
graphify-out/20[0-9][0-9]-[0-9][0-9]-[0-9][0-9]/
```

Par défaut, Graphify indexe aussi les fichiers de config locaux (`.obsidian/*.json`, `.claude/settings.local.json`) s'ils ne sont pas déjà exclus par le `.gitignore` (Graphify s'appuie dessus pour ses propres exclusions) — ça crée des nœuds isolés qui polluent le graphe sans lien avec le reste. Les exclure du `.gitignore` avant le premier run évite d'avoir à les purger a posteriori.

---

## COUCHE DÉCISIONS (ADR) — recommandé

Les décisions structurantes sont aujourd'hui capturées dans `daily/` (section « Décisions prises ») — mais `daily/` est purgé à 30 jours. Résultat : le **pourquoi** d'un choix est éphémère par construction, alors que c'est souvent l'information la plus coûteuse à reconstituer six mois plus tard. Un [ADR — Architecture Decision Record](https://github.com/joelparkerhenderson/architecture-decision-record) est une page qui fige ce pourquoi : le contexte, les options envisagées, le choix retenu et ses conséquences.

Ce n'est pas réservé à l'ingénierie : un médecin trace *pourquoi ce protocole plutôt qu'un autre*, un juriste *pourquoi cette stratégie*, un consultant *pourquoi cette recommandation*. La mécanique est universelle ; seul le domaine change.

### Frontière avec `lessons/` et « contre-exemples » (règle MECE)

Le principe « une place claire et une seule » impose de distinguer nettement :

| Type | Nature | Question à laquelle il répond |
|------|--------|-------------------------------|
| **Decision** (`wiki/decisions/`) | Choix **prospectif** entre alternatives | « On prend X plutôt que Y — voici pourquoi » |
| **Lesson** (`wiki/lessons/`) | Constat **rétrospectif** | « X a cassé — voici le piège à éviter » |
| « Limites et contre-exemples » | Section **au sein d'une page** existante | « Ce pattern ne s'applique pas quand… » |

> **Test rapide** : y avait-il plusieurs routes possibles à trancher au moment T ? → **Decision**. Ai-je découvert un piège en avançant ? → **Lesson**.

### Structure d'une page décision (ADR)

Combinaison du template Nygard (le plus simple) et de la section « options » de MADR :

```markdown
---
tags: [tag1, tag2]
type: Decision
created: YYYY-MM-DD
decision_date: YYYY-MM-DD
status: proposée | acceptée | remplacée | dépréciée
supersedes: "[[decisions/ancienne-decision]]"     # si elle en remplace une
superseded_by: "[[decisions/nouvelle-decision]]"  # si elle a été remplacée
related: ["[[page-concernée]]"]
source: "[[sources/nom-de-la-source]]"
---

# [Verbe + objet — ex. « Choisir Postgres plutôt que Mongo »]

## Contexte
La force qui a déclenché la décision — le problème, la contrainte, l'échéance.

## Options envisagées
- **Option A** — [description] → retenue / rejetée car [raison]
- **Option B** — [description] → rejetée car [raison]
- **Option C (statu quo)** → rejetée car [raison]

## Décision
Ce qu'on a choisi et le rationale — pourquoi cette route et pas les autres.

## Conséquences
Ce qu'on accepte en échange — bénéfices ET coûts. Les compromis assumés.
```

> **Règle** : la section `## Options envisagées` est **obligatoire**. Un ADR sans alternative rejetée n'est pas une décision, c'est une annonce — il perd tout son intérêt (le « *plutôt qu'un autre* »).

### Statut et mutabilité

Les décisions ont leur **propre vocabulaire de statut**, distinct des statuts wiki :

| Statut | Signification |
|--------|--------------|
| `proposée` | En réflexion, pas encore actée |
| `acceptée` | Décision en vigueur |
| `remplacée` | Révisée par un ADR plus récent (`superseded_by`) |
| `dépréciée` | Plus pertinente, sans remplaçante |

Deux façons de faire évoluer une décision :

- **Revirement** (on change de choix) → créer une **nouvelle** page qui `supersedes` l'ancienne, et passer l'ancienne en `remplacée`. On garde la trace du *pourquoi on a changé d'avis* — c'est le cœur de la mémoire.
- **Clarification** (on précise sans changer le choix) → **annotation datée** dans la page existante. Pas besoin de superséder.

> Ne jamais réécrire silencieusement une décision `acceptée` pour la retourner : sans trace du revirement, l'amnésie revient par la fenêtre. (Le repo de référence assume cette mutabilité pragmatique — supersession pour un vrai revirement, annotation datée pour le reste ; pas d'immuabilité stricte.)

### Nommage

Comme le repo de référence : **descriptif et impératif**, kebab-case, sans numéro (`[[choisir-postgres]]` se lit, `[[adr-0007]]` non) :

- `wiki/decisions/choisir-postgres-plutot-que-mongo.md`
- `wiki/decisions/abandonner-le-cache-redis.md`

Préfixe numérique (`0001-`) uniquement si tu tiens à un ordre chronologique visible — optionnel.

### Le flux `daily → ADR` (le mécanisme qui fait vivre le module)

Un ADR ne s'écrit pas « en plus » du travail — il **gradue** depuis le quotidien :

```
Décision notée dans daily/ (« Décisions prises »)
        │
        ▼   si elle est structurante ET durable
Page wiki/decisions/ créée avant la purge des 30 jours
        │
        ▼
Indexée par Graphify → requêtable (« pourquoi a-t-on choisi X ? »)
```

Sans ce réflexe de graduation, le module est mort : les décisions restent dans `daily/` et disparaissent. **Règle : à la clôture d'une daily, toute décision qui survivra à ce mois-ci part en `wiki/decisions/`.**

---

## GUIDE DES CATÉGORIES

| Dossier | Contenu | Durée de vie |
|---------|---------|--------------|
| `wiki/concepts/` | Notions théoriques, frameworks, architectures | Permanente |
| `wiki/entities/` | Personnes, entreprises, projets, outils nommés | Permanente |
| `wiki/patterns/` | Templates, recettes, méthodes step-by-step | Permanente |
| `wiki/case-studies/` | Projets réels, retours d'expérience | Permanente |
| `wiki/sources/` | Métadonnées des fichiers `raw/` | Permanente |
| `wiki/syntheses/` | Synthèses issues de sessions `/query` | Permanente |
| `wiki/tools/` | Setup, configurations, intégrations | Permanente |
| `wiki/lessons/` | Erreurs, gotchas, pièges à éviter | Permanente |
| `wiki/decisions/` | Décisions tracées (ADR) : choix + options rejetées + conséquences | Permanente |
| `context/` | Identité, conventions, scope du vault | Semi-permanente |
| `daily/` | Notes quotidiennes opérationnelles | Temporaire |
| `projects/` | Initiatives à durée limitée | Temporaire |

> **Règle de granularité** : n'ajouter un nouveau sous-dossier que si **30+ notes** justifient la catégorie. Démarrer avec le minimum, étendre à la demande.

---

## CONVENTIONS DE NOMMAGE

### Fichiers `wiki/`
- **Format** : `kebab-case`, descriptif, sans abréviations, sans caractères spéciaux
- **Exemples** :
  - `wiki/concepts/feedback-loop-model.md`
  - `wiki/entities/nom-outil-ou-personne.md`
  - `wiki/patterns/template-prise-de-decision.md`
  - `wiki/sources/nom-du-livre-ou-article.md`
  - `wiki/syntheses/synthese-sur-sujet-cle.md`
  - `wiki/decisions/choisir-x-plutot-que-y.md`

### Fichiers `raw/`
- **Format** : `YYYY-MM-DD_source-description.md` — date ISO + tiret bas + description kebab-case
- **Règle** : la date est celle de récupération de la source, pas de publication
- **Exemples** :
  - `raw/2026-05-21_nom-livre-reference.md`
  - `raw/2026-05-15_notes-reunion-projet.md`
  - `raw/2026-04-10_youtube-titre-video.md`
  - `raw/2026-03-22_article-sujet-cle.md`

### Fichiers `daily/`
- **Format** : `YYYY-MM-DD.md`

### Tags
- La liste des tags autorisés est **fermée** et définie dans `context/naming-conventions.md`
- Ne jamais inventer un nouveau tag à la volée — l'ajouter d'abord à la liste
- Format : `#kebab-case`

### Liens internes Obsidian
- `[[nom-de-page]]` → sans extension `.md`
- `[[dossier/nom-de-page]]` → avec chemin si ambiguïté
- Toujours tisser des liens entre pages connexes

### Qualité du Markdown source (`raw/`)
Avant ingestion, s'assurer que le Markdown est **propre** :
- Publicités, menus, footers, éléments HTML parasites supprimés
- OCR ou conversion PDF effectuée via un outil de qualité (ex: Mistral Document AI)
- Un Markdown sale réduit les performances d'ingestion de **20 à 60%**

---

## STRUCTURE D'UNE PAGE WIKI

```markdown
---
tags: [tag1, tag2, tag3]
type: Concept | Entity | Pattern | Case Study | Source | Synthesis | Tool | Lesson | Decision
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
status: actif | en-revision | a-verifier | obsolete
related: ["[[page1]]", "[[page2]]"]
part_of: "[[concept-parent]]"
requires: ["[[prerequis]]"]
used_in: ["[[case-study-ou-pattern]]"]
source: "[[sources/nom-de-la-source]]"
---

# [Titre de la page]

## TL;DR (5 lignes max)
Résumé exécutif — ce que cette page apporte en un coup d'œil.

---

## Contenu principal

### Sous-section 1
[Contenu]

### Sous-section 2
[Contenu]

---

## Limites et contre-exemples

- Cas où ce pattern ne s'applique pas : [contexte]
- Approche abandonnée et pourquoi : [raison]
- Signal d'alerte pour ne pas l'utiliser : [trigger]

---

## Metrics (si applicable)

- Gain de performance : X%
- Temps d'exécution : Y sec
- Score de complexité : Z/10
```

> **Règle** : la section `## Limites et contre-exemples` est obligatoire pour les types Pattern et Lesson. Optionnelle pour Concept et Case Study. Un pattern sans contre-exemples est une vitrine, pas un outil de travail.

---

## STRUCTURE D'UNE NOTE DAILY

```markdown
---
status: en-cours | cloturee
---

# Daily — YYYY-MM-DD

## Objectifs du jour
- [ ] Objectif 1
- [ ] Objectif 2

---

## Tâches ouvertes (report du jour précédent)
- [ ] Tâche reportée 1

---

## Décisions prises
- Décision 1 : [contexte + choix]

---

## Apprentissages du jour
- Apprentissage 1

---

## À reporter demain
- [ ] Tâche non terminée

---

## Pages wiki mises à jour aujourd'hui
- [[page-mise-a-jour]]
```

### Politique de rétention `daily/`

| État | Délai | Action |
|------|-------|--------|
| ✅ Clôturée, apprentissages transférés au wiki | 30 jours | Supprimer |
| ✅ Clôturée, aucun apprentissage transféré | 7 jours | Relire → transférer ou supprimer |
| 🔄 En cours (session ouverte) | — | Conserver |

> Une note daily clôturée depuis plus de 30 jours sans transfert wiki est du bruit, pas de la mémoire.

---

## STATUTS DE CONTENU

| Valeur YAML | Signification |
|-------------|--------------|
| `actif` | Page à jour, fiable, utilisable immédiatement |
| `en-revision` | Contenu présent mais nécessite vérification |
| `a-verifier` | Contradictoire, incomplet ou non testé |
| `obsolete` | Plus pertinent — à archiver ou supprimer |

---

## SLASH COMMANDES

### `/ingest [fichier]`
**But** : Transformer une source brute (`raw/`) en pages wiki via le **Wiki Ingestor**.

**Processus** :
1. Vérifier que le Markdown source est propre
2. Vérifier que le fichier respecte la convention `YYYY-MM-DD_description.md` — signaler si non conforme
3. Lire le fichier source dans `raw/`
4. Identifier : concepts, entités, patterns, lessons, cas d'usage
5. Créer ou mettre à jour les pages wiki correspondantes
6. Si `wiki/sources/` existe → créer `wiki/sources/[nom-source].md` avec les métadonnées
7. Ajouter des liens entre les nouvelles pages et les existantes
8. Logger dans `log.md`
9. Confirmer : "✅ Ingestion complétée : X pages créées, Y pages mises à jour — ratio de compression : Z:1"

**Garde-fous** :
- ❌ Ne jamais modifier `raw/`
- ❌ Vérifier les doublons avant de créer
- ❌ Ne pas créer sans frontmatter complet
- ❌ Utiliser uniquement les tags de `context/naming-conventions.md`
- ⚠️ Signal de sur-ingestion : si pages créées > 20% du volume source, vérifier la granularité — l'ingestor compile, il ne copie pas

---

### `/query [question]`
**But** : Répondre depuis le wiki via le **Wiki Query**, pas depuis la connaissance générale.

**Processus** :
1. Identifier les concepts clés
2. Charger uniquement les pages wiki pertinentes
3. Assembler une réponse sourcée avec `[[liens]]`
4. Offrir : "Veux-tu que j'ajoute ce résumé au wiki ?"
5. Logger dans `log.md`

**Garde-fous** :
- ❌ Ne pas inventer si le wiki est vide sur le sujet
- ❌ Ne pas charger tout le vault — uniquement les pages pertinentes
- ❌ Toujours chercher dans le wiki avant la connaissance générale

---

### `/lint`
**But** : Vérifier la santé du vault via le **Wiki Librarian**.

**Processus** :
1. Parcourir tout `wiki/` et ses sous-dossiers
2. Détecter :
   - ❌ **Orphelins** : pages sans lien entrant
   - ❌ **Doublons** : deux pages sur le même concept
   - ❌ **Contradictions** : infos conflictuelles
   - ❌ **Formatage** : frontmatter manquant ou incomplet
   - ❌ **Liens cassés** : `[[dead-link]]` inexistant
   - ❌ **Stale content** : pages non mises à jour depuis 6+ mois
   - ❌ **Exemples manquants** : pattern sans exemple concret
   - ❌ **Tags hors liste** : tags non définis dans `context/naming-conventions.md`
   - ❌ **Sources non liées** : pages sans `[[sources/...]]`
3. Produire un rapport par catégorie
4. Proposer les corrections
5. Logger dans `log.md`

**Fréquence** :
- Hebdomadaire : orphelins + liens cassés + tags hors liste
- Mensuelle : lint complet

**Garde-fous** :
- ❌ Ne pas supprimer sans confirmation
- ❌ Ne pas merger automatiquement — proposer d'abord
- ❌ Ne modifier que le formatage, pas le contenu substantiel

---

### `/save [titre]`
**But** : Convertir une réponse en page wiki permanente.

**Processus** :
1. Récupérer la réponse à sauvegarder
2. Déterminer le type et le dossier destination
3. Vérifier l'absence de doublon
4. Créer la page avec frontmatter complet + backlinks + lien source
5. Logger dans `log.md`
6. Confirmer : "✅ Page créée : wiki/[dossier]/[titre].md"

**Garde-fous** :
- ❌ Reformater proprement, ne pas copier-coller brut
- ❌ Toujours frontmatter + backlinks
- ❌ Ne pas sauver une réponse incomplète ou low-quality
- ❌ Ne pas sauver si doublon existant

---

### `/adr [titre]` — optionnel (couche décisions)

**But** : Capturer une décision au format ADR dans `wiki/decisions/`.

**Processus** :
1. Récupérer la décision (depuis la daily en cours ou la question posée)
2. Reconstituer : contexte, options envisagées (dont celles rejetées), choix, conséquences
3. Créer `wiki/decisions/[verbe-objet].md` avec le frontmatter décision complet
4. Si elle en remplace une autre → renseigner `supersedes` / `superseded_by` et passer l'ancienne en `remplacée`
5. Lier aux pages concernées (le tool, concept ou pattern que la décision justifie)
6. Logger dans `log.md`

**Garde-fous** :
- ❌ Refuser de créer un ADR sans au moins une option rejetée — sinon ce n'est pas une décision
- ❌ Ne pas réécrire une décision `acceptée` pour la retourner → superséder à la place
- ❌ Distinguer Decision (choix prospectif) de Lesson (constat rétrospectif)

---

## RÈGLE DE SPLIT VAULT

Un vault = un domaine cohérent. Deux seuils distincts à surveiller :

### Seuil thématique — signal de divergence de domaine

Créer un **nouveau vault séparé** si :

| Critère | Seuil |
|---------|-------|
| **Domaine** | Les sujets ne partagent aucun concept (ex: finance d'entreprise ≠ photographie) |
| **Audience** | Les notes s'adressent à des contextes de vie distincts (pro vs perso) |
| **Tags** | Les deux univers n'ont aucun tag commun |
| **Taille** | Le vault dépasse 200 pages et une moitié ne cite jamais l'autre |
| **Agents** | Les sous-agents auraient besoin de contextes radicalement différents |

Conserver dans **le même vault** si :
- Les sujets partagent des concepts (ex: stratégie + négociation + gestion de projet = même vault)
- Les projets citent les mêmes patterns ou lessons
- Les tags se recoupent naturellement

> **Règle simple** : si tu dois expliquer pourquoi ces deux sujets sont dans le même vault, c'est qu'ils ne devraient pas l'être.

### Seuil technique — plafond de performance : 500 pages

**Limite absolue indépendante du domaine.** Au-delà de 500 pages, la recherche vectorielle se dégrade structurellement :
- Signal/bruit s'effondre → queries retournent des résultats non pertinents
- Hallucinations augmentent → le modèle comble les lacunes de recherche
- Coût tokens explose → plus de contexte injecté pour moins de signal utile

| Seuil | Action |
|-------|--------|
| **> 400 pages** | Alerte — lancer un lint de purge avant d'ingérer |
| **> 500 pages** | Arrêt d'ingestion — splitter ou archiver les pages obsolètes |

Ce seuil est technique, pas thématique. Un vault parfaitement cohérent peut quand même s'effondrer au-delà de 500 pages.

---

## LOG.MD — FORMAT STANDARD

```markdown
### [YYYY-MM-DD] — [AGENT] — [OPÉRATION] — [✅ | ❌ | ⚠️]

**Details**: Description de ce qui a été fait
**Files touched**:
- `wiki/[dossier]/[fichier].md` (créé | modifié | supprimé)
- `log.md` (modifié)
**Notes**: Décisions prises, observations, questions ouvertes
```

---

## GUIDE DE DÉMARRAGE

### Étape 0 — Définir ta ligne rouge (avant tout)

Prends 30 minutes. Pas de structure, pas de dossiers, pas de commandes. Juste cette question :

> **Quelle est ton exigence irréductible pour ce vault ?**
> Ce que tu refuses catégoriquement que l'IA lisse, approxime, ou ignore.

C'est le premier fichier que tu crées : `context/ligne-rouge.md`. Tout le reste en découle.

#### Exemples par métier

| Métier | Ligne rouge possible |
|--------|---------------------|
| **Comptable / Financier** | Aucune approximation sur les chiffres — chaque montant est exact ou explicitement marqué comme estimation |
| **Juriste / Avocat** | Zéro hallucination sur une loi ou jurisprudence — toute affirmation juridique cite sa source exacte |
| **Médecin / Pharmacien** | Aucune imprécision sur les dosages ou contre-indications — si incertain, signaler explicitement |
| **Ingénieur / Développeur** | Tout pattern a été testé en conditions réelles — aucune théorie non validée comme "bonne pratique" |
| **Consultant** | Transparence absolue — les décisions abandonnées sont documentées autant que les succès |
| **Journaliste / Chercheur** | Toute affirmation est sourcée — pas de fait non vérifié dans le wiki |
| **Manager / Chef de projet** | Toute décision prise est tracée avec son contexte — pas de "on a décidé ça" sans le pourquoi |

> Ta ligne rouge peut aussi être une combinaison : *"pas d'imprécision sur les chiffres + toutes les méthodes abandonnées documentées"*. L'essentiel : elle tient en 1-2 phrases et tu ne feras jamais de compromis dessus.

---

### Étape 1 — Initialiser la structure (15 min)

Une fois la ligne rouge écrite, créer les dossiers et fichiers `context/` restants. La structure découle naturellement de ce que tu viens d'écrire.

### Étape 2 — Première ingestion (10 min)

Mettre un fichier source dans `raw/` (format : `YYYY-MM-DD_description.md`) et lancer `/ingest`.
Signal de succès : ratio de compression ≥ 5:1 (30 pages source → max 6 pages wiki).

### Étape 3 — Cycle quotidien

```
[travail]            → Ingérer, requêter, capitaliser
/save (si pattern)   → Cristalliser un apprentissage avant de fermer
```

### Étape 4 — Maintenance

| Fréquence | Action |
|-----------|--------|
| Hebdomadaire | `/lint` — orphelins, liens cassés |
| Mensuelle | `/lint` complet |
| À 400 pages | Alerte — lint de purge avant toute ingestion |
| À 500 pages | Stop — splitter le vault ou archiver |

### Étape 5 — Activer les couches recommandées (passer à la puissance supérieure)

Une fois la boucle de base maîtrisée et le vault suffisamment fourni, activer les deux couches **recommandées** — dans cet ordre :

1. **ADR** (`wiki/decisions/` + `/adr`) — commence à figer le *pourquoi* de tes décisions. Le plus léger à adopter : une décision structurante qui gradue depuis tes daily, de temps en temps.
2. **Graphify** (`graphify-out/` + skill Read Graph) — quand le volume rend la relecture coûteuse, indexe le vault en graphe requêtable.

Ensemble, elles décuplent la puissance : le graphe ne répond plus seulement « qu'est-ce que je sais » mais « **pourquoi ai-je décidé ça** », et cette structure partagée ouvre l'**interopérabilité multi-agents** — plusieurs agents (local, distant) qui raisonnent depuis le même graphe plutôt que chacun sa relecture.

> **Piste d'évolution** : au-delà de la consultation, faire tourner le vault dans un **harnais agentique** — un runtime toujours actif (souvent sur un petit VPS) qui exécute les commandes, entretient le graphe et capitalise en continu, l'humain arbitrant plutôt qu'exécutant. C'est le passage du *chat* au *système agentique*. Des harnais comme [Hermès](https://composio.dev/content/openclaw-vs-hermes-agent) vont dans ce sens.

---

## CHECKLIST D'INITIALISATION

### Socle obligatoire (tout vault)

- [ ] **PREMIER** — Créer `context/ligne-rouge.md` — contrainte irréductible + ontologie fermée ← *tout le reste en découle*
- [ ] Créer la structure `raw/`, `wiki/concepts/`, `wiki/patterns/`, `wiki/lessons/`, `context/`, `daily/`
- [ ] Créer `context/scope.md` — focus, sous-domaines, hors scope + limite 500 pages
- [ ] Créer `context/identity.md` — qui tu es, ton domaine, ta cible
- [ ] Créer `context/naming-conventions.md` — liste fermée des tags autorisés
- [ ] Créer `.claude/commands/` avec les 4 commandes core (ingest, query, lint, save)
- [ ] Créer `.claude/agents/` avec les 3 agents core (ingestor, librarian, query)
- [ ] Créer `cloud.md` ≤ 500 lignes en adaptant le domaine
- [ ] Créer `index.md` avec navigation rapide et statistiques initiales
- [ ] Créer `log.md` avec la première entrée "Vault initialisé"
- [ ] Configurer `.claude/settings.local.json` (MCP, permissions)
- [ ] Nettoyer la première source et la placer dans `raw/`
- [ ] Lancer `/ingest` pour la première ingestion

### Recommandé — à activer une fois le socle maîtrisé (décuple la puissance et l'interopérabilité)

Ces deux couches ne sont pas des gadgets : une fois la boucle de base (`/ingest`, `/query`, `/lint`, `/save`) devenue un réflexe, elles font passer le vault de « base de notes » à « second cerveau requêtable et traçable ». À activer quand tu es à l'aise, pas avant.

- [ ] `wiki/decisions/` + commande `/adr` — couche **ADR** (le « pourquoi ») : trace le contexte des décisions avant qu'il ne disparaisse dans la purge des daily
- [ ] `graphify-out/` + `AGENTS.md` (skill Read Graph) — couche **Graphify** (le cerveau requêtable) : accès token-efficient et cohérence multi-agents

### Optionnel — selon l'usage

- [ ] `wiki/case-studies/` — si retours d'expérience terrain (consultant, praticien)
- [ ] `wiki/entities/` — si le vault référence des acteurs nommés (personnes, outils, entreprises)
- [ ] `wiki/sources/` — si traçabilité source critique (juriste, chercheur, audit)
- [ ] `wiki/tools/` — si le vault couvre des configurations et setups techniques
- [ ] `wiki/syntheses/` — si sessions `/query` génèrent des synthèses à conserver
- [ ] `projects/` — si initiatives à durée limitée à tracker séparément du wiki

---

## NOTES UNIVERSELLES

- **CLAUDE.md ≤ 500 lignes** : l'orchestrateur doit rester léger, le vrai contenu est dans les dossiers
- **Toujours citer les sources** : chaque page wiki pointe vers sa `wiki/sources/`
- **Dater tous les updates** : pour tracker la fraîcheur
- **Tags fermés** : jamais de tag inventé à la volée
- **Granularité progressive** : max 7-8 dossiers au démarrage, 30+ notes avant d'en ajouter un
- **Contexte minimal par agent** : chaque agent ne reçoit que ce dont il a besoin
- **Plain Markdown** : indépendance totale du modèle IA (portable vers Gemini, Codex, etc.)
- **Compounding knowledge** : le système devient exponentiellement plus utile avec le temps
- **Principe du vault** : chaque conversation utile enrichit le wiki, elle ne disparaît pas dans l'historique
- **Limite 500 pages** : plafond technique absolu — au-delà, la recherche vectorielle s'effondre, les hallucinations augmentent et le coût tokens explose. Densité > volume.
- **Ligne rouge obligatoire** : chaque vault doit avoir `context/ligne-rouge.md` — la contrainte irréductible écrite par l'humain, pas inférée par l'IA

---

## EXEMPLE DE CLOUD.MD INSTANCIÉ

```markdown
# [Nom du domaine] Brain — Schema

**Version**: 1.0
**Purpose**: Base de connaissances pour [domaine]
**Owner**: [Ton nom]
**Last Updated**: YYYY-MM-DD

## IDENTITÉ DU VAULT

**Focus**: [Domaine principal]
**Sous-domaines**: [1], [2], [3]
**Hors scope**: [Ce qu'on n'ingère pas]

## AGENTS

- **Wiki Ingestor** → `.claude/agents/ingestor.md`
- **Wiki Librarian** → `.claude/agents/librarian.md`
- **Wiki Query** → `.claude/agents/query.md`

## CONVENTIONS

Voir `context/naming-conventions.md` pour la liste complète des tags.

## COMMANDES

/ingest | /query | /lint | /save

[Renvoyer vers vault-methodology-template.md pour le détail complet]
```

---

*Version 4.1 — CC BY-NC 4.0 — Nicolas / Datashiru.*
*Inspiré de : Andrej Karpathy (mémoire LLM) + Pillitteri/Second cerveau + agricidaniel/claude-obsidian + Carpati/oubli-stratégique + Joel Parker Henderson (ADR)*
