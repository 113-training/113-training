---

name: fix-bug
description: Fix a bug using the standard workflow: reproduce, investigate, fix, regression test, and commit. Use only when the user explicitly asks to fix a bug.
------------------------------------------------------------------------------------------------------------------------------------------------------------------

Fix the bug described by the user by following this workflow. The reported symptoms are provided in the user's message:

1. Based on the reported symptoms, identify the likely affected page and workflow, then confirm your understanding of the issue with the user.
2. Trace the relevant flow from the Controller through the Service and Repository to identify the root cause. Explain the root cause and **wait for the user's confirmation** before making any changes.
3. Apply the smallest possible fix. Do not refactor unrelated code as part of the change.
4. Use `code-reviewer` to review and validate the changes.
5. Add a regression test. First verify that the test logic would fail before the fix, then use `test-runner` to run `dotnet test` and confirm that all tests pass.
6. Ask the user to verify the fix manually through the affected page. After the user confirms the fix, write the commit message using the format **“Symptom → Root Cause → Fix”**, then create the commit.
