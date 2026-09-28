# Review cycle

Reviews repeat until they're **clean**: no critical or important findings
left open. There are two kinds:

- **Plan review** runs on a new plan, or on a brief that changed beyond
  line numbers, before the user sees it. You fix every finding yourself.
  A finding that needs the user's decision becomes an open question or a
  `(?)` assumption in the plan, so it reaches the user with the plan.
- **Code review** runs on a finished task after verification, whether
  you or the user wrote the code. Findings in the user's code go to the
  user, who may also accept a finding and leave it open.

## How to run a review

- **Fresh eyes.** If your environment can start a subagent or a separate
  session, give it the matching prompt below plus the inputs. If not,
  review yourself: re-read the inputs from scratch and ignore what you
  remember about the intent.
- **Inputs for a plan:** the plan file, its research files if any, and
  the goal and decisions as the user stated them.
- **Inputs for code:** the task text from the plan (or the in-chat brief),
  the base commit (from the plan's Log, or the chat for small tasks), and
  the test output from the verification step. The diff is
  `git diff <base>` plus untracked files from `git status`, since the
  user may not have committed yet.

## The loop

1. Run the review.
2. Critical or important findings: fix them. You fix findings in plans
   and in agent-owned code. The user fixes findings in their own code,
   unless they hand them to you.
3. Minor findings: in plans and agent-owned code, fix the quick, trivial
   ones and log the more involved ones (e.g. a fix that adds many lines)
   in the plan. In the user's code, offer to fix them yourself in one
   batch; if the user declines, log them. Small tasks: leave them in chat.
4. Re-review against the open findings: the whole plan, or the full task
   diff. For each finding, report whether it's addressed, and flag any
   new breakage.
5. If a finding is still disputed after three rounds, stop looping. Put
   both positions to the user in two lines and let them decide.

Tell the user the verdict and the findings.

## Code review prompt

```
You are reviewing one finished task. Read-only: do not modify files, the
index, or branches.

Task: <task text or path to plan + task id>
Diff: `git diff <base>` plus untracked files listed by `git status`
Test output: <verification output or path>

Check:
- Spec: anything missing, anything extra that wasn't requested, anything
  misunderstood?
- Correctness: bugs, unhandled edge cases, swallowed errors, behavior a
  reasonable user would not expect, even where the task is silent.
- Tests: do they test real behavior, and would they fail if the code
  broke? Is the output clean?
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
behavior. Important means: you would block a merge over it.
```

## Plan review prompt

```
You are reviewing an implementation plan before a human reads it.
Read-only.

Plan: <path>   Research: <paths>   Stated goal/decisions: <text or path>

Check:
- Coverage: does every part of the goal have a task? Is anything planned
  that nobody asked for?
- Order: does any task use something that only a later task creates?
- Consistency: are names, signatures and files the same across tasks?
- Hidden decisions: does the plan decide something the user never
  decided? Is every assumption marked (?)?
- Emptiness: in briefed tasks, "TBD", "handle edge cases", no "done
  when", or no test and no stated reason for skipping it (config, glue,
  no test framework). Tasks marked outline only need a goal and what they
  wait on. Flag an outline that makes a decision, and fewer than two
  tasks briefed ahead when nothing pending blocks the next ones.
- Ownership: flag design-heavy work marked owner: agent unless a
  decision records the user chose that, and user-owned tasks that contain
  implementation code.
- Simplicity: is there a simpler approach, or unnecessary abstractions or
  dependencies?
- Research: does a decision rest on a research claim without a source?

Don't flag wording or formatting. Don't praise. "No findings" is a valid
and welcome answer.

Output:
Verdict: clean | needs fixes
Critical / Important / Minor, each finding as:
  <task or section>: problem → suggested fix
Critical means: following the plan as written would fail or build the
wrong thing (e.g., a part of the goal is not covered by any task, a task
that needs something only a later task creates, a contradiction with a
stated decision, or a decision the user never made and that isn't marked
`(?)`). Important means: a task can't be started or finished without
first settling something (e.g., inconsistent names or signatures, an
empty or untestable task, unknown facts), a task breaks the ownership
rules above, or the plan adds work the goal doesn't need.
Minor: everything else.
```

## Reviewing the user's code

The user is a peer and the author. Be direct and specific, without
flattery or condescension. Your findings are suggestions and the user
decides. When the user pushes back, check their argument against the
code. If they're right, say so in one line and drop the finding.
