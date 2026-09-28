# Review cycle

Every artifact, whether a plan, your code or the user's code, gets reviewed
until the review is **clean**: no critical or important findings left open,
except ones the user explicitly accepted. Then it's fit for the user's
eyes, or done.

## How to run a review

- **Fresh eyes.** If your environment can start a subagent or a separate
  session, give it the matching prompt below plus the inputs. If not,
  review yourself: re-read the inputs from scratch and ignore what you
  remember about the intent.
- **Inputs for code:** the task text from the plan (or the in-chat brief),
  the base commit, and the test output from the verification step. The
  diff is `git diff <base>` plus untracked files from `git status`, since
  the user may not have committed yet.
- **Inputs for a plan:** the plan file, its research files, and the goal
  and decisions as the user stated them.

## The loop

1. Run the review.
2. Critical or important findings: fix them. The user fixes findings in
   their own code, unless they hand them to you. You fix findings in
   agent-owned code and in plans.
3. Minor findings: offer to fix them yourself in one batch. If the user
   declines, log them in the plan (small tasks: leave them in chat) and
   move on.
4. Re-review the full task diff against the open findings. For each
   finding, report whether it's addressed, and flag any new breakage.
5. If a finding is still disputed after three rounds, stop looping. Put
   both positions to the user in two lines and let them decide.

Tell the user the result briefly: a verdict and the findings. Don't paste
the whole review into chat when it's long; write it to a file and link it.

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
- Emptiness: "TBD", "handle edge cases", or a task with no test or no
  "done when".
- Ownership: is design-heavy work marked owner: agent? Does a
  user-owned task contain implementation code?
- Simplicity: is there a simpler approach, or unnecessary abstractions or
  dependencies?
- Research: does a decision rest on a research claim without a source?

Output: Verdict: clean | needs fixes, then findings as
  <task or section>: problem → suggested fix
```

## Reviewing the user's code

The user is a peer and the author. Be direct and specific, without
flattery or condescension. Your findings are suggestions and the user
decides. When the user pushes back, check their argument against the
code. If they're right, say so in one line and drop the finding.
