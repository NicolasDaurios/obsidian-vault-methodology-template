# Vault Methodology Template

*[Version française](vault-methodology-template.md)*

**Version**: 4.1
**Author**: Nicolas / Datashiru
**Created**: 2026-04-24
**Updated**: 2026-07-20
**Inspiration**: Andrej Karpathy (LLM memory) + Pillitteri/Second Brain + agricidaniel/claude-obsidian + Carpati/strategic-forgetting + Joel Parker Henderson (ADR)
**License**: [CC BY-NC 4.0](LICENSE) — Nicolas / Datashiru.
**Usage**: Adapt this template to your domain. Start with `context/ligne-rouge.md` (your "Red Line").

---

## FOUNDING PRINCIPLE

**Role of the AI**: You are the Wiki LLM Agent. You are the programmer, the wiki is your codebase (Markdown), and Obsidian is your IDE.

**Objective**: Compile knowledge persistently — not merely retrieving snippets (classic RAG), but building structured, traceable, reusable knowledge that becomes exponentially more useful.

**Structural philosophy (MECE principle)**: every file must have **one clear place, and only one**. Orphans signal a broken architecture. If content doesn't fit anywhere, create a category rather than forcing it in.

The continuous learning loop:

```
Raw source → Cleaning → Ingestion → Structured wiki → Query → Synthesis → Enriched wiki
     ↑                                                                            |
     └────────────────────── Daily → Projects → Context ←───────────────────────┘
```

> **Founding act**: before creating any structure at all, answer this question: *what is my non-negotiable requirement for this vault?* Write the answer in `context/ligne-rouge.md` ("Red Line"). Everything else — ontology, scope, conventions — flows from it. A vault without a Red Line is a system without a spine.

---

## FULL ARCHITECTURE

```
[VAULT ROOT]
│
├── raw/                        → Immutable raw sources (articles, docs, transcripts, PDF→MD)
│
├── wiki/                       → Permanent knowledge compiled by the agents
│   ├── concepts/               → Abstract ideas, mental models, theories, frameworks
│   ├── entities/               → People, companies, projects, specific technologies
│   ├── patterns/                → Reusable templates, recipes, step-by-step methods
│   ├── case-studies/           → Real stories, concrete projects, lessons from the field
│   ├── sources/                → Metadata and summaries of files present in raw/
│   ├── syntheses/              → Complex syntheses generated during /query sessions
│   ├── tools/                  → Setup, configurations, tool integrations
│   ├── lessons/                → Errors, gotchas, pitfalls to avoid
│   └── decisions/              → recommended — decision log (ADR): the "why" (see below)
│
├── context/                    → The vault's identity (who you are, your domain, your conventions)
│   ├── identity.md             → Your positioning, values, mission, target audience
│   ├── naming-conventions.md   → CLOSED list of allowed tags + naming rules
│   └── scope.md                → Vault's domain, sub-domains, out of scope
│
├── daily/                      → Daily notes (operational, non-permanent)
│   └── YYYY-MM-DD.md           → Today's goals, decisions, open tasks
│
├── projects/                   → Time-boxed initiatives (≠ permanent knowledge)
│   └── [project-name]/
│       ├── brief.md
│       └── log.md
│
├── .claude/
│   ├── commands/               → Slash commands (ingest, query, lint, save)
│   ├── agents/                 → Specialized sub-agent definitions
│   └── settings.local.json     → Local MCP config
│
├── graphify-out/                → recommended — queryable knowledge graph (see below)
│   ├── graph.json
│   ├── manifest.json
│   └── cache/                   → exclude from version control (regenerable)
│
├── cloud.md                    → The "contract": rules, conventions (≤ 500 lines)
├── index.md                    → Entry page + navigation + vault stats
├── log.md                      → Global history of all agent actions
└── AGENTS.md                   → optional — multi-agent protocol (see below)
```

### Layer rules

| Layer | Editable? | By whom? | Role |
|-------|-----------|----------|------|
| `raw/` | ❌ Never | — | Immutable archive of sources |
| `wiki/` | ✅ Yes | Agents | Permanent knowledge |
| `context/` | ✅ Yes | Human | Vault identity and conventions |
| `daily/` | ✅ Yes | Human + Agents | Daily operational notes (retention: 30 days after closing) |
| `projects/` | ✅ Yes | Human + Agents | Active, time-boxed projects |
| `.claude/` | ✅ Yes | Human | Behaviors and agents |
| `cloud.md` | ✅ Yes | Human | Contract & conventions (≤ 500 lines) |

---

## SPECIALIZED SUB-AGENTS

To avoid context saturation and reduce costs (~40% token savings), each task is delegated to an **isolated sub-agent**. Each agent receives only the context it needs for its task.

> **Practical note**: in the absence of native sub-agent support in Claude Code, each command must explicitly list which files to load to keep the context window bounded.

### Wiki Ingestor (`.claude/agents/ingestor.md`)
- **Trigger**: `/ingest`
- **Role**: Reads `raw/`, strips distractors, extracts salient facts, creates wiki pages
- **Context received**: Source file + `cloud.md` + `context/naming-conventions.md`
- **Output**: Pages created/updated in `wiki/` + an entry in `log.md`

### Wiki Librarian (`.claude/agents/librarian.md`)
- **Trigger**: `/lint`
- **Role**: Indexing, link creation, overall vault health
- **Context received**: List of all wiki pages + `cloud.md`
- **Output**: Health report + proposed fixes + updated `log.md`

### Wiki Query (`.claude/agents/query.md`)
- **Trigger**: `/query`
- **Role**: Queries the base and returns only the final answer
- **Context received**: The question + only the relevant wiki pages (not the whole vault)
- **Output**: Sourced answer with `[[links]]` + offer to `/save`

---

## KNOWLEDGE GRAPH LAYER (GRAPHIFY) — recommended

Past a certain volume, an agent that has to reread the entire wiki to answer a question becomes slow and token-expensive. [Graphify](https://github.com/Graphify-Labs/graphify) solves this by indexing the vault into a queryable graph (`graphify-out/graph.json`): nodes (pages, concepts) + edges (explicit and inferred semantic relations) + detected communities.

### "Read Graph" skill

A 3-step protocol, to be documented in an `AGENTS.md` at the root (a neutral file, readable by any agent — Claude Code, GPT, or an agent running on another server — not just `CLAUDE.md`, which is Claude-Code-specific):

1. **Consult the graph first** — read `graphify-out/graph.json` before scanning Markdown files.
2. **Targeted file identification** — spot relevant paths and metadata from the graph, without opening every document.
3. **Selective final reading** — only open a file's full content when strictly necessary.

**Why**: this technique reduces token consumption compared to reading through every file directly. And if several agents follow the same protocol on the same graph, they answer from the same structure — multi-agent consistency instead of each agent rereading the vault its own way.

### Multi-agent and distribution

If several agents need to consult the same vault from different environments (e.g., a local agent + an agent on a VPS), prefer a **read-only Git sync** (the remote agent does a `git pull`, never a direct push) over exposing the vault over the network — this avoids exposing a personal machine to the network and avoids multi-writer merge conflicts. The vault stays managed by a single write point (the human + their main agent); feedback from a remote agent is surfaced and integrated manually, never auto-merged.

> **Reference implementation (author's own)**: the architecture is built around **Claude Code** (main agent, write access) + **Hermès** (agentic harness) running on a **VPS** as a background process, with the vault synced via read-only Git. This is just one possible instantiation — the method, being plain Markdown, applies to many situations: a personal knowledge base, a team or organization wiki, project documentation, market watch, client work, research.

### Git hygiene (lesson learned)

`graphify-out/cache/` (extraction cache, regenerable) and the dated backup folders created before each re-clustering run must **never** be committed — they bloat git history without adding value for a consuming agent, which only needs `graph.json`/`manifest.json`. Add to `.gitignore`:

```
graphify-out/cache/
graphify-out/20[0-9][0-9]-[0-9][0-9]-[0-9][0-9]/
```

By default, Graphify also indexes local config files (`.obsidian/*.json`, `.claude/settings.local.json`) if they aren't already excluded by `.gitignore` (Graphify relies on it for its own exclusions) — this creates isolated nodes that pollute the graph without connecting to the rest. Excluding them from `.gitignore` before the first run avoids having to purge them afterward.

---

## DECISION LAYER (ADR) — recommended

Structuring decisions are currently captured in `daily/` (the "Decisions made" section) — but `daily/` is purged after 30 days. The result: the **why** behind a choice is ephemeral by construction, even though it's often the most expensive piece of information to reconstruct six months later. An [ADR — Architecture Decision Record](https://github.com/joelparkerhenderson/architecture-decision-record) is a page that fixes this why in place: the context, the options considered, the choice made, and its consequences.

This isn't reserved for engineering: a doctor traces *why this protocol rather than another*, a lawyer *why this strategy*, a consultant *why this recommendation*. The mechanism is universal; only the domain changes.

### Boundary with `lessons/` and "counter-examples" (MECE rule)

The "one clear place, and only one" principle requires a sharp distinction between:

| Type | Nature | Question it answers |
|------|--------|----------------------|
| **Decision** (`wiki/decisions/`) | **Prospective** choice between alternatives | "We're going with X over Y — here's why" |
| **Lesson** (`wiki/lessons/`) | **Retrospective** finding | "X broke — here's the pitfall to avoid" |
| "Limits and counter-examples" | Section **within** an existing page | "This pattern doesn't apply when…" |

> **Quick test**: were there several possible routes to choose from at time T? → **Decision**. Did I discover a pitfall while moving forward? → **Lesson**.

### Structure of a decision page (ADR)

A combination of the Nygard template (the simplest) and MADR's "options" section:

```markdown
---
tags: [tag1, tag2]
type: Decision
created: YYYY-MM-DD
decision_date: YYYY-MM-DD
status: proposed | accepted | superseded | deprecated
supersedes: "[[decisions/old-decision]]"      # if it replaces one
superseded_by: "[[decisions/new-decision]]"   # if it has been replaced
related: ["[[relevant-page]]"]
source: "[[sources/source-name]]"
---

# [Verb + object — e.g. "Choose Postgres over Mongo"]

## Context
The force that triggered the decision — the problem, the constraint, the deadline.

## Options considered
- **Option A** — [description] → chosen / rejected because [reason]
- **Option B** — [description] → rejected because [reason]
- **Option C (status quo)** → rejected because [reason]

## Decision
What was chosen and the rationale — why this route and not the others.

## Consequences
What we're accepting in exchange — benefits AND costs. The trade-offs we're owning.
```

> **Rule**: the `## Options considered` section is **mandatory**. An ADR with no rejected alternative isn't a decision, it's an announcement — it loses the whole point (the "*rather than another*").

### Status and mutability

Decisions have their **own status vocabulary**, distinct from wiki statuses:

| Status | Meaning |
|--------|---------|
| `proposed` | Under consideration, not yet enacted |
| `accepted` | Decision currently in effect |
| `superseded` | Revised by a more recent ADR (`superseded_by`) |
| `deprecated` | No longer relevant, with no replacement |

Two ways a decision can evolve:

- **Reversal** (changing the choice) → create a **new** page that `supersedes` the old one, and flip the old one to `superseded`. This keeps the trace of *why you changed your mind* — that's the heart of the memory.
- **Clarification** (refining without changing the choice) → **dated annotation** on the existing page. No need to supersede.

> Never silently rewrite an `accepted` decision to reverse it: without a trace of the reversal, amnesia creeps back in through the window. (The reference repo embraces this pragmatic mutability — supersession for a genuine reversal, dated annotation for everything else; no strict immutability.)

### Naming

Like the reference repo: **descriptive and imperative**, kebab-case, no numbering (`[[choose-postgres]]` reads; `[[adr-0007]]` doesn't):

- `wiki/decisions/choose-postgres-over-mongo.md`
- `wiki/decisions/drop-the-redis-cache.md`

A numeric prefix (`0001-`) only if you want a visible chronological order — optional.

### The `daily → ADR` flow (the mechanism that keeps the module alive)

An ADR isn't written "in addition to" the work — it **graduates** from the daily log:

```
Decision noted in daily/ ("Decisions made")
        │
        ▼   if it is structuring AND durable
Page created in wiki/decisions/ before the 30-day purge
        │
        ▼
Indexed by Graphify → queryable ("why did we choose X?")
```

Without this graduation reflex, the module is dead: decisions stay in `daily/` and vanish. **Rule: when closing a daily note, any decision that will still matter beyond this month goes into `wiki/decisions/`.**

---

## CATEGORY GUIDE

| Folder | Content | Lifespan |
|--------|---------|----------|
| `wiki/concepts/` | Theoretical notions, frameworks, architectures | Permanent |
| `wiki/entities/` | People, companies, projects, named tools | Permanent |
| `wiki/patterns/` | Templates, recipes, step-by-step methods | Permanent |
| `wiki/case-studies/` | Real projects, lessons from the field | Permanent |
| `wiki/sources/` | Metadata for `raw/` files | Permanent |
| `wiki/syntheses/` | Syntheses from `/query` sessions | Permanent |
| `wiki/tools/` | Setup, configurations, integrations | Permanent |
| `wiki/lessons/` | Errors, gotchas, pitfalls to avoid | Permanent |
| `wiki/decisions/` | Traced decisions (ADR): choice + rejected options + consequences | Permanent |
| `context/` | Vault identity, conventions, scope | Semi-permanent |
| `daily/` | Operational daily notes | Temporary |
| `projects/` | Time-boxed initiatives | Temporary |

> **Granularity rule**: only add a new sub-folder once **30+ notes** justify the category. Start with the minimum, extend on demand.

---

## NAMING CONVENTIONS

### `wiki/` files
- **Format**: `kebab-case`, descriptive, no abbreviations, no special characters
- **Examples**:
  - `wiki/concepts/feedback-loop-model.md`
  - `wiki/entities/tool-or-person-name.md`
  - `wiki/patterns/decision-making-template.md`
  - `wiki/sources/book-or-article-name.md`
  - `wiki/syntheses/synthesis-on-key-topic.md`
  - `wiki/decisions/choose-x-over-y.md`

### `raw/` files
- **Format**: `YYYY-MM-DD_source-description.md` — ISO date + underscore + kebab-case description
- **Rule**: the date is when the source was retrieved, not when it was published
- **Examples**:
  - `raw/2026-05-21_reference-book-name.md`
  - `raw/2026-05-15_project-meeting-notes.md`
  - `raw/2026-04-10_youtube-video-title.md`
  - `raw/2026-03-22_key-topic-article.md`

### `daily/` files
- **Format**: `YYYY-MM-DD.md`

### Tags
- The list of allowed tags is **closed** and defined in `context/naming-conventions.md`
- Never invent a new tag on the fly — add it to the list first
- Format: `#kebab-case`

### Internal Obsidian links
- `[[page-name]]` → without the `.md` extension
- `[[folder/page-name]]` → with the path if there's ambiguity
- Always weave links between related pages

### Source Markdown quality (`raw/`)
Before ingestion, make sure the Markdown is **clean**:
- Ads, menus, footers, stray HTML elements removed
- OCR or PDF conversion done with a quality tool (e.g. Mistral Document AI)
- Dirty Markdown reduces ingestion performance by **20 to 60%**

---

## STRUCTURE OF A WIKI PAGE

```markdown
---
tags: [tag1, tag2, tag3]
type: Concept | Entity | Pattern | Case Study | Source | Synthesis | Tool | Lesson | Decision
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
status: active | under-revision | to-verify | obsolete
related: ["[[page1]]", "[[page2]]"]
part_of: "[[parent-concept]]"
requires: ["[[prerequisite]]"]
used_in: ["[[case-study-or-pattern]]"]
source: "[[sources/source-name]]"
---

# [Page title]

## TL;DR (5 lines max)
Executive summary — what this page delivers at a glance.

---

## Main content

### Sub-section 1
[Content]

### Sub-section 2
[Content]

---

## Limits and counter-examples

- Case where this pattern doesn't apply: [context]
- Approach abandoned and why: [reason]
- Warning sign not to use it: [trigger]

---

## Metrics (if applicable)

- Performance gain: X%
- Execution time: Y sec
- Complexity score: Z/10
```

> **Rule**: the `## Limits and counter-examples` section is mandatory for the Pattern and Lesson types. Optional for Concept and Case Study. A pattern with no counter-examples is a showcase, not a working tool.

---

## STRUCTURE OF A DAILY NOTE

```markdown
---
status: in-progress | closed
---

# Daily — YYYY-MM-DD

## Today's goals
- [ ] Goal 1
- [ ] Goal 2

---

## Open tasks (carried over from the previous day)
- [ ] Carried-over task 1

---

## Decisions made
- Decision 1: [context + choice]

---

## Today's learnings
- Learning 1

---

## To carry over tomorrow
- [ ] Unfinished task

---

## Wiki pages updated today
- [[updated-page]]
```

### `daily/` retention policy

| State | Delay | Action |
|-------|-------|--------|
| ✅ Closed, learnings transferred to the wiki | 30 days | Delete |
| ✅ Closed, no learnings transferred | 7 days | Review → transfer or delete |
| 🔄 In progress (open session) | — | Keep |

> A daily note closed for more than 30 days with no wiki transfer is noise, not memory.

---

## CONTENT STATUSES

| YAML value | Meaning |
|------------|---------|
| `active` | Up to date, reliable, ready to use immediately |
| `under-revision` | Content present but needs checking |
| `to-verify` | Contradictory, incomplete, or untested |
| `obsolete` | No longer relevant — to archive or delete |

---

## SLASH COMMANDS

### `/ingest [file]`
**Purpose**: Turn a raw source (`raw/`) into wiki pages via the **Wiki Ingestor**.

**Process**:
1. Verify the source Markdown is clean
2. Verify the file follows the `YYYY-MM-DD_description.md` convention — flag it if not
3. Read the source file in `raw/`
4. Identify: concepts, entities, patterns, lessons, use cases
5. Create or update the corresponding wiki pages
6. If `wiki/sources/` exists → create `wiki/sources/[source-name].md` with the metadata
7. Add links between the new pages and existing ones
8. Log to `log.md`
9. Confirm: "✅ Ingestion complete: X pages created, Y pages updated — compression ratio: Z:1"

**Guardrails**:
- ❌ Never modify `raw/`
- ❌ Check for duplicates before creating anything
- ❌ Never create a page without complete frontmatter
- ❌ Only use tags from `context/naming-conventions.md`
- ⚠️ Over-ingestion signal: if pages created > 20% of the source volume, check the granularity — the ingestor compiles, it doesn't copy

---

### `/query [question]`
**Purpose**: Answer from the wiki via the **Wiki Query**, not from the model's general knowledge.

**Process**:
1. Identify the key concepts
2. Load only the relevant wiki pages
3. Assemble a sourced answer with `[[links]]`
4. Offer: "Want me to add this summary to the wiki?"
5. Log to `log.md`

**Guardrails**:
- ❌ Never make things up if the wiki is empty on the topic
- ❌ Never load the whole vault — only the relevant pages
- ❌ Always search the wiki before falling back to general knowledge

---

### `/lint`
**Purpose**: Check vault health via the **Wiki Librarian**.

**Process**:
1. Scan all of `wiki/` and its sub-folders
2. Detect:
   - ❌ **Orphans**: pages with no inbound link
   - ❌ **Duplicates**: two pages on the same concept
   - ❌ **Contradictions**: conflicting information
   - ❌ **Formatting**: missing or incomplete frontmatter
   - ❌ **Broken links**: a `[[dead-link]]` that doesn't exist
   - ❌ **Stale content**: pages not updated in 6+ months
   - ❌ **Missing examples**: a pattern with no concrete example
   - ❌ **Out-of-list tags**: tags not defined in `context/naming-conventions.md`
   - ❌ **Unlinked sources**: pages with no `[[sources/...]]`
3. Produce a report by category
4. Propose fixes
5. Log to `log.md`

**Frequency**:
- Weekly: orphans + broken links + out-of-list tags
- Monthly: full lint

**Guardrails**:
- ❌ Never delete without confirmation
- ❌ Never auto-merge — propose first
- ❌ Only touch formatting, never substantive content

---

### `/save [title]`
**Purpose**: Turn an answer into a permanent wiki page.

**Process**:
1. Retrieve the answer to save
2. Determine the type and destination folder
3. Check there's no duplicate
4. Create the page with complete frontmatter + backlinks + source link
5. Log to `log.md`
6. Confirm: "✅ Page created: wiki/[folder]/[title].md"

**Guardrails**:
- ❌ Reformat cleanly, never copy-paste raw
- ❌ Always include frontmatter + backlinks
- ❌ Never save an incomplete or low-quality answer
- ❌ Never save if a duplicate already exists

---

### `/adr [title]` — optional (decision layer)

**Purpose**: Capture a decision in ADR format inside `wiki/decisions/`.

**Process**:
1. Retrieve the decision (from the current daily note or the question asked)
2. Reconstruct: context, options considered (including rejected ones), choice, consequences
3. Create `wiki/decisions/[verb-object].md` with the full decision frontmatter
4. If it replaces another one → fill in `supersedes` / `superseded_by` and flip the old one to `superseded`
5. Link to the relevant pages (the tool, concept, or pattern the decision justifies)
6. Log to `log.md`

**Guardrails**:
- ❌ Refuse to create an ADR with no rejected option at all — otherwise it isn't a decision
- ❌ Never rewrite an `accepted` decision to reverse it → supersede it instead
- ❌ Keep Decision (prospective choice) distinct from Lesson (retrospective finding)

---

## VAULT SPLIT RULE

One vault = one coherent domain. Two distinct thresholds to watch:

### Thematic threshold — signal of domain divergence

Create a **new, separate vault** if:

| Criterion | Threshold |
|-----------|-----------|
| **Domain** | The topics share no concepts (e.g., corporate finance ≠ photography) |
| **Audience** | The notes serve distinct life contexts (professional vs. personal) |
| **Tags** | The two worlds have no tag in common |
| **Size** | The vault exceeds 200 pages and one half never references the other |
| **Agents** | The sub-agents would need radically different contexts |

Keep it in **the same vault** if:
- The topics share concepts (e.g., strategy + negotiation + project management = same vault)
- Projects cite the same patterns or lessons
- Tags naturally overlap

> **Simple rule**: if you have to explain why these two topics are in the same vault, they probably shouldn't be.

### Technical threshold — performance ceiling: 500 pages

**An absolute limit, independent of domain.** Beyond 500 pages, vector search degrades structurally:
- Signal/noise collapses → queries return irrelevant results
- Hallucinations increase → the model fills in the search gaps
- Token cost explodes → more context injected for less useful signal

| Threshold | Action |
|-----------|--------|
| **> 400 pages** | Alert — run a purge lint before ingesting anything else |
| **> 500 pages** | Stop ingesting — split the vault or archive stale pages |

This threshold is technical, not thematic. A perfectly coherent vault can still collapse past 500 pages.

---

## LOG.MD — STANDARD FORMAT

```markdown
### [YYYY-MM-DD] — [AGENT] — [OPERATION] — [✅ | ❌ | ⚠️]

**Details**: Description of what was done
**Files touched**:
- `wiki/[folder]/[file].md` (created | modified | deleted)
- `log.md` (modified)
**Notes**: Decisions made, observations, open questions
```

---

## GETTING STARTED GUIDE

### Step 0 — Define your Red Line (before anything else)

Take 30 minutes. No structure, no folders, no commands. Just this question:

> **What is your non-negotiable requirement for this vault?**
> What you categorically refuse to let the AI smooth over, approximate, or ignore.

This is the first file you create: `context/ligne-rouge.md`. Everything else flows from it.

#### Examples by profession

| Profession | Possible Red Line |
|------------|--------------------|
| **Accountant / Finance** | No approximation on figures — every amount is exact or explicitly marked as an estimate |
| **Lawyer** | Zero hallucination on a law or ruling — every legal claim cites its exact source |
| **Doctor / Pharmacist** | No imprecision on dosages or contraindications — if uncertain, flag it explicitly |
| **Engineer / Developer** | Every pattern has been tested under real conditions — no unvalidated theory passed off as "best practice" |
| **Consultant** | Absolute transparency — abandoned decisions are documented as thoroughly as successes |
| **Journalist / Researcher** | Every claim is sourced — no unverified fact in the wiki |
| **Manager / Project lead** | Every decision made is traced with its context — no "we decided that" without the why |

> Your Red Line can also be a combination: *"no imprecision on figures + every abandoned method documented"*. What matters: it fits in 1-2 sentences, and you will never compromise on it.

---

### Step 1 — Initialize the structure (15 min)

Once the Red Line is written, create the remaining `context/` folders and files. The structure naturally follows from what you just wrote.

### Step 2 — First ingestion (10 min)

Drop a source file into `raw/` (format: `YYYY-MM-DD_description.md`) and run `/ingest`.
Signal of success: compression ratio ≥ 5:1 (30 pages of source → 6 pages of wiki, max).

### Step 3 — Daily cycle

```
[work]                → Ingest, query, capitalize
/save (if a pattern)  → Crystallize a learning before closing out
```

### Step 4 — Maintenance

| Frequency | Action |
|-----------|--------|
| Weekly | `/lint` — orphans, broken links |
| Monthly | Full `/lint` |
| At 400 pages | Alert — purge lint before any more ingestion |
| At 500 pages | Stop — split the vault or archive |

### Step 5 — Activate the recommended layers (leveling up)

Once the base loop is second nature and the vault is well-fed enough, activate the two **recommended** layers — in this order:

1. **ADR** (`wiki/decisions/` + `/adr`) — start fixing the *why* behind your decisions in place. The lightest way in: graduate one structuring decision from your daily notes, every so often.
2. **Graphify** (`graphify-out/` + the Read Graph skill) — once volume makes rereading everything expensive, index the vault into a queryable graph.

Together, they multiply the vault's power: the graph no longer just answers "what do I know" but "**why did I decide this**", and this shared structure opens up **multi-agent interoperability** — several agents (local, remote) reasoning from the same graph instead of each one rereading the vault its own way.

> **A path forward**: beyond mere consultation, run the vault inside an **agentic harness** — an always-on runtime (often on a small VPS) that executes commands, maintains the graph, and keeps capitalizing continuously, with the human arbitrating rather than executing. This is the shift from *chat* to *agentic system*. Harnesses like [Hermès](https://composio.dev/content/openclaw-vs-hermes-agent) point in this direction.

---

## INITIALIZATION CHECKLIST

### Mandatory foundation (every vault)

- [ ] **FIRST** — Create `context/ligne-rouge.md` — non-negotiable constraint + closed ontology ← *everything else flows from this*
- [ ] Create the `raw/`, `wiki/concepts/`, `wiki/patterns/`, `wiki/lessons/`, `context/`, `daily/` structure
- [ ] Create `context/scope.md` — focus, sub-domains, out of scope + the 500-page limit
- [ ] Create `context/identity.md` — who you are, your domain, your target audience
- [ ] Create `context/naming-conventions.md` — closed list of allowed tags
- [ ] Create `.claude/commands/` with the 4 core commands (ingest, query, lint, save)
- [ ] Create `.claude/agents/` with the 3 core agents (ingestor, librarian, query)
- [ ] Create `cloud.md`, ≤ 500 lines, adapted to your domain
- [ ] Create `index.md` with quick navigation and initial stats
- [ ] Create `log.md` with the first entry, "Vault initialized"
- [ ] Configure `.claude/settings.local.json` (MCP, permissions)
- [ ] Clean up your first source and place it in `raw/`
- [ ] Run `/ingest` for the first ingestion

### Recommended — activate once the foundation is solid (multiplies power and interoperability)

These two layers aren't gadgets: once the base loop (`/ingest`, `/query`, `/lint`, `/save`) has become second nature, they take the vault from "note base" to "queryable, traceable second brain." Activate once you're comfortable, not before.

- [ ] `wiki/decisions/` + the `/adr` command — the **ADR** layer (the "why"): traces the context behind decisions before it disappears in the daily-notes purge
- [ ] `graphify-out/` + `AGENTS.md` (Read Graph skill) — the **Graphify** layer (the queryable graph layer): token-efficient access and multi-agent consistency

### Optional — depending on usage

- [ ] `wiki/case-studies/` — if you have real-world experience reports (consultant, practitioner)
- [ ] `wiki/entities/` — if the vault references named actors (people, tools, companies)
- [ ] `wiki/sources/` — if critical source traceability matters (lawyer, researcher, auditor)
- [ ] `wiki/tools/` — if the vault covers configurations and technical setups
- [ ] `wiki/syntheses/` — if `/query` sessions produce syntheses worth keeping
- [ ] `projects/` — if you have time-boxed initiatives to track separately from the wiki

---

## UNIVERSAL NOTES

- **CLAUDE.md ≤ 500 lines**: the orchestrator should stay light; the real content lives in the folders
- **Always cite sources**: every wiki page points to its `wiki/sources/` entry
- **Date every update**: to track freshness
- **Closed tags**: never invent a tag on the fly
- **Progressive granularity**: max 7-8 folders at the start, 30+ notes before adding another
- **Minimal context per agent**: each agent gets only what it needs
- **Plain Markdown**: total independence from any specific AI model (portable to Gemini, Codex, etc.)
- **Compounding knowledge**: the system becomes exponentially more useful over time
- **Vault principle**: every useful conversation enriches the wiki, it doesn't vanish into chat history
- **500-page limit**: absolute technical ceiling — beyond it, vector search collapses, hallucinations increase, and token cost explodes. Density > volume.
- **Mandatory Red Line**: every vault must have `context/ligne-rouge.md` — the non-negotiable constraint written by the human, not inferred by the AI

---

## EXAMPLE OF AN INSTANTIATED CLOUD.MD

```markdown
# [Domain name] Brain — Schema

**Version**: 1.0
**Purpose**: Knowledge base for [domain]
**Owner**: [Your name]
**Last Updated**: YYYY-MM-DD

## VAULT IDENTITY

**Focus**: [Main domain]
**Sub-domains**: [1], [2], [3]
**Out of scope**: [What we don't ingest]

## AGENTS

- **Wiki Ingestor** → `.claude/agents/ingestor.md`
- **Wiki Librarian** → `.claude/agents/librarian.md`
- **Wiki Query** → `.claude/agents/query.md`

## CONVENTIONS

See `context/naming-conventions.md` for the full tag list.

## COMMANDS

/ingest | /query | /lint | /save

[Refer to vault-methodology-template.md for full detail]
```

---

*Version 4.1 — CC BY-NC 4.0 — Nicolas / Datashiru.*
*Inspired by: Andrej Karpathy (LLM memory) + Pillitteri/Second Brain + agricidaniel/claude-obsidian + Carpati/strategic-forgetting + Joel Parker Henderson (ADR)*
