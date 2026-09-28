# Test-driven development

If you never watched a test fail, you don't know whether it tests the
right thing.

## Modes (agree on one per plan or small task)

- **user:** the user writes the tests and the code. You propose the test
  cases in the brief.
- **ping-pong:** you write the failing test, and the user makes it pass.
  The test is the spec for their task, so show it to them and get their
  agreement before they start. Keep it minimal, and don't write helpers
  that already implement part of the logic.
- **per-task:** the user picks the mode for each task during the brief.
  Suggest one with a one-line reason.

## The cycle

1. **Red.** Write one test for one behavior. Give it a name that describes
   the behavior. Test real code; use mocks only when a real dependency
   can't be used. Before writing it, name the production change that
   would make it fail.
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

## Exceptions

Throwaway spikes, generated code, pure configuration, and places with no
test harness. Ask before skipping a test; don't skip silently. If the
project has no test setup, propose the smallest possible one as its own
task. Test behavior, not every function. Trivial glue needs no test of
its own.
