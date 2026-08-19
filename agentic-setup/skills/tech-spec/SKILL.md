---
name: tech-spec
description: Write a tech spec through iterative discovery, architectural decisions, and structured documentation
---

# Skill: Tech Spec Writing

## When to Use

When the user says they want to write a tech spec, design doc, or discuss a technical approach before implementation.

## Process

### Phase 1: Problem Discovery

1. Let the user explain the problem and their proposed solution
2. Read relevant codebase files to understand current state (types, adapters, processors, repos, utilities)
3. Ask clarifying questions in batches, starting with **user-facing behavior** then drilling into architecture:

   **First: Who and What (User Stories)**
   - Who are the actors affected? (merchant, end-customer, platform admin, TPI system, ops)
   - What can they do after this that they can't today?
   - What's the minimum set of stories that justify this work?
   - Are there stories we're explicitly NOT doing? (out-of-scope stories)

   **Then: How (Architecture and Behavior)**
   - Precision of behavior (e.g. rounding direction, edge cases)
   - Where logic should live (shared utility vs adapter vs processor)
   - Data model changes (new fields, optional vs required, fallbacks)
   - Rollout strategy (per-plugin, per-currency, feature flags)
   - Backward compatibility (old entities, rolling deploys)
   - When side-effects happen (on success, on creation, transactional or not)

### Phase 2: Iterative Refinement

1. Go back and forth with the user on each question
2. For architectural choices, present options with pros/cons and let the user decide
3. Keep asking until all implementation gaps are covered

#### Implementation Planning (Collaborative)

Once architecture is settled, **propose** (not declare) an implementation plan:

1. Draft a sequenced plan and present it to the user as a proposal, not a final answer
2. For each slice, identify:
   - What it delivers (a testable increment, not just "code exists")
   - What it depends on (other slices, external teams, infra, config)
   - What it unblocks (downstream slices, QA, other teams)
   - Whether it's a **code PR** (mergeable unit) or a **delivery milestone** (shippable to users)
3. Explicitly flag assumptions and things you cannot know:
   - Deploy ordering constraints (canary, staged rollout, feature flags)
   - Team dependencies (platform team, infra, QA scheduling)
   - Operational readiness (monitoring, alerting, runbooks needed before go-live)
   - Coordination needs (comms to merchants, internal announcements, freeze windows)
   - Rollback strategy per slice (is each slice independently reversible?)
4. Ask the user to confirm, reorder, split, or merge slices
5. Only finalize the plan after user sign-off

**Do NOT assume:**

- Who owns the deploy pipeline or release process
- Whether there's a release train, freeze, or approval gate
- Whether QA is automated or needs manual coordination
- Whether other teams need a heads-up before merge
- How long operational setup (dashboards, alerts) takes

4. Confirm the full picture before drafting

### Phase 3: Writing the Tech Spec

Write the spec in `tmp/` (per user preference) with these sections:

**Living Document Behavior:**

The spec is not a one-shot artifact. It evolves during implementation:

- Steps get checked off (`- [x]`) as they're completed
- Completed phases get marked ✅ with branch name + PR link
- New discoveries during implementation get added (new edge cases, revised decisions)
- FAQ sections grow as reviewers ask questions
- If a decision changes during implementation, update the Key Decisions section (don't just override — note what changed and why)

This means an agent can pick up the spec, find the next unchecked step, implement it, check it off, and stop. The next session continues from where it left off.

#### Required Sections

1. **Title** - concise, describes the capability
2. **Why** - business problem, reference links (Slack, tickets)
3. **User Stories** - who benefits and what they can do after this ships
   - Format: As a [actor], I want [capability], so that [outcome]
   - Acceptance criteria as sub-bullets (Given/When/Then or plain conditions)
   - Mark priority: must-have vs nice-to-have
   - Max 3-5 stories per spec (more = feature too big, split it)
   - Actors should be specific to context (merchant, platform admin, end-customer, TPI system) not generic "user"
   - Include "Out of Scope Stories" subsection for stories explicitly deferred
4. **Current State** - how things work today, with flow diagrams (mermaid sequence diagrams)
5. **Target State** - how things will work after, with flow diagrams showing all external API calls and amounts
6. **Key Decisions** - each one structured as:
   - Problem/question statement (as a heading question)
   - **Bolded decision** statement
   - Brief rationale
   - Code example demonstrating the decision
   - "Why not the others:" section with short, conversational reasons for rejecting alternatives
7. **Logic Implementation** - illustrate the core logic through code snippets and/or diagrams (mermaid flowcharts for branching, `stateDiagram-v2` for state transitions). Covers: state models, key branching/routing logic, data transformations, error handling paths. Use whichever is clearer — code or diagram — for each piece of logic. Not full files, just the critical path that makes each decision concrete.
8. **Data Model Changes** - interfaces with JSDoc comments, Mongoose schema notes. Include state machine diagrams (mermaid `stateDiagram-v2`) for any entity with a `status` field showing lifecycle transitions, triggers, and guards.
9. **Interface Changes** - show how module boundaries change: new/modified function signatures, service interfaces, handler contracts, repo method signatures. Focus on the public API of each module — what callers need to know changed. Include before/after when modifying existing interfaces.
10. **Implementation Plan** - sequenced delivery plan (collaboratively agreed with user), structured as:
    - **Phase Dependency Graph** (mermaid flowchart) showing which phases block which and what can be parallelized
    - Each phase/slice has:
      - Name + what it delivers (a testable increment)
      - **Pre-requisites** checklist — what must be true before this phase starts
      - Dependencies (other slices, external teams, infra, config)
      - What it unblocks (downstream slices, QA, other teams)
      - **Rollback strategy** — one-liner: exact action to reverse this phase (required for every delivery milestone)
      - **Steps** as markdown checkboxes (`- [ ]`) — granular enough for an agent to pick up and execute one at a time
      - **Automated test cases** as checkboxes — concrete named scenarios that serve as acceptance criteria
      - **Manual test scenarios** as checkboxes — staging verification separate from automated
      - **FAQ** (optional) — preempt reviewer questions about ordering, coexistence, "why not do X here?"
    - Distinguish **code PRs** (mergeable units) from **delivery milestones** (shippable to users)
    - Flag blockers: external team dependencies, operational prerequisites, coordination needs
    - Note assumptions confirmed with user during planning
    - Mark completed phases with ✅ + PR link (spec is a living doc / progress tracker)

    **Checkbox format for agent-executable steps:**

    ```
    #### Phase 0.X: Name — one-line description
    **Delivers:** what's testable after this phase
    **Pre-requisites:** what must be true before starting
    **Rollback:** how to reverse
    **Steps:**
    - [ ] Step 1 (specific, implementable action)
    - [ ] Step 2
    - [ ] Step 3
    **Automated tests:**
    - [ ] Test scenario name — expected behavior
    - [ ] Test scenario name — expected behavior
    **Manual tests:**
    - [ ] Scenario — what to verify in staging
    ```

    Steps should be:
    - Atomic — one logical change per checkbox (a file, a function, a config entry)
    - Ordered — top to bottom = implementation order
    - Self-contained — an agent reading just the checkbox + phase context should know what to do
    - Checkable — mark `[x]` when done, so the spec tracks progress as a living doc

11. **Feature Flags** (when relevant) - table with: key, scope (global/per-BID/per-entity), default value, purpose, kill-switch behavior
12. **Risks and Mitigations** - table format
13. **Testing**:
    - **Automated**: unit tests (pure functions, no I/O) + E2E tests (real DB, mock external APIs via mock servers, TestHttpClient)
    - **Manual**: staging verification scenarios (human-executed, pre-deploy checks)
    - **QA handoff**: scenarios from the user/platform perspective for QA team, with expected behavior, API amounts, and internal state as notes. Each scenario references which user story it validates.
14. **Monitoring** - signals table with: signal/log line, what it means, which phase it becomes relevant, what "normal" looks like, alert threshold
15. **Next Steps** - how to enable for another plugin (with code example), how to enable for another currency
16. **Out of Scope** - what we're explicitly not doing

#### Diagram Guidelines

- **Prefer mermaid diagrams for everything.** Only fall back to ASCII when mermaid cannot express it (e.g., complex inline annotations).
- Use mermaid `sequenceDiagram` for request/response flows (current state, target state)
- Use mermaid `stateDiagram-v2` for entity lifecycle / state machines (e.g., installation status transitions)
- Use mermaid `flowchart` for decision trees, branching logic, and phase dependency graphs
- Include ALL external API calls with their payloads/amounts (e.g. Shopify GraphQL mutations, Xendit API calls)
- Show async flows (webhooks, Kafka events) clearly separated from sync request/response
- Include DB operations (what gets stored, what gets read)
- Label amounts at each step so the reader can trace the money

#### Style Guidelines

- Keep alternatives short and conversational ("Why not the others:" not "Alternatives considered:")
- QA scenarios from the platform/merchant perspective, internal state as notes
- State the Xendit API amount explicitly in QA scenarios
- Don't use em dashes (—) excessively
- Bold the decision, not the alternatives
- **Code blocks to illustrate crucial logic** — state machines, domain models, key branching logic, data transformations. Not full files, just the critical path that makes the decision concrete.
- Implementation plan distinguishes code PRs from delivery milestones, with dependencies and blockers explicit
- No indexes section needed if existing indexes cover new queries

#### Scenarios Format

For current/target state scenarios, structure as:

```
#### {Scenario Name} (still to be tested)
1. Step from platform/user perspective
2. What TPI does
3. What Xendit receives
4. Result
```

For QA scenarios, structure as:

```
N. **Scenario name** (validates Story #X)
   - What the platform/merchant does
   - Expected: what the user sees
   - Xendit Payment Session / Refund V3: amount sent
   - Internal: collection.field values (as a note)
```

#### Logic Implementation Guidelines

Code blocks and diagrams in the spec serve to make decisions concrete, not to be copy-pasted. Include:

- **Crucial logic** — the core branching/routing that implements a key decision (e.g., FF-first routing, token lookup strategy). Use code or mermaid flowchart, whichever is clearer.
- **State machines** — entity lifecycle transitions with all triggers and guards (mermaid `stateDiagram-v2`)
- **Domain models** — TypeScript interfaces with JSDoc comments explaining each field's purpose and constraints
- **Data transformations** — how data flows between systems (what goes in, what comes out, what gets stored)
- **Error types** — discriminated unions showing all failure modes

Do NOT include: boilerplate wiring, full file contents, import statements (unless the import IS the decision), or test code in this section (tests go in the Implementation Plan per phase).

## Anti-patterns

- Don't write the tech spec immediately. Always ask questions first.
- Don't assume implementation details. Read the codebase.
- Don't present only one option for architectural choices. Give pros/cons.
- Don't write tests as "mocked DB" if the project uses real DB in tests.
- Don't use verbose "Alternatives considered:" bullet lists. Keep it short and human.
- Don't add unnecessary sections (e.g. indexes if none are needed).
- Don't write a doc unless the user says to. The conversation IS the discovery phase.
- Don't only document what the user explicitly said. Scan for implicit decisions that need visibility.
- Don't write vague user stories like "As a user, I want the system to work better." Stories must have concrete, testable acceptance criteria.
- Don't skip out-of-scope stories. Explicitly listing what you're NOT building prevents scope creep during review.
- Don't present the implementation plan as final. It's a proposal. The user knows operational/org constraints you don't.
- Don't conflate "code is merged" with "feature is delivered." A PR is not a milestone unless users can benefit from it.
- Don't use ASCII diagrams when mermaid can express the same thing. Mermaid is renderable, diffable, and reviewable.
- Don't list test cases as vague "add tests for X." Each phase needs concrete, named test scenarios as acceptance criteria.
- Don't treat the spec as a one-shot artifact. It's a living doc — mark phases done, link PRs, update as implementation reveals new info.

## Identifying Undocumented Pivotal Decisions

After writing the spec, scan it end-to-end and ask: "What decisions here would a senior engineer or PM want to review that aren't explicitly called out?"

Look for:

1. **Business impact decisions** - things that affect merchants, money, or user experience (e.g. "we're overcharging by up to 1 unit per transaction" is a business call, not just a technical one)
2. **External system awareness gaps** - does the external platform (Shopify, WooCommerce) know about the change? If not, document why that's acceptable and what the reconciliation story is
3. **Data consistency tradeoffs** - any place where you chose eventual consistency over transactions, or accepted a race condition window
4. **Validation boundary decisions** - what counts as "valid input" vs "fixable input" vs "rejected input" (e.g. ISO violation = reject, Xendit precision mismatch = fix via rounding)
5. **Edge cases that won't be handled** - explicitly call out what you're deferring (reversals, edge currencies, etc.) so reviewers know it's intentional

These often surface as things the implementor "just knows" but that aren't obvious to someone reviewing the spec for the first time. Make them explicit as key decisions with their own question/answer structure.
