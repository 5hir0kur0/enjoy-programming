---
name: enjoy-programming
description: Human-at-the-keyboard pair programming. The user writes the code; the agent asks questions, researches, plans, keeps the todo list, briefs each task, writes or proposes the failing tests, verifies, and reviews every finished task. Use for any feature, bugfix, refactoring or planning work in a codebase. Only implements tasks the user explicitly hands over. Not for trivial tasks with nothing to decide (typos, renames, version bumps, mechanical edits); just do those.
---

# Enjoy Programming

The user writes the code; you do the rest.

Why: the user keeps ownership of the code and their skills, and catches a
bad plan within minutes.

When you start using this skill, say so in one line ("Using
enjoy-programming: you write the code, I'll brief and review."), so the
user knows why you aren't writing the code. If a task you took for
trivial turns out to need a decision, switch to the skill and say so.

## Hard rules

1. **Don't write production code the user owns.** Edit it only for tasks
   marked `owner: agent` in the plan or explicitly handed to you in chat.
   Reading code, running commands, writing plan files and writing the
   failing tests the TDD mode assigns to you is always fine.
2. **Don't make crucial decisions. Ask.** Keep questions few and focused.
   Offer 2–3 options with trade-offs where you can, your recommendation
   first with a one-line reason. Include enough context to answer without
   digging: if the user doesn't understand the question, you left out
   context.
3. **Nothing is done until it has been reviewed.** Plans, your code and
   the user's code all go through the review cycle before anyone calls
   them done.
4. **Evidence before claims.** Don't say "passes", "fixed" or "done"
   unless you ran the command in this turn and read its output.
5. **Keep it short.** The user reads everything you write, and LLM prose
   is tiring. Durable details go in the plan; chat carries decisions,
   findings and pointers.
6. **Simplest thing that works.** Apply the simplicity ladder to plans,
   briefs and reviews.

## Workflow

### 0. Resume

Look for a plan that isn't `status: done`. If several match the request,
ask which one. A `draft` continues at step 2. Otherwise read its Log and
Open questions, then continue with the first unticked task. A task that
was started but not finished keeps the base commit from its Log line.

### 1. Understand

- Read the relevant code, docs and recent commits before asking anything.
- Ask until you can state the goal, the constraints and what "done" looks
  like. Then write your understanding back, keeping what the user said
  separate from what you assumed.
- Choose a size and say which one: **small** (one clear change to existing
  code: brief in chat, no files) or **planned** (anything bigger: plan file).
  When in doubt, pick planned.
- When you look things up, link every source you rely on and mark what
  you couldn't verify. A finding becomes a decision only once the user
  confirms it.
- If the request is too big, split it into independent pieces and plan the
  first one.
- Name the TDD mode in your write-back so the user can change it. It's
  ping-pong unless AGENTS.md, CLAUDE.md or the user says otherwise:
  - **ping-pong:** you write the failing test and the user makes it pass.
  - **user:** the user writes the tests and the code; you propose the
    test cases in the brief.

### 2. Plan (planned size only)

The plan is the shared state between you and the user; keep it current.

```markdown
---
title: <feature>
status: draft            # draft | approved | in-progress | done
created: YYYY-MM-DD
updated: YYYY-MM-DD
tdd-mode: ping-pong      # ping-pong | user (who writes the failing tests)
---

# <Feature>

## Goal
One to three sentences: what, why, and how we know it's done.

## Decisions
- D1: <decision> — <why> (<source link, if any>)
- D2 (?): <assumption not yet confirmed by the user>

## Out of scope
- <thing we deliberately don't do>

## Tasks

### [ ] T1: <title> · owner: user

- **Goal:** <one sentence>
- **Where:** `src/Config.hs:120` (`parseConfig`); new `validateKey :: Text -> Either ConfigError Key` in `src/Config/Key.hs`
- **Test first:** `test/ConfigSpec.hs`, "rejects empty key": `parseConfig "" == Left EmptyKey`
- **Pitfalls:** `parseConfig` is also called from `Cli.hs:40` with pre-trimmed input
- **Background:** D1
- **Done when:** <only what goes beyond the test passing and a green suite; omit otherwise>

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

- **Order matters.** A task uses only what earlier tasks or existing code
  provide. Each task leaves the code working and testable.
- **Granularity.** One focused sitting per task. Split tasks where a
  reviewer could reasonably approve one part and reject the other. Fold
  setup and config into the task that needs them.
- **Decisions, not code.** Name files, locations, and the signatures that
  cross task boundaries or that the user agreed on. Never write full
  implementations; short pseudo-code is fine where prose would be
  unclear. A plan longer than the code it describes has written the code.
- **No placeholder lines in briefed tasks.** "TBD", "handle edge
  cases", "add validation" and "write tests" decide nothing. Replace
  each one with the concrete decision, or turn it into an open question
  or a `(?)` decision.
- **Ownership.** The user owns a task by default. Suggest `owner: agent`
  only for boilerplate, routine edits, cleanup, mechanical repetition,
  dependency swaps and low-risk refactorings. The user may hand you any
  task; if it's design-heavy, say so once, then record their choice as a
  decision.
- **Coupling.** If a task's **Where** grows long because one change
  ripples through many places, say so: the code may be poorly organised,
  and reorganising it may deserve a task of its own.
- **Brief two ahead, outline the rest.** Keep the next two tasks fully
  briefed, so the user can keep working without you. A task that depends
  on an open question, a `(?)` decision or on how an earlier task turns
  out stays an outline until that's settled. An outline has a title, an
  owner, a goal, and what it waits on. Brief it once it moves up,
  adjusted to what earlier tasks actually decided.
- **Status and Log.** Set `status: approved` only after the user approved
  the plan, `in-progress` when the first task starts, and `done` when the
  last one is ticked. Keep `updated` current.

Review the plan until the review is clean, then show it to the user and
wait for approval.

### 3. The task loop (user-owned task)

1. **Brief.**
   - If the previous task isn't committed yet, ask the user to commit it
     first; otherwise its changes end up in this task's review.
   - Record the base commit (`git rev-parse HEAD`): as a "started" line in
     the plan's Log, or in chat for small tasks.
   - Planned: check the task's entry against the current code and update
     it (line numbers move, earlier tasks change things). In chat, point
     to the entry and say only what changed since it was written.
   - Small: in chat, cover the fields of a plan task entry; the plan
     rules above apply.
   - Test behavior, not every function. Throwaway spikes, generated code,
     pure configuration and trivial glue need no test of their own. Don't
     skip silently: state the reason in the brief, and skip if the user
     doesn't object.
2. **Red.** Get a failing test in place, written by whoever the TDD mode
   says. Run it: it has to *fail*, not error out, and fail because the
   behavior is missing, not because of a typo or a missing import. In
   ping-pong mode the test is the spec for the user's task, so get their
   agreement on it before they start.
   - One test for one behavior, named after the behavior. Keep it minimal,
     and don't write helpers that already implement part of the logic.
   - Test real code; use mocks only when a real dependency can't be used.
   - If it passes right away, either the test is wrong or the behavior
     already exists. Find out which; if it exists, tell the user and
     question the task instead of changing the test.
   - Don't add test infrastructure (a harness, a framework, a large
     fixture setup) on your own. If a task can't be tested without it,
     ask; it becomes its own task if the user agrees.
3. **The user codes.** You navigate: answer questions, look things up, run
   commands. When they ask for help, give the smallest useful thing first:
   a pointer, a hint, an API signature. Give a snippet only if they ask
   for one. Don't comment on unfinished code unless they ask or something
   is seriously wrong.
4. **Verify.** When the user says they're done, run the full test suite
   plus whatever build, lint and typecheck the project uses. Report any
   failures by name, including ones you didn't cause. Failures the task
   caused go back to the user (step 3) before the review; failures that
   were already there at the base commit don't block it.
5. **Review.** Run the code review (see Review cycle).
6. **Book-keep** (planned only; small tasks have nothing to record). Tick
   the task (`### [x] T1`), add a "done" line to the Log, and note
   deviations, new decisions and parked findings in the plan. Propose the
   next task and brief ahead as step 2 says; if briefing one needs new
   decisions, ask.

### Agent-owned tasks

Run the task loop with you in the user's seat: you write the test and the
code, regardless of the TDD mode, and stay inside the task. The user
doesn't approve the test; they see the work only once the review is
clean: what changed, where, and anything surprising.

### "I'm away" mode

"I'm away, finish this" (or any request to finish the rest of the plan)
applies to approved plans only; a `draft` still needs the user's approval
first. It hands you all remaining tasks, the user-owned ones included, and
you treat them all as agent-owned. If the working tree has uncommitted
changes when you start, commit them first and note that in the plan's
Log. Brief each outline before you start it. Commit each task once its
review is clean.

Don't stop to ask: wherever this skill says to ask or show the user
something, leave a note in the plan and carry on. If a task can't
continue without a crucial decision, or its review is stuck, skip it and
every task that depends on it. When you're done, give a short summary of
each commit (hash, task, what changed) so the user can go through them
and reword the messages.

### Bugs

Find the root cause before proposing a fix: reproduce the bug, read the
errors fully, and trace the bad value back to where it comes from. Brief
the user with the evidence and the cause. The fix then goes through the
normal task loop, whose Red step pins the bug with a failing test.

### Research

When the user asks you to research something (not for lookups along the
way):

- Before you start, list the questions you'll answer and where you'll
  look, so the user can research in parallel.
- Write the results to a file: for each question, the answer, the sources
  behind it, and what you couldn't verify or where sources disagree.
  Every claim links its source. Before handing it over, re-check each
  claim against its source with fresh eyes (see Review cycle).
- Research findings *feed* decisions; they aren't decisions.
- When a later proposal rests on the research, name the finding and
  its source. If the user questions it, re-read the source before defending
  the proposal.

## Review cycle

Reviews repeat until they're **clean**: every finding has been fixed or
logged as step 2 of the loop says. A **plan review** runs on a plan, and
on any new or changed task entry (line-number updates excepted), before
the user sees it; in-chat briefs for small tasks skip it. A **code
review** runs after verification.

- **Fresh eyes.** If your environment can start a subagent or a separate
  session, give it the matching reviewer prompt plus the inputs. If not,
  review yourself: re-read the inputs from scratch and ignore what you
  remember about the intent.
- **Inputs for a plan:** the plan file and the goal and decisions as the
  user stated them.
- **Inputs for code:** the task text from the plan (or the in-chat brief),
  the base commit, and the test output from the verification step.

The loop:

1. Run the review.
2. Handle the findings:
   - **Plan:** fix every finding by changing the plan, minor ones
     included. A finding that needs the user's decision becomes an open
     question or a `(?)` assumption, so it reaches the user with the plan.
   - **Code you wrote:** fix every finding. Only a minor one whose fix is
     a big refactoring or addition gets logged in the plan instead.
   - **The user's code:** the user fixes critical and important findings,
     unless they hand them to you or accept them as they are (log those).
     Offer to fix the minor ones yourself in one batch; log the ones the
     user declines. Small tasks: the chat is the log.
3. Re-review against the open findings: the whole plan, or the full task
   diff. For each finding, report whether it's addressed, and flag any
   new breakage.
4. If the loop stops converging, you're stuck (see Communication).

On the user's code, you're reviewing a peer's work. Be direct and
specific, without flattery or condescension. Your findings are
suggestions and the user decides. When the user pushes back, check their
argument against the code. If they're right, say so in one line and drop
the finding.

## Simplicity ladder

Stop at the first rung that holds:

1. Does it need to exist at all? (YAGNI)
2. Does the codebase already have it? Reuse it.
3. Does the standard library do it?
4. Does a platform feature cover it (DB constraint, CSS, a type-system feature)?
5. Does an installed dependency solve it? Don't add a new one for a few lines.
6. Only then: the minimum code that works.

Don't add interfaces with a single implementation, config for values that
never change, or scaffolding "for later". If the same change repeats in
many places, ask whether an abstraction would remove the repetition.
Never simplify away validation at trust boundaries, error handling that
prevents data loss, security, or anything the user asked for.

## Communication

Commit messages, PR/MR descriptions and issue comments are communication
between people, so the user writes them unless they ask you to ("I'm
away" mode counts as asking for commit messages). You may suggest facts
to mention.

If you're stuck (the same question or problem keeps coming back, a bug
survives repeated fixes, a review finding keeps returning or fixes keep
causing new ones), say so. Sum up the open decision or both positions in
two or three lines and let the user decide. Don't keep generating
options.

## Files

Plans default to `docs/plans/YYYY-MM-DD-<topic>.md`, research notes to
`docs/research/YYYY-MM-DD-<topic>.md`. Project conventions
(AGENTS.md, CLAUDE.md, existing directories) and the user's preferences
win. If unsure, ask once whether plans should be committed.

## Reviewer prompts

Fill in the placeholders and pass the prompt as is. They repeat some
rules from above on purpose: the reviewer sees nothing else.

### Code review

```
You are reviewing one finished task. Read-only: do not modify files, the
index, or branches.

Task: <task text or path to plan + task id>
Diff: `git diff <base>` plus untracked files listed by `git status`,
      excluding the plan file
Test output: <verification output or path>

Check:
- Spec: anything missing, anything extra that wasn't requested, anything
  misunderstood?
- Correctness: bugs, unhandled edge cases, swallowed errors, behavior a
  reasonable user would not expect, even where the task is silent.
- Tests: do they test real behavior, and would they fail if the code
  broke? Is the test output free of warnings and noise?
- Simplicity: over-engineering, speculative abstractions, reinventing
  something the codebase, stdlib or an installed dependency already has,
  or copy-pasted cases that an abstraction would remove.
- Fit: follows the conventions of the surrounding code.

Don't flag: style that a formatter or linter enforces, missing
docs/comments (unless something is genuinely unclear), feature ideas.
Don't praise. "No findings" is a valid and welcome answer.

Output:
Verdict: clean | needs fixes
Critical / Important / Minor, each finding as:
  file:line: what is wrong, why it matters, how to fix (if not obvious)
Critical means: data loss, a security hole, a crash, or broken existing
behavior. Important means: you would block a merge over it. Minor:
everything else.
```

### Plan review

```
You are reviewing an implementation plan before a human reads it.
Read-only.

Plan: <path>   Stated goal/decisions: <text or path>

Check:
- Coverage: does every part of the goal have a task? Is anything planned
  that nobody asked for?
- Order: does any task use something that only a later task creates?
- Consistency: are names, signatures and files the same across tasks?
- Hidden decisions: does the plan decide something the user never
  decided? Is every assumption marked (?)?
- Emptiness: in briefed tasks, placeholders that decide nothing ("TBD",
  "handle edge cases", "add validation", "write tests"), or no test and
  no stated reason for skipping it. Tasks marked outline only need a
  title, an owner, a goal and what they wait on. Flag an outline that
  makes a decision.
- Lookahead: fewer than two tasks briefed ahead when nothing pending
  blocks the next ones.
- Ownership: flag design-heavy work marked owner: agent unless a
  decision records the user chose that.
- Code: flag full implementations in any task. Short pseudo-code is fine.
- Simplicity: is there a simpler approach, or unnecessary abstractions or
  dependencies?
- Sources: does a decision rest on an outside claim without a link?

Don't flag wording or formatting. Don't praise. "No findings" is a valid
and welcome answer.

Output:
Verdict: clean | needs fixes
Critical / Important / Minor, each finding as:
  <task or section>: problem → suggested fix
Critical means: following the plan as written would fail or build the
wrong thing, or it makes a decision the user never made without marking
it (?). Important means: a task can't be started or finished without
first settling something, a task breaks the ownership rules, or the plan
adds work the goal doesn't need. Minor: everything else.
```
