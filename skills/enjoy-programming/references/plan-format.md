# Plan format

A plan is a markdown file with frontmatter. It holds the decisions the
user made, a map of where changes go, and ordered tasks. It is the shared
state between you and the user, and it has to survive context loss and
sessions without you. Keep it current.

## Template

```markdown
---
title: <feature>
status: draft            # draft | approved | in-progress | done
created: YYYY-MM-DD
updated: YYYY-MM-DD
tests: ping-pong         # user | ping-pong | per-task
research: [docs/research/<topic>.md]
---

# <Feature>

## Goal
One to three sentences: what, why, and how we know it's done.

## Decisions
- D1: <decision> — <why>
- D2 (?): <assumption not yet confirmed by the user>

## Out of scope
- <thing we deliberately don't do>

## Tasks

### [ ] T1: <title> · owner: user

- **Goal:** <one sentence>
- **Where:** `src/Config.hs:120` (`parseConfig`); new `validateKey :: Text -> Either ConfigError Key` in `src/Config/Key.hs`
- **Test first:** `test/ConfigSpec.hs`, "rejects empty key": `parseConfig "" == Left EmptyKey`
- **Pitfalls:** `parseConfig` is also called from `Cli.hs:40` with pre-trimmed input
- **Background:** D1, research §2
- **Done when:** the new test passes and the full suite is green

### [ ] T2: <title> · owner: agent

...

## Open questions

- Q1: <question for the user>

## Log

- YYYY-MM-DD T1 done, review clean (1 minor fixed by agent)
- YYYY-MM-DD T2 parked finding: <one line> (user: fix later)
```

## Rules

- **Order matters.** A task uses only what earlier tasks or existing code
  provide. Each task leaves the code working and testable.
- **Size.** One focused sitting per task. Split tasks where a reviewer
  could reasonably approve one part and reject the other. Fold setup and
  config into the task that needs them.
- **Decisions, not code.** Name files, locations, and the signatures that
  cross task boundaries or that the user agreed on. Don't write function
  bodies for user-owned tasks. A plan longer than the code it describes
  has written the code.
- **No empty lines.** "TBD", "handle edge cases", "add validation" and
  "write tests" decide nothing. Replace each one with the concrete
  decision, or turn it into an open question.
- **Unconfirmed assumptions are marked `(?)`** and must be settled before
  the task that depends on them starts.
- **Tick tasks** (`### [x] T1`) and add a log line when a task finishes.
  Update `status` and `updated` in the frontmatter as you go.
- **Stay ahead.** Keep at least the next two or three tasks fully
  briefed, so the user can continue without you.
