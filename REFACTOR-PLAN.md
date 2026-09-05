# guru — refactor plan

*How to move `guru-core` and `guru-app` from the goal-first product they implement today
onto the design in [`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md).*

**Read first:** [`README.md`](README.md) for the vocabulary,
[`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md) for the target.

## Contents

- [The collision](#the-collision) — where the names mean different things
- [What survives](#what-survives--reuse-rather-than-rewrite)
- [Method](#method--from-the-concept-canvas)
- [Stages](#stages) · [What makes this cheap](#what-makes-this-cheap) · [Trip hazards](#trip-hazards)

---

`guru-core` and `guru-app` today implement **a different product**: goal-first planning.
It is not an earlier spelling of this design, and the names collide.

## The collision

| Noun | Legacy meaning | Meaning in this design |
|---|---|---|
| **Role Model** | `role_models.kind = trait \| persona` — a pacing style, or a person to imitate | A life-shape template with a five-year path and a stated cost |
| **Profile** | `profiles{answers JSONB, timezone}` — questionnaire output | The normalized read of uploaded personal data; one per user |
| **Plan** | Generated from a required `goal` string, in three difficulty variants | Generated from a Direction Hypothesis; exactly one |
| **Milestone** | `Milestone{title, metric}` nested inside a `Phase`, flattened by the client into `milestone_title`/`milestone_metric` | A first-class tree node with its own table |
| **Task** | `plan_tasks`, keyed `(template_key, week_index, occurrence)` | A flat leaf under a Milestone |
| **Question** | Readiness follow-ups that **gate** plan generation | Constraint questions that set the Quota; always skippable |

The legacy model has **no** Report, Fit Verdict, Probe, Quota, Direction Hypothesis, or
reconciliation. More pointedly: `POST /v1/plan-sessions` requires a `goal`, and
`config/readiness_metrics.yaml` lists `goal_outcome` among its *required* metrics — the
system currently blocks on the exact question this product refuses to ask.

**This is a domain replacement, not a rename.**

## What survives — reuse rather than rewrite

The backend is well-kept: hexagonal, with five `.importlinter` contracts enforcing layer
direction, service independence, a thin `cmd/`, and vendor-free domain code. Most of the
machinery is worth keeping.

| Asset | Where | Becomes |
|---|---|---|
| 7 parsers + `Document{events, text_chunks}` | `packages/importers/` | The **Uploader** (step 1), unchanged |
| Google Calendar import, `oauth_connections` | `services/api/` | Intake source, unchanged |
| `schedule()` — deterministic placement | `services/plan_engine/domain/scheduler.py` | **Step 11**, unchanged arithmetic |
| `Capacity`, `TimeWindow`, `BusyBlock` | `plan_engine/domain/capacity.py` | Scheduler input; **not** the Quota |
| Whole LLM stack + `llm_calls` observability | `packages/llm/` | Every model call in the new pipeline |
| State machine + `assert_transition` | `plan_engine/domain/session.py` | The Station-1 run state machine |
| `diff_tasks` | `plan_engine/domain/diff.py` | Station 3's comparison |
| `postpone` / `reduce` strategies | `plan_engine/domain/revision.py` | Growth-driven vs avoidance-driven classification |
| Tag vocabulary | `role_model/domain/tags.py`, `config/tag_vocab.yaml` | Report dimensions, Role Model tags |
| Queue, storage, cache, config loaders, logging | `packages/` | Unchanged |

**Dropped:** `difficulty.py`'s three plan variants (one hypothesis produces one plan), and
`readiness_metrics.yaml`'s goal-gating.

## Method — from the concept canvas

The same canvas carries a *Working with legacy project* decision tree. The staging below
follows it rather than inventing a process:

- *Change existing behaviour?* → **No, adding a new feature** → **wrap** the existing
  function/class with a new one.
- *Change existing behaviour?* → **Yes** → **sprout** the new function/class from the
  existing expression.
- *Able to break all dependencies?* → **No** → find a **high-level interception** and write
  an integration test there. → **Yes** → mock the API/third party behind a defined
  interface.
- *Don't know what test to write?* → **characterization test** → sketch the effects between
  object operations → boundary conditions.

## Stages

**Stage 0 — pin the behaviour.**
Characterization tests at the highest useful interception: the `/v1` HTTP boundary and
`guru-app/app/lib/guru-api.ts`. The dependencies here are not all breakable, so this is
deliberately an integration-level net, not a unit one. `guru-app/tests/guru-api-contract.test.mjs`
is the existing seam to extend. **No schema changes in this stage.**

**Stage 1 — sprout the new contexts.**
`intake`, `analysis` and `direction` are new behaviour, so they are *sprouted*: new modules
and new tables alongside the untouched goal-first pipeline. Nothing in `plan_sessions`
changes. New domain code must stay vendor-free and must not import across services — the
existing `.importlinter` contracts already enforce both, and should be extended to cover the
new packages rather than relaxed.

**Stage 2 — wrap the Plan Engine.**
The scheduler is the most valuable thing in the repository and should not be rewritten.
*Wrap* it behind a port that accepts a Direction Hypothesis and a Milestone tree instead of
a goal string. Add the `milestones` table, point `tasks.milestone_id` at it. The placement
arithmetic inside `schedule()` does not change.

**Stage 3 — rename, under protection, last.**
Only once Stage 0's net holds: legacy `role_models` (trait/persona) →
`execution_styles` / `persona_references`, freeing the name `RoleModel` for the life-shape
template. This is the highest-risk change in the whole refactor — it touches
`seeds/role_models/*.yaml`, the entire `services/role_model` service, and every
`GET /v1/role-models*` route. **It must not be first.**

**Stage 4 — retire.**
Delete the goal-first entrypoints, the `goal_outcome` gate, the three difficulty variants,
and the dead scaffolding: `guru-app/app/chatgpt-auth.ts` (unimported), `db/`, `drizzle/`,
`examples/d1/` (all inert — the D1/R2 bindings are `null`).

## What makes this cheap

`guru-core` has **exactly one migration**, and it has never been evolved. Squashing to a new
initial schema is a live option rather than a fantasy. Treat that as a decision to make
explicitly — the alternative is a long chain of rename migrations for a schema no production
data depends on yet.

## Trip hazards

| Hazard | Where | Consequence |
|---|---|---|
| Han-character allowlist is hard-coded to three files | `guru-app/scripts/check-language.mjs` | Splitting `GuruApp.tsx` (540 lines, ~25 `useState`, 10 modals, 3 polling loops) fails `npm test` until the list is updated |
| Three Chinese strings are pinned by an SSR smoke test | `guru-app/tests/guru-html.test.mjs` | Any restructure must preserve them |
| `Pacing` and `FollowupQuestion` are **deliberately duplicated** across services | `plan_engine/domain/difficulty.py` ↔ `role_model/domain/content.py` | Bound only by JSON contract, because the `services-independent` import contract forbids sharing. The most fragile seam in the codebase — do not "fix" it by importing across services |
| `ALLOW_FAKE_LOGIN` dev bypass | `services/api/settings.py` | Signs in as any user when set; the refactor touches auth-adjacent code |
| `services/api/adapters/role_model_client.py` HTTP-proxies to the catalog service | | Satisfies the import contract but violates the older PRD's "no HTTP between services" rule. Decide which rule stands before Stage 3 |

---

## Open questions

Modelling questions that are still open are tracked in
[SYSTEM-DESIGN · Open questions](SYSTEM-DESIGN.md#open-questions). Two of them —
whether user-authored Role Models share a table with the shipped six, and whether the
Probe is its own object or the first Milestone — change the Stage 1 schema, so settle
them before sprouting the new tables.

## See also

| | |
|---|---|
| [`README.md`](README.md) · [`README.zh-TW.md`](README.zh-TW.md) | What the system is and what it is responsible for (English / 繁體中文) |
| [`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md) | The target design this plan migrates toward |
