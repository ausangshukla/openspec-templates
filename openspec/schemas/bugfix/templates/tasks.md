# Tasks

> **Reproduce first.** Phase 1 writes the acceptance test scenarios from the report's
> Verification table that fail against the CURRENT code, and runs them to watch them fail. A bug
> fix that was never seen failing might be fixing nothing.

> **Phase gate:** each numbered group below is a phase. A phase's last task is always to run its
> acceptance tests and get them green. Do not start the next phase until that happens.

> `<test-cmd>` below is the project's acceptance test command, from the project context in
> `openspec/config.yaml`. Respect any per-run requirements listed there.

## 1. Reproduce

- [ ] 1.1 Write `tests/acceptance/<path>/<test>` covering <!-- H/E/X IDs --> from the report's
      Verification table. Run `<test-cmd> tests/acceptance/<path>/<test>` and confirm the rows
      listed as "Fails against current code" do fail. Record the failure output.

## 2. Fix

- [ ] 2.1 <!-- Task description — name the report's Fix row it implements -->
- [ ] 2.2 <!-- Task description, with how to verify it -->
- [ ] 2.3 Run `<test-cmd> tests/acceptance/<path>/<test>` and confirm <!-- IDs --> now pass
      before starting phase 3.

<!-- Add further numbered phases only if the fix genuinely has separable parts, each ending in
     its own test file + a green run before the next phase starts. -->

## 3. Regression and cleanup

- [ ] 3.1 Run the test directory the fix touches: `<test-cmd> tests/acceptance/<dir>` and confirm
      it is green.
- [ ] 3.2 Run the tests of every caller listed in the report's Blast Radius that is marked as
      needing checking, and confirm they are green.
- [ ] 3.3 Run the project's linter on the changed files and confirm it is clean, including any
      custom project rules.
- [ ] 3.4 N+1 check (if the fix touches a query, a list page or a job): run the tests with the
      project's N+1 detector enabled, read its output and fix any N+1 the fix introduced.
- [ ] 3.5 UI check (if the fix touches a screen): screenshot it beside its unchanged neighbours and
      confirm it still matches the app's standard screens (breadcrumbs, tabs, buttons, tooltips).
      Otherwise write "no screens touched".
