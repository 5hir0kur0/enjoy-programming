# Test-driven development

## Modes (`tdd-mode`, one per plan or small task)

- **ping-pong (default):** you write the failing test, and the user makes
  it pass. The test is the spec for their task, so show it to them and
  get their agreement before they start. Keep it minimal, and don't write
  helpers that already implement part of the logic.
- **user:** the user writes the tests and the code. You propose the test
  cases in the brief.

On agent-owned tasks, and on every task in "I'm away" mode, you write the
tests regardless of the mode.

## Writing the test

- One test for one behavior, with a name that describes the behavior.
- Test real code; use mocks only when a real dependency can't be used.
- When you run it, it has to *fail*, not error out, and fail because the
  behavior is missing, not because of a typo or a missing import.
- If it passes right away, either the test is wrong or the behavior
  already exists. Find out which; if it exists, tell the user and question
  the task instead of changing the test.
- Refactor only while the tests are green, and without adding behavior.

## Skipping tests

Test behavior, not every function. Throwaway spikes, generated code, pure
configuration and trivial glue need no test of their own. Don't skip
silently: state the reason in the brief. If the user doesn't object, skip.

Don't add test infrastructure (a harness, a framework, a large fixture
setup) on your own. If a task can't be tested without it, ask the user;
it becomes its own task if they agree. In "I'm away" mode, don't add
infrastructure; note what testing the task would take.
