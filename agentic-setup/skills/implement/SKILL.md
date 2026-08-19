---
name: implement
description: Implement what the user asks, then find and update any related todo/checklist tracking in specs, docs, or task files
argument-hint: "<what to implement>"
---

# Skill: Implement

Just implement. No lengthy preamble, no asking unnecessary questions. Read relevant code, do the work, then update any tracking docs.

## Process

### 1. Understand the Task

- Read the user's request
- If a tech spec, task file, or checklist is referenced (or obviously exists nearby), read it to find the specific step(s) being asked about
- Read relevant source files to understand current code shape before writing anything
- If the request is genuinely ambiguous in a way that would cause you to implement the wrong thing, ask ONE clarifying question — not multiple, not a batch

### 2. Implement

- Do the work: create/edit files, wire things up, follow existing patterns in the repo
- Match the project's style, conventions, and libraries — don't introduce new ones without reason
- Keep the implementation scoped to what was asked — don't expand scope, don't add speculative abstractions

### 3. Verify

- Run the build or type-check if available
- Run relevant tests if they exist
- Fix any errors before reporting done

### 4. Update Tracking Docs

After implementing, scan for any tracking documents that reference this work and update them:

**Where to look:**

- `tmp/*.md` — tech specs written by the tech-spec skill
- Any file with markdown checkboxes (`- [ ]`) that references the implemented feature/step
- `TODO`, `TASKS`, `CHANGELOG` files in the repo root
- `.kiro/specs/*/tasks.md` — Kiro spec task files

**What to update:**

- Check off completed steps: `- [ ]` → `- [x]`
- Mark completed phases with ✅ and add branch/PR link if known
- If a phase is fully checked off, mark its heading as done
- Do NOT modify unchecked items, reorder steps, or change content — only check off what was just implemented

**Format:**

```
- [x] Step that was just implemented
```

If multiple steps were completed in one go, check all of them off.

### 5. Report

Keep it short:

- What was implemented (1-2 sentences)
- Any files created or modified (list)
- What was checked off in tracking docs (if any)
- Any blockers or follow-up needed

## Rules

- Don't ask for permission to start — just implement
- Don't rewrite things that weren't asked about
- Don't add tests unless the user asked or the project's conventions require it for the change
- If you can't find a tracking doc, that's fine — skip step 4 and say so
- If a step in the tracking doc doesn't map cleanly to what was implemented, use judgement — only check it off if it's genuinely done
