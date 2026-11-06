# [Feature Name] — Low-Level Design

<!-- STYLE: bullets and diagrams, not prose. One idea per bullet; a diagram per concept (model, lifecycle, flow); table cells a phrase each — longer reasoning goes in bullets under the table. -->

> **Audience:** The developer who reviews the implementation plan before coding starts.
> **Companion documents:** [Design](design.md) · [Spec](specs/) · [Tasks](tasks.md)

> **Planning phase — no code changes.** This document records the implementation-level decisions
> a reviewing developer must check. It adds to [design.md](design.md): link to its diagrams and
> worked examples rather than repeating them.

> **Reviewable, not descriptive.** Every section states concrete decisions (types, limits,
> indexes, locks, retries, error mappings) and ends with **Reviewer checks** to tick. Short
> snippets (a signature, a query shape, a config line) are fine. Full class bodies and full
> migrations are not.

> **Scale to the change.** **CORE** sections are always present. **CONDITIONAL** sections say when
> they apply. Otherwise keep the heading with `N/A — <reason>`.

---

## Contents

<!-- CORE. Links to every ## and ### heading, in order. Update it whenever headings change. -->

- [Review Summary](#review-summary)
- [File Map](#file-map)
- [Schema Detail](#schema-detail)
- [Method Contracts](#method-contracts)
- [Controllers, Params and Responses](#controllers-params-and-responses)
- [Queries and Indexes](#queries-and-indexes)
- [Transactions, Locking and Concurrency](#transactions-locking-and-concurrency)
- [Background Jobs](#background-jobs)
- [External Integrations](#external-integrations)
- [Error Mapping and Logging](#error-mapping-and-logging)
- [Security Review](#security-review)
- [Configuration and Rollout](#configuration-and-rollout)
- [Test Plumbing](#test-plumbing)
- [Project Rules Checklist](#project-rules-checklist)
- [Sign-off](#sign-off)

---

## Review Summary

<!-- CORE. 3-6 bullets: the decisions most likely to be wrong or contested, each linking to its
     section below. This is where the reviewer starts. -->

- <!-- e.g. Keyset batching on X instead of offset — see Queries and Indexes -->

---

## File Map

<!-- CORE. Every new and modified file, grouped by the design's numbered boxes (①②③). Tasks
     reference these rows. -->

| Boundary | File | New / Modified | Responsibility |
|---|---|---|---|
| <!-- UI --> | <!-- `<module>/<path>/x_controller.<ext>` --> | <!-- New --> | <!-- one line --> |

**Reviewer checks**
- [ ] Every class in the design's class diagram has a file here, and nothing here is missing from it
- [ ] Files follow the project's module layout and respect its module boundaries

---

## Schema Detail

<!-- CONDITIONAL: include when tables or columns change. Column-level detail the ER diagram
     omits. One table per database table, opened by one sentence on why the table changes. Every
     row says what changed (Added / Changed from X / New index) and why it is needed; the design's
     Schema changes block holds the business reason, this one the type-level reason (why this
     type, limit, null and default). -->

**`table_name`** (<!-- new / existing -->): <!-- why this table changes, one sentence -->

| Column | Change | Type | Limit | Null | Default | Index / FK | Why it's needed |
|---|---|---|---|---|---|---|---|
| <!-- `status` --> | <!-- Added --> | <!-- string --> | <!-- 20 --> | <!-- no --> | <!-- "draft" --> | <!-- composite idx --> | <!-- why this column, and why this type/limit/default --> |

**Migration safety:** <!-- table size in prod, lock/DDL impact, backfill needed?, reversible?, order
of migrations; every checked-in schema dump updated -->

**Reviewer checks**
- [ ] Every new or changed column and index has a "why", and none is unjustified
- [ ] Every foreign key has an index, and every uniqueness rule is enforced by a unique index
- [ ] Column limits match validation limits
- [ ] Migration is reversible and safe on the production table size

---

## Method Contracts

<!-- CORE. One table per class from the design's class diagram: the public methods other
     classes call. -->

**`ClassName`** (<!-- Service / Model / Job -->)

| Method | Arguments | Returns | Raises | Side effects | Idempotent? |
|---|---|---|---|---|---|
| <!-- `.call(record, user:)` --> | <!-- types --> | <!-- Result / record --> | <!-- errors --> | <!-- writes, enqueues, emails --> | <!-- Yes / No, and why --> |

**Reviewer checks**
- [ ] Every raised error is handled by a caller or listed in Error Mapping
- [ ] Anything a retry can repeat is idempotent

---

## Controllers, Params and Responses

<!-- CONDITIONAL: include when controller actions are added. Extends the design's route table
     with the implementation detail. -->

| Action | Permitted params | Policy method | Success response | Failure response |
|---|---|---|---|---|
| <!-- `ThingsController#create` --> | <!-- `thing: [:name, :kind]` --> | <!-- `create?` --> | <!-- redirect / partial update --> | <!-- 422 + form errors --> |

**Reviewer checks**
- [ ] No action without an authorization check and a scoped load
- [ ] Permitted params can't set the tenant id, `user_id` or other ownership fields
- [ ] Actions that change one row or card update just that row/card in place (and refresh cards and
      flash) with a full-page redirect fallback, as design.md *Screen standards* says
- [ ] Every page action sets breadcrumbs; tab panes and lazy-loaded partials render only their own
      fragment when requested as a fragment

---

## Queries and Indexes

<!-- CONDITIONAL: include when the feature reads the database. The actual query shape, with the
     index it relies on and the expected row count. -->

| Where | Query shape | Index used | Rows expected | Preload / batching |
|---|---|---|---|---|
| <!-- `Svc#call` --> | <!-- `Thing.where(tenant_id:, status:)` --> | <!-- `[tenant_id, status]` --> | <!-- ≤ 30 --> | <!-- eager-load parent / batched iteration --> |

**Reviewer checks**
- [ ] Every query is scoped by tenant, or explains why not
- [ ] No N+1 on list pages. No unbounded loads (iterate in batches)
- [ ] Every page / frame / job from design.md's **Query budget** has its preloads and grouped
      queries named here; nothing inside a row loop issues a query (exists, count, find, a
      querying policy check, an un-preloaded association)
- [ ] The phase's tests run clean under the project's N+1 detector for this change's code

---

## Transactions, Locking and Concurrency

<!-- CONDITIONAL: include for multi-row writes or anything two processes can do at once. -->

| Operation | Transaction scope | Lock | Race it prevents | Idempotency key |
|---|---|---|---|---|
| <!-- submit order --> | <!-- order + line items --> | <!-- row lock on the order --> | <!-- double-submit --> | <!-- unique index on X --> |

**Reviewer checks**
- [ ] No external HTTP call or email inside a transaction
- [ ] Lock order is consistent everywhere two rows are locked

---

## Background Jobs

<!-- CONDITIONAL: include when jobs or cron entries are added. -->

| Job | Queue | Trigger | Retry / backoff | Discard on | Concurrency guard | Timeout |
|---|---|---|---|---|---|---|
| <!-- `ThingJob` --> | <!-- `:low` --> | <!-- cron every 15 min --> | <!-- 5, polynomial --> | <!-- `RecordNotFound` --> | <!-- claim column --> | <!-- --> |

**Reviewer checks**
- [ ] A job that runs twice, or out of order, is harmless
- [ ] Scheduled entries are registered in the scheduler config that actually runs in each environment

---

## External Integrations

<!-- CONDITIONAL: include when calling a third-party API. -->

| Call | Method + path | Auth | Timeouts | Rate limit handling | Error → exception |
|---|---|---|---|---|---|
| <!-- list items --> | <!-- `GET /v1/items` --> | <!-- Bearer key --> | <!-- 10s / 30s --> | <!-- 429 Retry-After --> | <!-- 401 → AuthError --> |

**Reviewer checks**
- [ ] Secrets never logged, and the request/response logging level is appropriate
- [ ] Every non-2xx status maps to a named exception

---

## Error Mapping and Logging

<!-- CORE. Technical side of the design's Exception Handling table: exception → handling → what
     the user sees → log/alert. -->

| Exception | Raised in | Handled by | User sees | Logged / alerted |
|---|---|---|---|---|
| <!-- `AuthError` --> | <!-- client --> | <!-- connection failure handler --> | <!-- needs attention --> | <!-- warn log + notifier --> |

**Reviewer checks**
- [ ] No bare `rescue => e` that swallows errors silently
- [ ] Log lines carry the ids needed to debug (tenant, record, external id)

---

## Security Review

<!-- CORE. -->

| Concern | Decision |
|---|---|
| Authorization | <!-- policy per action, scope rule --> |
| Tenant isolation | <!-- how every read/write is tenant-scoped --> |
| Secrets | <!-- storage (encrypts?), display masking, log filtering --> |
| Input / output sanitisation | <!-- HTML escaping, markdown rendering, file uploads --> |
| Mass assignment | <!-- permitted params --> |

**Reviewer checks**
- [ ] A user from another tenant cannot reach any record through any route, job, API or tool

---

## Configuration and Rollout

<!-- CORE. -->

| Item | Value / location |
|---|---|
| Credentials / env vars | <!-- per environment; never shared between environments or regions --> |
| Feature flag | <!-- if any --> |
| Cron / schedules | <!-- --> |
| Deploy order | <!-- migrations → code → cron --> |
| Rollback | <!-- how to turn it off, and whether data must be cleaned up --> |

**Reviewer checks**
- [ ] Safe to deploy with the feature unused, and safe to roll back

---

## Test Plumbing

<!-- CORE. What the acceptance tests in design.md's Worked examples (Part 4) need to run. -->

| Test file | Stubs / fixtures | Factories | New steps / helpers |
|---|---|---|---|
| <!-- `tests/acceptance/<domain>/x` --> | <!-- HTTP stub helper --> | <!-- `:thing` --> | <!-- `Given a connected ...` --> |

---


## Pattern Conformance

<!-- CORE. For each new class: its pattern (operation / service / job / policy / component /
     decorator / mixin), its module, its public interface, and which existing class it is modelled
     on. Reviewer checks that no class mixes roles (e.g. a controller doing business logic). -->

| Class | Pattern | Module | Public interface | Modelled on |
|---|---|---|---|---|

## Security Detail

<!-- CORE. One row per new controller action, job, AI call, upload and token: the policy method,
     the scope used to load the record, permitted params, tenant/parent-id checks, and the
     security test scenario that covers it. Mirrors design.md › Security. -->

| Action / job / call | Policy method | Loaded through | Permitted params | Tenant checks | Security scenario |
|---|---|---|---|---|---|

## Project Rules Checklist

<!-- CORE. This repo's standing rules: the ones listed in the project context (openspec/config.yaml)
     and in the repo's agent / contributor docs (CLAUDE.md, AGENTS.md, CONTRIBUTING.md). Mark each
     Applies / N/A with a one-line reason. The rows below are common starting points: delete the
     ones that don't fit and add the project's own. -->

| Rule | Applies? | How it is met |
|---|---|---|
| Every query tenant-scoped; every action authorized through the policy layer | | |
| No unbounded loads of large tables; bulk operations batched; only needed columns loaded from wide tables | | |
| Dedup and aggregate in SQL, not in application code | | |
| Dates, times and money rendered through the app's locale/format helpers, never hand-written formats | | |
| Date inputs use the app's standard date input component | | |
| Acceptance tests run with the project's test command and per-run requirements | | |
| Linter / formatter clean on changed files, including any custom project cops/rules | | |
| No query inside a row loop (per-row exists/count/find, querying policy checks, un-preloaded associations); access checks and nav/card counts once per request; the phase's tests clean under the N+1 detector | | |
| Screens copy the reference screen named in design.md *Screen standards*: same layout, tabs, actions menu, help as tooltips (never grey paragraphs), no new CSS | | |
| <!-- add the project's own standing rules --> | | |

---

## Sign-off

| Reviewer | Date | Decision | Notes |
|---|---|---|---|
| | | <!-- Approved / Changes requested --> | |
