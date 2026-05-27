# 🧠 Vault Methodology Template

**Version**: 3.5
**Auteur**: [Ton nom]
**Créé le**: YYYY-MM-DD
**Mis à jour le**: YYYY-MM-DD
**Sources**: Vault v1 + Schéma agents isolés (v2) + Pillitteri/Second cerveau (v3) + agricidaniel/claude-obsidian (v3.1) + Carpati/oubli-stratégique (v3.2)
**Usage**: Adapte ce template à ton domaine. Commence par `context/ligne-rouge.md`.

---

## 🎯 PRINCIPE FONDATEUR

**Rôle de l'IA** : Tu es l'Agent LLM Wiki. Tu es le programmeur, le wiki est ta base de code (Markdown), et Obsidian est ton IDE.

**Objectif** : Compiler la connaissance de manière persistante — pas simplement piocher des extraits (RAG classique), mais construire un savoir structuré, traçable et réutilisable qui devient plus utile exponentiellement.

**Philosophie structurelle (principe MECE)** : chaque fichier doit avoir **une place claire et une seule**. Les orphelins signalent une architecture défaillante. Si un contenu ne rentre nulle part, créer une catégorie plutôt que forcer.

La boucle d'apprentissage continue :

```
Source brute → Nettoyage → Ingestion → Wiki structuré → Query → Synthèse → Wiki enrichi
     ↑                                                                            |
     └────────────────────── Daily → Projects → Context ←───────────────────────┘
```

> ⚡ **Acte fondateur** : avant de créer la moindre structure, répondre à cette question : *quelle est mon exigence irréductible pour ce vault ?* Écrire la réponse dans `context/ligne-rouge.md`. Tout le reste — l'ontologie, le scope, les conventions — en découle. Un vault sans ligne rouge est un système sans colonne vertébrale.

---

## 🏗️ ARCHITECTURE COMPLÈTE

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
│   └── lessons/                → Erreurs, gotchas, pièges à éviter
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
│   ├── commands/               → Slash commandes (ingest, query, lint, save, bonjour, challenge)
│   ├── agents/                 → Définition des sous-agents spécialisés
│   └── settings.local.json     → Config MCP locale
│
├── cloud.md                    → Le "contrat" : règles, conventions (≤ 500 lignes)
├── index.md                    → Page d'entrée + navigation + stats du vault
└── log.md                      → Historique global de toutes les actions des agents
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

## 🤖 SOUS-AGENTS SPÉCIALISÉS

Pour éviter la saturation du contexte et réduire les coûts (~40% de tokens économisés), chaque tâche est déléguée à un **sous-agent isolé**. Chaque agent reçoit uniquement le contexte nécessaire à sa tâche.

> ⚠️ **Note pratique** : en l'absence de support natif des sous-agents dans Claude Code, chaque commande doit explicitement lister les fichiers à charger pour limiter la fenêtre de contexte.

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

### Daily Agent (`.claude/agents/daily.md`) — optionnel
- **Déclencheur** : `/bonjour`
- **Rôle** : Charge le contexte du jour — tâches ouvertes, état du vault, priorités
- **Contexte reçu** : Note du jour + note du jour précédent + `context/identity.md`
- **Output** : Note journalière créée ou ouverte + résumé 3 priorités + `log.md` mis à jour
- **Note** : la session peut aussi démarrer directement sans `/bonjour` — l'agent est un raccourci, pas une obligation

### Challenge Agent (`.claude/agents/challenge.md`)
- **Déclencheur** : `/obsidian-challenge`
- **Rôle** : Fouille le vault pour retrouver les échecs, revirements et patterns d'erreur sur un sujet
- **Contexte reçu** : La décision à challenger + `wiki/lessons/` + `wiki/case-studies/` + `projects/*/log.md`
- **Output** : Rapport de contre-arguments sourcés depuis le vault + entrée dans `log.md`

---

## 📂 GUIDE DES CATÉGORIES

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
| `context/` | Identité, conventions, scope du vault | Semi-permanente |
| `daily/` | Notes quotidiennes opérationnelles | Temporaire |
| `projects/` | Initiatives à durée limitée | Temporaire |

> 💡 **Règle de granularité** : n'ajouter un nouveau sous-dossier que si **30+ notes** justifient la catégorie. Démarrer avec le minimum, étendre à la demande.

---

## 🏷️ CONVENTIONS DE NOMMAGE

### Fichiers `wiki/`
- **Format** : `kebab-case`, descriptif, sans abréviations, sans caractères spéciaux
- **Exemples** :
  - `wiki/concepts/incremental-load-strategy.md`
  - `wiki/entities/dbt-labs.md`
  - `wiki/patterns/surrogate-key-macro.md`
  - `wiki/sources/ultimate-dbt-guide.md`
  - `wiki/syntheses/optimisation-query-bigquery.md`

### Fichiers `raw/`
- **Format** : `YYYY-MM-DD_source-description.md` — date ISO + tiret bas + description kebab-case
- **Règle** : la date est celle de récupération de la source, pas de publication
- **Exemples** :
  - `raw/2026-05-21_ultimate-dbt-guide.md`
  - `raw/2026-05-15_meeting-notes.md`
  - `raw/2026-04-10_youtube-dbt-incremental-models.md`
  - `raw/2026-03-22_article-metabase-intermediate-questions.md`

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

## 📝 STRUCTURE D'UNE PAGE WIKI

```markdown
# [Titre de la page]

**Tags**: #tag1 #tag2 #tag3
**Type**: Concept | Entity | Pattern | Case Study | Source | Synthesis | Tool | Lesson
**Created**: YYYY-MM-DD
**Last Updated**: YYYY-MM-DD
**Status**: ✅ Actif | 🔄 En révision | 📌 À vérifier | ⚠️ Obsolète

---

## 📌 TL;DR (5 lignes max)
Résumé exécutif — ce que cette page apporte en un coup d'œil.

---

## Contenu principal

### Sous-section 1
[Contenu]

### Sous-section 2
[Contenu]

---

## ⚠️ Limites et contre-exemples

- Cas où ce pattern ne s'applique pas : [contexte]
- Approche abandonnée et pourquoi : [raison]
- Signal d'alerte pour ne pas l'utiliser : [trigger]

---

## 🔗 Relations

- **Related**: [[page1]], [[page2]]
- **Part of**: [[concept-parent]]
- **Requires knowledge of**: [[prérequis]]
- **Used in**: [[case-study-ou-pattern]]
- **Source**: [[sources/nom-de-la-source]]

---

## 📊 Metrics (si applicable)

- Gain de performance : X%
- Temps d'exécution : Y sec
- Score de complexité : Z/10
```

> 💡 **Règle** : la section `## ⚠️ Limites et contre-exemples` est obligatoire pour les types Pattern et Lesson. Optionnelle pour Concept et Case Study. Un pattern sans contre-exemples est une vitrine, pas un outil de travail.

---

## 📝 STRUCTURE D'UNE NOTE DAILY

```markdown
# Daily — YYYY-MM-DD

**Status**: 🔄 En cours | ✅ Clôturée

---

## 🎯 Objectifs du jour
- [ ] Objectif 1
- [ ] Objectif 2

---

## 📋 Tâches ouvertes (report du jour précédent)
- [ ] Tâche reportée 1

---

## 📝 Décisions prises
- Décision 1 : [contexte + choix]

---

## 💡 Apprentissages du jour
- Apprentissage 1

---

## 🔁 À reporter demain
- [ ] Tâche non terminée

---

## 🔗 Pages wiki mises à jour aujourd'hui
- [[page-mise-a-jour]]
```

### Politique de rétention `daily/`

| État | Délai | Action |
|------|-------|--------|
| ✅ Clôturée, apprentissages transférés au wiki | 30 jours | Supprimer |
| ✅ Clôturée, aucun apprentissage transféré | 7 jours | Relire → transférer ou supprimer |
| 🔄 En cours (session ouverte) | — | Conserver |

> Une note daily clôturée depuis plus de 30 jours sans transfert wiki est du bruit, pas de la mémoire. Le `/purge` mensuel inclut le nettoyage de `daily/`.

---

## 📋 STATUTS DE CONTENU

| Statut | Signification |
|--------|--------------|
| ✅ **Actif** | Page à jour, fiable, utilisable immédiatement |
| 🔄 **En révision** | Contenu présent mais nécessite vérification |
| 📌 **À vérifier** | Contradictoire, incomplet ou non testé |
| ⚠️ **Obsolète** | Plus pertinent — à archiver ou supprimer |

---

## ⚙️ SLASH COMMANDES

### `/bonjour` _(optionnel)_
**But** : Ouvrir la session de travail et charger le contexte du jour.

**Processus** :
1. Créer ou ouvrir `daily/YYYY-MM-DD.md`
2. Récupérer les tâches ouvertes du jour précédent
3. Consulter les objectifs long terme dans `context/identity.md`
4. Résumer les 3 priorités de la journée
5. Logger dans `log.md`

> La session peut démarrer directement sans `/bonjour`. Cette commande est un raccourci de contexte, pas un prérequis.

---

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
5. Si des pages `⚠️ Obsolète` ou stale ont été identifiées → proposer : *"Veux-tu lancer `/purge` pour traiter les candidats ?"*
6. Logger dans `log.md`

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

### `/purge`
**But** : Retirer activement le contenu obsolète pour éviter la dégradation du signal.

> ⚠️ **Prérequis** : lancer `/lint` d'abord pour avoir un inventaire à jour des candidats.

**Processus** :
0. Appliquer la politique de rétention `daily/` :
   - Notes `✅ Clôturées` depuis > 30 jours → supprimer
   - Notes `✅ Clôturées` depuis > 7 jours sans transfert wiki visible → relire → transférer ou supprimer
1. Lister toutes les pages avec statut `⚠️ Obsolète` ou `Last Updated` > 6 mois
2. Pour chaque page candidate :
   - Vérifier si elle est citée par d'autres pages (`[[backlinks]]`)
   - Vérifier si elle a été utilisée dans une session `/query` récente (log)
   - Décision : **Mettre à jour** | **Archiver** | **Supprimer**
3. Archiver = déplacer vers `raw/YYYY-MM-DD_archived-[nom].md` (immuable, hors wiki)
4. Supprimer = uniquement si aucun backlink et aucune valeur historique
5. Mettre à jour `index.md` + logger dans `log.md`

**Garde-fous** :
- ❌ Ne jamais supprimer sans confirmation explicite
- ❌ Ne pas purger une page avec des backlinks actifs — mettre à jour d'abord
- ❌ Ne pas purger plus de 10% du vault en une session — risque de perte de cohérence
- ❌ Toujours archiver avant de supprimer si la valeur historique est incertaine

**Fréquence** : Au lint mensuel ou quand le vault dépasse 400 pages.

---

### `/obsidian-challenge [décision]`
**But** : Confronter une décision avec les échecs, revirements et patterns d'erreur déjà documentés dans le vault.

**Processus** :
1. Identifier le domaine de la décision (technique, client, process, organisationnel...)
2. Chercher dans :
   - `wiki/lessons/` → erreurs et gotchas documentés
   - `wiki/case-studies/` → projets passés sur des sujets proches
   - `projects/*/log.md` → décisions prises et leurs conséquences
   - `daily/` → uniquement les 30 derniers jours (filtrer par date de fichier `YYYY-MM-DD`)
3. Extraire : décisions similaires passées, erreurs commises, patterns d'échec récurrents
4. Formuler des objections en citant **exactement** les pages du vault (`[[page]]`, verbatim si possible)
5. Produire un rapport structuré :
   - **Ce que le vault dit sur ce sujet**
   - **Décisions similaires passées et leur résultat**
   - **Risques identifiés depuis ton propre vécu**
   - **Questions à te poser avant de décider**
6. Logger dans `log.md`

**Garde-fous** :
- ❌ Ne pas inventer de contre-arguments — uniquement ce qui est dans le vault
- ❌ Si le vault est vide sur le sujet → signaler explicitement "Pas d'historique sur ce sujet dans le vault"
- ❌ Ne pas prendre position — présenter les faits du vault, pas une opinion
- ❌ Ne pas modifier le vault pendant cette commande

---

## 🔀 RÈGLE DE SPLIT VAULT

Un vault = un domaine cohérent. Deux seuils distincts à surveiller :

### Seuil thématique — signal de divergence de domaine

Créer un **nouveau vault séparé** si :

| Critère | Seuil |
|---------|-------|
| **Domaine** | Les sujets ne partagent aucun concept (ex: analytics engineering ≠ photographie) |
| **Audience** | Les notes s'adressent à des contextes de vie distincts (pro vs perso) |
| **Tags** | Les deux univers n'ont aucun tag commun |
| **Taille** | Le vault dépasse 200 pages et une moitié ne cite jamais l'autre |
| **Agents** | Les sous-agents auraient besoin de contextes radicalement différents |

Conserver dans **le même vault** si :
- Les sujets partagent des concepts (ex: SQL + dbt + BigQuery = même vault)
- Les projets citent les mêmes patterns ou lessons
- Les tags se recoupent naturellement

> 💡 **Règle simple** : si tu dois expliquer pourquoi ces deux sujets sont dans le même vault, c'est qu'ils ne devraient pas l'être.

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

## 📅 LOG.MD — FORMAT STANDARD

```markdown
### [YYYY-MM-DD] — [AGENT] — [OPÉRATION] — [✅ | ❌ | ⚠️]

**Details**: Description de ce qui a été fait
**Files touched**:
- `wiki/[dossier]/[fichier].md` (créé | modifié | supprimé)
- `log.md` (modifié)
**Notes**: Décisions prises, observations, questions ouvertes
```

---

## 🗺️ GUIDE DE DÉMARRAGE

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
| **Analytics Engineer** | SQL propre, efficace, élégant — aucun pattern "ça marche mais c'est sale" n'entre dans le wiki |
| **Consultant** | Transparence absolue — les décisions abandonnées sont documentées autant que les succès |
| **Journaliste / Chercheur** | Toute affirmation est sourcée — pas de fait non vérifié dans le wiki |
| **Développeur** | Tout pattern a été testé en production — pas de théorie non validée comme "bonne pratique" |
| **Manager / Chef de projet** | Toute décision prise est tracée avec son contexte — pas de "on a décidé ça" sans le pourquoi |

> 💡 Ta ligne rouge peut aussi être une combinaison : *"pas d'imprécision sur les chiffres + toutes les méthodes abandonnées documentées"*. L'essentiel : elle tient en 1-2 phrases et tu ne feras jamais de compromis dessus.

---

### Étape 1 — Initialiser la structure (15 min)

Une fois la ligne rouge écrite, créer les dossiers et fichiers `context/` restants. La structure découle naturellement de ce que tu viens d'écrire.

### Étape 2 — Première ingestion (10 min)

Mettre un fichier source dans `raw/` (format : `YYYY-MM-DD_description.md`) et lancer `/ingest`.
Signal de succès : ratio de compression ≥ 5:1 (30 pages source → max 6 pages wiki).

### Étape 3 — Cycle quotidien

```
/bonjour (optionnel) → Charger le contexte du jour, tâches ouvertes
[travail]            → Ingérer, requêter, capitaliser
/save (si pattern)   → Cristalliser un apprentissage avant de fermer
```

### Étape 4 — Maintenance

| Fréquence | Action |
|-----------|--------|
| Hebdomadaire | `/lint` — orphelins, liens cassés |
| Mensuelle | `/lint` complet + `/purge` si pages obsolètes |
| À 400 pages | Alerte — lint de purge avant toute ingestion |
| À 500 pages | Stop — splitter le vault ou archiver |

---

## 🚀 CHECKLIST D'INITIALISATION

### Socle obligatoire (tout vault)

- [ ] **PREMIER** — Créer `context/ligne-rouge.md` — contrainte irréductible + ontologie fermée ← *tout le reste en découle*
- [ ] Créer la structure `raw/`, `wiki/concepts/`, `wiki/patterns/`, `wiki/lessons/`, `context/`, `daily/`
- [ ] Créer `context/scope.md` — focus, sous-domaines, hors scope + limite 500 pages
- [ ] Créer `context/identity.md` — qui tu es, ton domaine, ta cible
- [ ] Créer `context/naming-conventions.md` — liste fermée des tags autorisés
- [ ] Créer `.claude/commands/` avec les 5 commandes core (bonjour, ingest, query, lint, save)
- [ ] Créer `.claude/agents/` avec les 4 agents core (ingestor, librarian, query, daily)
- [ ] Créer `cloud.md` ≤ 500 lignes en adaptant le domaine
- [ ] Créer `index.md` avec navigation rapide et statistiques initiales
- [ ] Créer `log.md` avec la première entrée "Vault initialisé"
- [ ] Configurer `.claude/settings.local.json` (MCP, permissions)
- [ ] Nettoyer la première source et la placer dans `raw/`
- [ ] Lancer `/bonjour` pour la première session
- [ ] Lancer `/ingest` pour la première ingestion

### Optionnel — selon l'usage

- [ ] `wiki/case-studies/` — si retours d'expérience terrain (consultant, praticien)
- [ ] `wiki/entities/` — si le vault référence des acteurs nommés (personnes, outils, entreprises)
- [ ] `wiki/sources/` — si traçabilité source critique (juriste, chercheur, audit)
- [ ] `wiki/tools/` — si le vault couvre des configurations et setups techniques
- [ ] `wiki/syntheses/` — si sessions `/query` génèrent des synthèses à conserver
- [ ] `projects/` — si initiatives à durée limitée à tracker séparément du wiki
- [ ] `/bonjour` + Daily Agent — si chargement de contexte quotidien souhaité
- [ ] `/obsidian-challenge` + Challenge Agent — si vault avec historique d'échecs documentés

---

## 🔑 NOTES UNIVERSELLES

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

## 📌 EXEMPLE DE CLOUD.MD INSTANCIÉ

```markdown
# 🧠 [Nom du domaine] Brain — Schema

**Version**: 1.0
**Purpose**: Base de connaissances pour [domaine]
**Owner**: [Ton nom]
**Last Updated**: YYYY-MM-DD

## 🎯 IDENTITÉ DU VAULT

**Focus**: [Domaine principal]
**Sous-domaines**: [1], [2], [3]
**Hors scope**: [Ce qu'on n'ingère pas]

## 🤖 AGENTS

- **Wiki Ingestor** → `.claude/agents/ingestor.md`
- **Wiki Librarian** → `.claude/agents/librarian.md`
- **Wiki Query** → `.claude/agents/query.md`
- **Daily Agent** → `.claude/agents/daily.md`
- **Challenge Agent** → `.claude/agents/challenge.md`

## 📋 CONVENTIONS

Voir `context/naming-conventions.md` pour la liste complète des tags.

## ⚙️ COMMANDES

/bonjour | /ingest | /query | /lint | /save | /obsidian-challenge

[Renvoyer vers vault-methodology-template.md pour le détail complet]
```

---

*Version 3.5 — CC BY-NC 4.0 — [Ton nom]*
*Basé sur : Pillitteri/Second cerveau + agricidaniel/claude-obsidian + Carpati/oubli-stratégique*
