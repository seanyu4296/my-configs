---
name: create-sub-issues
description: Break down an implementation plan into GitHub sub-issues under a parent epic
inclusion: manual
---

# Create Sub-Issues from Implementation Plan

Skill for breaking down an implementation plan document into GitHub sub-issues under a parent epic.

## When to Use

When the user provides:

1. A parent GitHub issue (epic) URL or number
2. An implementation plan document with phased tasks (can be a md or confluence doc or others)

## Process

### Step 1: Gather Context

1. Fetch the parent issue to get its `node_id`, repo owner, and repo name
2. Read the implementation plan to identify phases/tasks
3. Check if any sub-issues already exist on the parent
4. Query the `repositoryId` and `assigneeId` (if assigning):

```bash
gh api graphql -f query='{ repository(owner: "OWNER", name: "REPO") { id } }'
gh api graphql -f query='{ user(login: "USERNAME") { id } }'
```

### Step 2: Confirm Assignment

Ask the user: "Should I assign all sub-issues to `seanyu4296`, or leave them unassigned?"

### Step 3: Propose Sub-Issues

Present the list of sub-issues to the user BEFORE creating them. Each sub-issue should have:

- **Title**: Prefixed with phase (e.g., `[Phase 0] Remove feature flag`)
- **Body format**:

  ```markdown
  **Repository:** `repo-name`

  ## Acceptance Criteria

  - [ ] Criterion 1
  - [ ] Criterion 2
  ```

Guidelines for acceptance criteria:

- Focus on observable outcomes, not implementation steps
- Include deploy/monitoring criteria where applicable
- Keep it verifiable — someone else should be able to check these off

### Step 4: Generate Scripts and Get Approval

Generate two bash scripts and present them to the user for review. Only execute after user approves.

**Script 1: `create-issues.sh`** — Creates all issues and outputs their node IDs

```bash
#!/bin/bash
set -e

REPO_ID="<REPO_NODE_ID>"
ASSIGNEE_ID="<USER_NODE_ID>"

echo "=== Creating sub-issues ==="

ISSUE1=$(gh api graphql -f query="
mutation {
  createIssue(input: {
    repositoryId: \"$REPO_ID\",
    title: \"[Phase X] Title here\",
    body: \"**Repository:** \\\`repo-name\\\`\n\n## Acceptance Criteria\n\n- [ ] Criterion 1\n- [ ] Criterion 2\",
    assigneeIds: [\"$ASSIGNEE_ID\"]
  }) { issue { id number title } }
}")
echo "$ISSUE1"
ISSUE1_ID=$(echo "$ISSUE1" | jq -r '.data.createIssue.issue.id')

# Repeat for each issue...

# Output all IDs for linking script
echo ""
echo "=== Node IDs for linking ==="
echo "ISSUE1_ID=$ISSUE1_ID"
```

**Script 2: `link-sub-issues.sh`** — Links all created issues as sub-issues of the parent

```bash
#!/bin/bash
set -e

PARENT_ID="<PARENT_NODE_ID>"

echo "=== Linking sub-issues to parent ==="

gh api graphql -f query="mutation { addSubIssue(input: { issueId: \"$PARENT_ID\", subIssueId: \"$ISSUE1_ID\" }) { subIssue { number title } } }"
gh api graphql -f query="mutation { addSubIssue(input: { issueId: \"$PARENT_ID\", subIssueId: \"$ISSUE2_ID\" }) { subIssue { number title } } }"
# Repeat for each issue...

echo "=== Done ==="
```

**Workflow:**

1. Present both scripts to the user for review
2. On approval, run Script 1 — capture node IDs from output
3. Fill node IDs into Script 2, run Script 2

This means only 2 approvals are needed (one per script execution).

### Step 5: Verify

Query the parent to confirm all sub-issues are linked:

```bash
gh api graphql -f query='
{
  repository(owner: "OWNER", name: "REPO") {
    issue(number: PARENT_NUMBER) {
      subIssues(first: 20) {
        nodes { number title }
      }
    }
  }
}'
```

## Important Notes

- Use `gh api graphql` for both creating issues and linking sub-issues — the REST API does not support sub-issues
- Always confirm with the user before executing scripts
- The `gh` CLI must be authenticated (`gh auth status`)
- Write scripts to a temp location, execute them, then clean up
