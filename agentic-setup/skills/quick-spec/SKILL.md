---
name: quick-spec
description: Take a list of features from the user, flesh out unclear things, then produce a lightweight spec to track implementation progress.
---

# Skill: Quick Spec

## When to Use

User gives you a list of features or tasks they want built. You need to clarify, then produce a minimal living spec to track progress.

## Process

### Phase 1: Clarify

Read relevant codebase context first (don't ask what you can look up).

Then ask — in a single batch, max 5 questions — about anything that is:

- Ambiguous in intent (what does "support X" actually mean?)
- Architecturally unclear (where does this live? new table or existing?)
- Has multiple valid approaches worth surfacing

For each question, state your assumed default so the user can just confirm.

Wait for answers before writing anything.

### Phase 2: Flesh Out Each Feature

For each feature in the list:

1. Restate it as a one-line deliverable (what's testable when done)
2. Note any edge cases, gotchas, or implicit decisions surfaced during clarify
3. Flag anything still unclear as a `> ⚠️ open question` block

### Phase 3: Write the Spec

Write to `tmp/quick-spec.md` (or a name the user provides).

#### Format

```md
# Quick Spec: <title>

> Status: In Progress

## Features

### [ ] 1. Feature Name

**Delivers:** one-line testable outcome
**Notes:** edge cases, decisions, constraints
**Steps:**

- [ ] atomic step
- [ ] atomic step

> ⚠️ Open question: ... (remove once resolved)

---

### [ ] 2. Feature Name

...
```

#### Rules

- Each feature gets a checkbox. Check it (`[x]`) when done.
- Steps are atomic — one logical change each.
- Keep notes tight — only decisions and gotchas that aren't obvious.
- No boilerplate sections. No risks table, no monitoring section, no feature flags unless the feature needs it.
- If a feature is genuinely simple (one file change), skip steps entirely.
- Mark open questions inline with `> ⚠️`. Resolve them during implementation and remove the block.

## Living Doc Behavior

- Check off steps and features as they complete
- Add new discoveries inline (don't leave them in chat — they belong in the spec)
- If scope changes, update the spec rather than creating a new one

## Anti-patterns

- Don't write the spec before clarifying. Questions first.
- Don't pad with sections that don't apply (testing, monitoring, etc.) unless asked.
- Don't write vague steps like "implement feature X." Steps must be actionable.
- Don't ask questions you can answer by reading the code.
