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
tests: ping-pong         # ping-pong | user | per-task
research: []            # research files, if the user asked for any
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
- **Background:** D1, `docs/research/<topic>.md` §2
- **Done when:** the new test passes and the full suite is green

### [ ] T2: <title> · owner: agent

...

### [ ] T3: <title> · owner: user · outline

- **Goal:** <one sentence>
- **Depends on:** Q1, whatever T2 decides about <thing>

## Open questions

- Q1: <question for the user>

## Log

- YYYY-MM-DD T1 started, base <sha>
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
- **No placeholder lines in briefed tasks.** "TBD", "handle edge
  cases", "add validation" and "write tests" decide nothing. Replace
  each one with the concrete decision, or turn it into an open question.
- **Unconfirmed assumptions are marked `(?)`** and must be settled before
  the task that depends on them starts.
- **Log each task's start with its base commit**, tick the task
  (`### [x] T1`) and add a log line when it finishes. Update `status` and
  `updated` in the frontmatter as you go. Set `status: approved` only
  after the user approved the plan, `in-progress` when the first task
  starts, and `done` when the last one is ticked.
- **Brief two ahead, outline the rest.** Aim to keep the next two tasks
  fully briefed, so the user can keep working without you (out of tokens,
  offline, during an outage). A task that depends on an open question or
  on how an earlier task turns out stays an outline until that's settled.
  Outlines (marked `outline`) have a title, an owner, a goal, and what
  they wait on. They name the decisions still to be made instead of
  guessing them. Brief an outline once it moves up, and adjust it to what
  earlier tasks actually decided.
