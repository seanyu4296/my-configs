---
name: xendit-release-post
description: Create a release post in #sean-releases Slack channel based on the latest GitHub release of a given repo
---

# Skill: Xendit Release Post

Create a release post in `#sean-releases` Slack channel based on the latest GitHub release of a given repo.

## Workflow

### Step 1: Ask for the GitHub repo

Ask the user which GitHub repository to create the release post for. Expect the format `owner/repo` (e.g. `xendit/third-party-integration-service`). If the user only gives the repo name, assume the org is `xendit`.

If the user mentions multiple repos in one request, create a single combined release post (see "Combined Multi-Repo Posts" section below).

### Step 2: Get the latest release from GitHub

Use the `gh` CLI:

1. List releases to find the latest tag:
   ```bash
   gh release list --repo {owner}/{repo}
   ```
2. View the release details using the tag from step 1:
   ```bash
   gh release view {tag} --repo {owner}/{repo}
   ```

This gives you the release version and the full release notes (the "What's Changed" section with PR list).

### Step 3: Read each PR description for context

For every PR listed in the release notes, use `mcp_github_get_pull_request` to fetch the full PR description. You need this to:

- Write the _Why_ section (a concise RCA-style summary of what the release does and why, based on the actual PR descriptions)
- Write the _Risks_ section (assess risk level based on the scope and nature of changes in the PRs)

### Step 4: Format the release post

Format the message following this template:

```
**[{repo-name} {version} release]**

Hi <!subteam^S03HY17BSJ2>, requesting approval for the following:

{repo-name} {version}:
**What's Changed**
* {PR title} by @{author} in {PR link}

**Full Changelog**: {changelog link}

**Why**
* {write a concise RCA-style summary based on the PR descriptions you read}
    * {add sub-bullets with key details, reference specific PRs for full investigation}
    * {if something is "code" like a variable name or value, wrap in backticks}

**When**
* as soon as approved

**Risks**
* {assess risk level: Low / Medium / High}
    * {explain why, based on the nature of changes}

**Tests**
* {see repo-specific defaults below, otherwise ask the user}

**Monitoring**
* {ask the user or leave as placeholder}

**Mitigation**
* {see repo-specific defaults below, otherwise ask the user}

Contributors: {list contributors as <@U01B6C4CQT0|sean yu>, resolve slack user IDs if possible}
```

### Slack Formatting Rules (apply to ALL repo templates)

**Bold text:**

- Use `**text**` for the title and all section headers
- Example: `**[tpi-service v9.24.5 release]**`, `**Why**`, `**Risks**`

**Bullets:**

- Use `*` for ALL bullets (top-level and sub-bullets)
- Sub-bullets are indented with 4 spaces before the `*`
- Do NOT use `•` (U+2022), `◦` (U+25E6), or `▪︎`. Only use `*`.
- Example:
  ```
  * Top-level bullet
      * Sub-bullet with 4-space indent
      * Another sub-bullet
  ```

**Section spacing:**

- Do NOT use `&#x200B;` or zero-width spaces
- Just use a single blank line between sections for separation
- Slack will render this with normal spacing

**Links:**

- Keep all links clickable. Do NOT strip URLs.
- PR links, changelog links, test links, monitoring links -- keep them all.
- For test links and monitoring links, use Slack's `<url|display text>` format when a display label makes sense.
- Example: `<https://xendit.slack.com/archives/C067FE8HX61/p123|using payment session>`

**Code formatting:**

- Wrap code-like values in backticks: variable names, field names, version numbers in context, error messages, etc.
- Example: `weightInKg: 24.0`, `JSON.parse`, `rawBody`

**Release notes section:**

- Do NOT add a "Release Notes" label. Go straight into the repo name and version.
- Copy the GitHub release notes as-is: keep the `**What's Changed**` header, `*` PR bullets with links, and `**Full Changelog**` line with link.
- Do NOT reformat the PR bullets or strip links from this section.

**Why section:**

- Write this as an RCA-style summary, not a dry TLDR.
- Reference specific PRs for full investigation details.
- Use sub-bullets for technical details.
- Always write your own summary based on reading the actual PR descriptions. Do not ask the user.

**Risks section:**

- Always assess risk yourself based on the PR changes.
- Consider: does it touch critical payment paths? Is it behind a feature flag? Is it a refactor vs new feature? Is it a config change?

**General:**

- Don't use emdash
- For contributors, try to resolve GitHub usernames to Slack users using `mcp_slack_slack_search_users`
- Use `<!subteam^S03HY17BSJ2>` to tag the approver group

### Combined Multi-Repo Posts

When the user mentions multiple repos (e.g. "release post for tpi-gateway and tpi-service"), auto-detect this and create a single combined post:

- Title includes all repos and versions: `**[tpi-gateway v3.5.1 + tpi-service v9.24.5 release]**`
- Each repo gets its own release notes block with `**What's Changed**` and `**Full Changelog**`
- The Why/Risks/Tests/etc. sections cover all repos combined
- Contributors list includes contributors from all repos

### Step 5: Apply repo-specific defaults

#### For `xendit/third-party-integration-service` (tpi-service)

**Tests**: default format (override if user provides test links):

```
**Tests**
* :white_check_mark: checkout flows using payment session
* :white_check_mark: refund flows
```

If the user provides test links, use Slack link format:

```
**Tests**
* :white_check_mark: <https://xendit.slack.com/archives/xxx/pxxx|using payment session>
* :white_check_mark: <https://xendit.slack.com/archives/xxx/pxxx|using payment links>
```

**Mitigation**: always use this format:

```
**Mitigation**
* deployment has canary
```

**Monitoring**: always use this format:

```
**Monitoring**
* <https://app.datadoghq.com/dashboard/xmj-rzh-fg3?fromUser=true&refresh_mode=sliding&live=true|New TPI Dashboard 2025>
* <https://app.datadoghq.com/dashboard/znd-tve-6i7/tpi-infrastructure-alerts?fromUser=false&refresh_mode=sliding&live=true|TPI Infrastructure Monitor>
```

#### For `xendit/tpi-gateway`

Uses the same defaults as tpi-service (Tests, Monitoring, Mitigation) since they are companion services. When released together with tpi-service, combine into a single post.

#### For other repos

Ask the user to provide Tests, Monitoring, and Mitigation details.

#### For `xendit/unified-payments-data` (upd)

This repo uses a different format. The release post is more freeform and context-heavy. Follow this structure:

```
**[unified-payments-data for prod-live]**

Hi <!subteam^S03HY17BSJ2> requesting approval for the following release:
* release {version}

{paste github release notes with **What's Changed** and **Full Changelog**}

**Who**
* <@U01B6C4CQT0|sean yu>
* <@URMGWKBN2|nic> for backup

**Tests**
* {describe what was tested and where}
```

Key differences from the default template:

- Title format is `**[unified-payments-data for prod-live]**` (includes environment)
- Uses **Who** instead of Contributors
- Includes Nic (`<@URMGWKBN2|nic>`) as backup by default
- No separate Monitoring or Mitigation sections
- Channel to post: `#sean-releases` (channel ID: `C0A79DZ0TSP`)

#### For `xendit/xendit-invoice-service` (invoice-service)

This repo uses a slightly different format. Follow this structure:

```
**[invoice/sessions release {version}]**
hi <!subteam^S03HY17BSJ2>, requesting approval for the following:

{paste the What's Changed and Full Changelog inside a code block}

steps
{list any deployment steps if applicable, e.g. merge order, config changes}

**when:**
on approval

**risks: {Low/Medium/High}**
* {describe the risk}
    * {explain mitigations or testing done}

**tests:**
* {link to test report or describe tests} :check-passed:
    * some failed tests
        * {explain any failures, e.g. flaky tests, deprecated features}

**who:**
* <@U01B6C4CQT0|sean yu>
* contributors: <@U01B6C4CQT0|sean yu>

**links:**
* <https://app.datadoghq.com/dashboard/rbp-dh2-24m/invoice-monitoring-v2-prod-live?fromUser=false&refresh_mode=sliding&live=true|invoice dashboard monitoring>
* <https://app.datadoghq.com/metric/explorer?...|payment traffic>
* <https://app.datadoghq.com/apm/traces?...|traces>

**mitigation:**
* canary deployment
* abort if there is anomaly mid rollout
* rollback the deployment to stable release via buddy pipeline
```

Key differences from the default template:

- Title format is `**[invoice/sessions release {version}]**`
- The What's Changed section goes inside a code block (triple backticks)
- Section headers use `**header:**` (bold with colon)
- Risk level is inline with the header: `**risks: Low**`
- Has a **steps** section for deployment order if needed
- Has a **links:** section with Datadog dashboards
- Has a **who:** section instead of Contributors
- Default mitigation is: canary deployment, abort on anomaly, rollback via buddy pipeline
- Channel to post: `#sean-releases` (channel ID: `C0A79DZ0TSP`)

### Step 6: Send to Slack

Before sending, show the formatted message to the user for review. Ask if they want to:

1. Edit anything
2. Send as-is

Once confirmed, send the message to the appropriate channel using `mcp_slack_slack_send_message`.

- Default channel: `#sean-releases` (channel ID: `C0A79DZ0TSP`)
- For releases that need broader approval: `#limited-release-policy-and-p0-approvals` (channel ID: `C05HXBKE886`)

## Important Notes

- Always use GitHub MCP to fetch release data and PR details, not web search.
- Always use Slack MCP to send the message.
- The `#sean-releases` channel ID is `C0A79DZ0TSP` (private channel).
- The approver subteam tag is `<!subteam^S03HY17BSJ2>`.
- Current user's Slack ID is `U01B6C4CQT0` (Sean Tristan Yu / sean yu).
- Don't use emdash.
