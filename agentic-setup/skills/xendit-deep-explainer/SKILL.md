---
name: xendit-deep-explainer
description: Create a comprehensive explainer doc for any Xendit service or feature. Teaches the reader everything from scratch so they can understand, operate, and extend the system.
---

# Skill: Deep Explainer Doc

## When to Use

When the user wants a comprehensive explainer document for a service, feature, or system they don't know. The output should teach someone with zero context enough to understand, operate, debug, and add features.

## Research Process

### Phase 1: Gather Everything

Use ALL available sources:

1. **GitHub** — repo structure, source code, README, package.json, OpenAPI specs, config files, PRs (for history/decisions), dictionary files
2. **Confluence** — design docs, RFCs, tech specs, runbooks, migration plans
3. **Slack** — discussions, incidents, decisions, context on why things are the way they are
4. **Local codebase** — if the service is in the workspace or has consumers in the workspace
5. **Related repos** — gateway dictionaries, SDK clients, consumer services

Focus on:

- The core business logic files (services layer)
- The wiring/DI file (how things connect)
- The data model (entities/schemas)
- The controller/route layer (HTTP interface)
- Config/env vars
- How other services integrate with it

### CRITICAL: You MUST read actual source code before writing

**Do NOT write the doc after only reading Confluence, READMEs, and directory listings.** Those give you the shape but not the substance. Before you start writing, you MUST have read:

1. **The main entry point / index file** — understand how the app boots and wires together
2. **The core business logic files** (at least 2-3 key files) — read the actual function implementations, not just their names
3. **Type definitions / schemas** — the actual Zod schemas, TypeScript interfaces, or DB entity definitions
4. **Configuration / env vars** — the `.env.example` or config file to know what's configurable
5. **At least one integration point** — an actual route handler or API contract with request/response shapes

If you skip this and write from metadata alone, the doc will be shallow and generic. The reader needs to see actual code patterns, real type definitions, and specific implementation details — not high-level paraphrasing of what a README already says.

**Rule of thumb**: If you haven't called `get_file_contents` on at least 5-8 actual source code files (not just directory listings), you haven't researched enough to write an in-depth doc.

### Phase 2: Write the Doc

Write to `tmp/{service-name}-explainer.md`. Include ALL of the following sections:

**Length expectation: The final doc should be 800+ lines of markdown.** If your doc is under 500 lines, it's too shallow. A comprehensive explainer for a real service needs space to show code, schemas, diagrams, examples, and gotchas. Don't compress — expand.

#### Required Sections

1. **What Is This Service?** — one paragraph plain english. What it does and why it exists.

2. **Who Uses It?** — table of all consumers with how they use it. Include a mermaid graph.

3. **High-Level Architecture** — mermaid diagram showing the service in context (what calls it, what it calls, what data stores it uses).

4. **Business Flows** — mermaid sequence diagrams for EACH major flow. Show the complete happy path including all service-to-service calls. No colors (`rect rgb(...)`) in diagrams.

5. **Codebase Layout** — full directory tree with one-line descriptions for every important file and folder. This is how the reader navigates the repo.

6. **Dependency Injection / Wiring** — explain the init/bootstrap file. Show how repositories, services, and controllers are wired together.

7. **Data Model** — mermaid ER diagram of database entities. Describe key fields and relationships in text.

8. **Core Logic Walkthrough** — the most important function(s) explained step-by-step with code snippets. ALWAYS include GitHub links for every code reference. **Show 2-4 key functions with their actual implementations (10-30 lines each), not just descriptions of what they do.** Walk through the code line by line where it matters. Include the actual type signatures, the actual Zod schemas, the actual data flow. This section alone should be 100+ lines.

9. **Integration Points** — how other services call this one. Include actual request/response JSON examples. Show the consumer code if available in the workspace.

10. **Configuration** — table of key env vars and what they control.

11. **Development & Operations** — local setup, running tests, migrations, deployment process.

12. **Adding New Features** — 2-3 concrete examples of common modifications. E.g., "add a new endpoint", "add a new entity", "add a new scope/permission".

13. **Key Files Reference** — table format: "if you're working on X, look at Y".

14. **Design Decisions & Gotchas** — why things are the way they are. Common traps that would cost hours.

15. **Sources** — links to ALL GitHub repos, Confluence pages, Slack threads referenced.

#### Style Guidelines

- **Top-down** — start with what/why before how. Context before code.
- **Code snippets are short** — show only the relevant 5-20 lines, not full files.
- **Every code reference has a GitHub link** — `[filename](https://github.com/...)`.
- **Mermaid diagrams have no colors** — no `rect rgb(...)` blocks.
- **Tables for structured info** — env vars, error codes, file references.
- **Gotchas are specific** — "the scope string `INVOICE` won't match `INVOICE.READ`" not "be careful with scopes".
- **Assume zero prior knowledge** — explain acronyms, service names, and Xendit-specific concepts.
- **Flag gaps explicitly** — if you couldn't find something, say so rather than guessing.

## Anti-patterns

- Don't write a wall of prose without diagrams. Every flow needs a sequence diagram.
- Don't show code without a GitHub link. The reader needs to find it.
- Don't skip the "Adding New Features" section. That's half the point.
- Don't assume the reader knows what XAG, TPI, BID, or app_mode means. Define them.
- Don't include colors in mermaid diagrams.
- Don't guess at behavior you haven't verified in code. Read the source.
- Don't dump the entire file. Show the 5-20 relevant lines with context.
- Don't skip the wiring/init explanation. New devs always struggle with "where does this get instantiated?"
- **Don't write the doc after only one round of research.** If your first draft only came from Confluence + directory listings, you MUST do a second research pass reading actual source files before publishing.
- **Don't describe code abstractly when you can show it.** "The function processes documents" is useless. Show the actual function with its parameters, return type, and key logic steps.
- **Don't skip type definitions.** Show the actual Zod schemas, interfaces, or entity shapes. Types ARE documentation.
- **Don't omit the .env / config.** Developers spend hours figuring out what env vars are needed. Show ALL of them from the actual `.env.example`.
