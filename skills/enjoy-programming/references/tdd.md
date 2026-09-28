# Test-driven development

If you never watched a test fail, you don't know whether it tests the
right thing.

## Modes (one per plan or small task)

- **ping-pong (default):** you write the failing test, and the user makes
  it pass. The test is the spec for their task, so show it to them and
  get their agreement before they start. Keep it minimal, and don't write
  helpers that already implement part of the logic.
- **user:** the user writes the tests and the code. You propose the test
  cases in the brief.
- **per-task:** the user picks the mode for each task during the brief.
  Suggest one with a one-line reason.

On agent-owned tasks, and on every task in "I'm away" mode, you write the
tests regardless of the mode.

## The cycle

1. **Red.** Write one test for one behavior. Give it a name that describes
   the behavior. Test real code; use mocks only when a real dependency
   can't be used.
2. **Verify red.** Run it. It has to *fail*, not error out, and fail
   because the behavior is missing, not because of a typo or a missing
   import. If it passes right away, either the test is wrong or the
   behavior already exists. Find out which; if it exists, tell the user
   and question the task instead of changing the test.
3. **Green.** Write the minimal code that makes the test pass. For a
   user-owned task, the user writes this.
4. **Verify green.** Run the full project suite, not only the new test.
   The output should be clean, with no new warnings. Report any failures
   by name, including ones you didn't cause.
5. **Refactor.** Only while the tests are green, and without adding
   behavior.

## Skipping tests

Test behavior, not every function. Throwaway spikes, generated code, pure
configuration and trivial glue need no test of their own. Don't skip
silently: ask, or state the reason in the plan or brief (that counts as
asking).

Don't add test infrastructure (a harness, a framework, a large fixture
setup) on your own. If a task can't be tested without it, ask the user;
it becomes its own task if they agree. In "I'm away" mode, test wherever
a framework already exists; elsewhere, leave a note in the plan that the
task couldn't be tested and what that would take.
