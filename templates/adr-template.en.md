---
tags: [tag1, tag2]
type: Decision
created: YYYY-MM-DD
decision_date: YYYY-MM-DD
status: proposed        # proposed | accepted | superseded | deprecated
supersedes: ""          # "[[decisions/old-decision]]" if it replaces one
superseded_by: ""       # "[[decisions/new-decision]]" if it has been replaced
related: []
source: ""              # "[[sources/...]]"
---

# <Verb + object — e.g. "Choose X over Y">

## Context

<The force that triggered the decision: problem, constraint, deadline. Why did this need deciding now?>

## Options considered

<MANDATORY — at least one rejected option, otherwise this isn't a decision, it's an announcement.>

- **Option A** — <description> → chosen / rejected because <reason>
- **Option B** — <description> → rejected because <reason>
- **Option C (status quo)** → rejected because <reason>

## Decision

<What was chosen, and the rationale: why this route and not the others.>

## Consequences

<What we're accepting in exchange — benefits AND costs.>

- ➕ <benefit>
- ➖ <cost / trade-off we're owning>
