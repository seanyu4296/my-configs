---
name: clarify
description: clarify before doing
---

<!-- old version ignore -->
<!-- clarify and ask questions (batch questions) any uncertain assumptions and unknown knowledge before executing or doing what im askign u to do. discuss and make sure u and i are both aligned. list down what you will do if you are sure already. wait for the go signal to actually do it. -->

Before executing anything the user asks, follow this process:

## 1. Identify Unknowns

- Read relevant code/context if needed to understand current state>
- If a _fact_ can be found by exploring the environment (filesystem, tools, etc.), look it up rather than asking me. The _decisions_, though, are mine — put each one to me and wait for my answer.
  (lean into using a subagent to explore)

Do not act on it until I confirm we have reached a shared understanding.

- Identify uncertain assumptions, ambiguous requirements, and knowledge gaps
- Categorize: what's blocking (can't proceed without answer) vs. what's a preference (could go either way)

## 2. Ask Questions (Batched)

- Ask all questions in a single message, grouped by theme
- Max 5 questions per batch — if more, prioritize blockers first, save nice-to-haves for a follow-up
- For each question, state what you'd assume if the user doesn't answer (so they can just say "yeah that's fine")

## 3. Propose Action Plan

- After answers come in, list exactly what you will do (files to create/edit, commands to run, architecture choices)
- If anything changed from your initial understanding, call it out explicitly
- Do NOT execute yet

## 4. Wait for Go Signal

**Do not write code, create files, run commands, or make any changes until the user explicitly says to proceed.**

A "go signal" is: "go", "do it", "yes", "proceed", "lgtm", or similar explicit confirmation.

Do the repo / services dev flow (e.g. lint, tests) only if i dont tell you

## 5. TO do list

If there is a tech spec or spec or to do list, if u are done, mark the tasks done (thru md or what not).
