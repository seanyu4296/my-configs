---
name: pr-desc
description: Write a concise, human PR description with scope, what changed, and test scenarios
argument-hint: "<branch or feature to describe>"
---

# Skill: PR Description

Write a PR description. Concise, human, no corporate polish. Matches the style of the team's existing PRs.

## Process

### 1. Gather context

- Check git diff to see exactly what files changed: `git diff --name-only <base>..<HEAD>`
- Read changed files to understand what the code actually does
- If a linked PR or spec exists, read it for rollout plan and FAQ context
- Check test files to enumerate the test scenarios covered

### 2. Write the description

Follow this structure exactly (same as the team's existing PRs):

```
**scope**

1-3 sentences explaining what this PR does and why. Plain English, no bullet soup.
Mention branch dependency if this stacks on another PR.
Include rollout notes if relevant (FF status, deploy order, etc.)

**scenarios**

- [x] scenario description -> expected outcome
- [x] scenario description -> expected outcome
...
```

### 3. Style rules

- No em dashes. Use `--` if you need a pause, or just restructure the sentence.
- No `->` replaced with words where it reads better, but `->` is fine for scenario outcomes.
- No "This PR", "This commit", "In this change" openings.
- Write like a human engineer explaining to a teammate, not a changelog generator.
- Scenarios use `[x]` (already tested/done), not `[ ]`.
- Keep the scope section to 3-5 sentences max. If it needs more, the PR is probably too big.
- Mention the shared helper / abstraction if one was introduced, briefly.
- For infra changes (helm, k8s), one sentence is enough.
- FAQ goes in the description only if it's genuinely non-obvious. Don't add FAQ for things that are self-evident from the code.

### 4. What NOT to include

- Tables of files changed
- Headers beyond "scope" and "scenarios"
- Detailed explanations of every implementation decision (that's what code comments are for)
- Marketing language ("powerful", "robust", "seamlessly")
- Passive voice

## Example output

```
**scope**

Adds the refresh token cron job for the Cloudbeds onboarding migration (Phase 0.4). Branches from #1152.

Two things it does:
- **Recover expired new-arch credentials** -- queries `cloudbeds_oauth_credentials` for `expires_at < now`, refreshes them. Dead refresh tokens get marked `EXPIRED`. Non-invalid failures (5xx) are skipped and retried next run. OCC via `__v` handles concurrent refreshes gracefully.
- **Migrate inactive legacy merchants** -- merchants with no checkout traffic never get hit by the eager migration path. Cron fetches up to 100 ACTIVE legacy merchants per run, checks FF per BID, and migrates those not yet in new arch using the same `migrateLegacyMerchantToNewArch` helper as checkout eager migration (same logic, `migration_source: 'cron'`).

Runs twice daily at 00:00 and 06:00 UTC via k8s CronJob. Entry point uses `traceCronJob` + `initInfraDeps` only (no full server boot). Infra added to `xendit-argo-catalog` -- staging enabled, prod `suspend: true` until staging validated.

**scenarios**

- [x] Expired credential -> refresh succeeds -> token updated, status stays ACTIVE
- [x] Expired credential -> CLOUDBEDS_INVALID_REFRESH_TOKEN -> marked EXPIRED
- [x] Expired credential -> Cloudbeds 500 -> stays ACTIVE, skipped (retried next run)
- [x] Non-expired credential -> not touched, zero API calls
- [x] FF=true, not migrated -> credential + installation rows created
- [x] FF=false -> skipped
- [x] Already migrated -> skipped, no duplicate
```
