# Tasks

> **Phase gate:** each numbered group below is a phase. A phase ends with three tasks, in order:
> (a) **UI review** — when the phase adds or changes screens: screenshot each one next to the
> reference screen named in design.md's *Screen standards* and check the same columns
> (breadcrumbs, lazy tabs, in-place updates, stats cards, filters, actions menu, tooltips/help,
> width); (b) **N+1 check** — run the phase's tests with the project's N+1 detector enabled and
> fix every N+1 in this change's code; (c) run the phase's acceptance tests green. Do not start
> the next phase until all three pass — fix the phase you're on rather than coming back later.

> `<test-cmd>` below is the project's acceptance test command, from the project context in
> `openspec/config.yaml`. Respect any per-run requirements listed there.

## 1. <!-- Task Group Name -->

- [ ] 1.1 <!-- Task description -->
- [ ] 1.2 <!-- Task description -->
- [ ] 1.3 Write `tests/acceptance/<domain>/<phase_1_name>` covering this phase's worked examples
      (happy path + edge/error cases from design.md, or business-rules.md when the change has one).
- [ ] 1.4 UI review (if this phase adds/changes screens): screenshots of each screen beside its
      reference screen from design.md *Screen standards*; fix anything that differs.
- [ ] 1.5 N+1 check: run `<test-cmd> tests/acceptance/<domain>/<phase_1_name>` with the N+1
      detector enabled, read its output, and fix every N+1 whose stack is in this change's code.
- [ ] 1.6 Run `<test-cmd> tests/acceptance/<domain>/<phase_1_name>` (wait on the run's PID, never
      a pgrep loop) and confirm it passes before starting phase 2.

## 2. <!-- Task Group Name -->

- [ ] 2.1 <!-- Task description -->
- [ ] 2.2 <!-- Task description -->
- [ ] 2.3 Write `tests/acceptance/<domain>/<phase_2_name>` covering this phase's worked examples.
- [ ] 2.4 UI review (if screens changed) against design.md *Screen standards*.
- [ ] 2.5 N+1 check with the N+1 detector; fix every N+1 in this change's code.
- [ ] 2.6 Run the phase's acceptance tests green before starting phase 3.

<!-- Add further numbered phases as needed, each ending with the same gate — test file, UI review
     (or "no screens changed"), N+1 check, green run — before the next phase starts. -->
