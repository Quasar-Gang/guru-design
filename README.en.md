<div align="center">

# Life Guru

**English** (this page) · [繁體中文](README.md)

**Give life goals a basis, and give every day a direction.**

A personal-growth AI coach for people who want to grow but struggle to keep going — it
finds a direction worth investing in, turns it into a plan that fits their actual ability
and time, and adjusts the pace through continuous feedback.

![Status](https://img.shields.io/badge/status-design-D97706)
[![Prototype](https://img.shields.io/badge/prototype-live-0F9D58)](https://wu0h9625-boop.github.io/guru-intake-prototype/)
![License](https://img.shields.io/badge/license-proprietary-A31515)

[Backend `backend/`](backend) · [Frontend `frontend/`](frontend) · [Prototype](https://wu0h9625-boop.github.io/guru-intake-prototype/) · [Video](https://www.youtube.com/watch?v=vP_6S0C3R7Q)

</div>

---

## Problem and goal

Life Guru exists to reduce two costs:

| | Cost | What it looks like |
|---|---|---|
| **1** | **Not knowing the direction** | Planning over and over, never starting, running in place |
| **2** | **The plan breaking down** | Starting, then stopping because the plan doesn't fit how you actually live |

**The target user** wants to grow but struggles to sustain action — they have a job, a
calendar and a résumé, they know something is off, but they cannot say where they want to
go and have no standing to write a five-year plan.

The root of it: ***"what is your vision?"* asks someone to invent something from nothing.**
Every goal product on the market starts there. For the minority who already know, that
works. For everyone else it is a wall on screen one.

So Life Guru does not begin by asking. It **begins from the data the user has already
left**, shows them their current situation first, then uses a short set of questions to
fill in what the data cannot explain. Together those become the basis for planning, and
**spare the user from having to organise themselves from scratch**.

> **The product's core principle:** every recommendation should tell the user
> **why it is worth trying, what it will cost, and how to start today.**

**Intended impact:** turn "I don't know what I want" into an answer you can walk forward
from, and let a plan survive changes in your life instead of breaking.

## Core features

Two flows, one for each cost above.

**1 · Explore role model — for "I don't know the direction"**

- **Start from what already exists** — import Google Calendar and a résumé PDF; no recall, no new tracking
- **See the current situation first** — build one Profile per user and produce multi-dimension Reports (work / exercise / social / learning / capacity); unclassified time is a first-class result
- **Short questions fill the gaps** — what the data cannot explain: conditions the user won't accept, and how much time they're willing to give; every question is skippable
- **Compare ways of living and working through Role Models** — each direction shows **the capabilities to accumulate, the investment and the trade-offs**, cross-checked against personal data to say **what foundations already exist and what is missing**
- **The user keeps the final say** — the reasoning is laid out, and the user can supplement or correct the AI's judgement

**2 · Review task progress — for "the plan broke down"**

- **Break the direction into milestones and concrete tasks** — Milestones nest; Tasks stay single-level beneath one
- **Schedule daily action against available time** — placed on real dates, respecting how much the user is willing to commit
- **Completion records and periodic review** — the Reviewer reads task progress weekly, monthly or quarterly
- **Diagnose what is actually blocking** — whether the task was **too big**, the **time insufficient**, or the **goal itself needs adjusting**
- **Replan** — when the direction needs to change, analysis and recommendation re-run; once the user picks a new Role Model, the Plan Engine rebuilds Milestones, Tasks and the schedule

> **The design decision that matters:** the Recommender never sees the raw Profile — it reads
> the multi-dimension Reports. Borrowed from chain-of-thought: producing intermediate,
> inspectable evidence before reasoning improves precision, and makes every recommendation
> something the user can **argue with** — which is how "the user keeps the final say" is
> actually implemented.

## System architecture

### Flow 1 · Explore role model

<img src="assets/explore_role_model.png" alt="Explore role model flow: the User uploads personal data to the Uploader, which builds one Profile; the Analyzer reads it and creates Reports across dimensions; the Recommender reads those Reports and recommends six Role Model templates; the User selects one and the Plan Engine sets up Milestones, creates Tasks and schedules them">

### Flow 2 · Review task progress

<img src="assets/review_task_progress.png" alt="Review task progress flow: the Reviewer periodically reads Task progress and, when it falls under the threshold, triggers the Analyzer to re-analyze, regenerating Reports and Role Model recommendations; after the User selects a new role, the Plan Engine sets up new Milestones, creates Tasks and reschedules">

### How the layers collaborate

| Layer | Component | Responsibility |
|---|---|---|
| **Frontend** | Web app ([`frontend/`](frontend)) | Upload UI, Report views, Role Model selection, daily tasks and progress check-in |
| **Backend · API** | API Service | Auth, OAuth, upload and parsing, every app-facing endpoint, job dispatch |
| **Backend · planning** | Plan Engine | Milestone tree, Task generation, **deterministic** scheduling and rescheduling |
| **Backend · review** | Reviewer | Reads task progress on a period, diagnoses what is blocking, and re-triggers analysis when needed |
| **Backend · recommendation** | Role Model Service | Role Model template queries and LLM-backed recommendation |
| **AI model** | LLM (xAI Grok) | Judgement only: analyze Reports, recommend Role Models and their reasoning, produce the plan template |
| **Database** | PostgreSQL · Redis | All state in the former; queues and cache in the latter |
| **External services** | Google Calendar · Cloudflare R2 | Calendar import and export, uploaded-file storage |

**The LLM does judgement, never arithmetic.** Scheduling, time maths and progress
comparison are deterministic code — the same inputs must give the same result, or "was this
blocked by the task being too big or by not enough time?" is a comparison against noise.

### Going deeper

This README stops at why the system is shaped this way and what it looks like. The full
specification underneath it is **[`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md)** — the domain model,
the invariants, the queue jobs and the LLM boundary all live there.

| What you want | Where to read it |
|---|---|
| The two flows, step by step | [`SYSTEM-DESIGN.md` · The two flows](SYSTEM-DESIGN.md#the-two-flows) |
| What each noun means exactly (Profile / Report / Role Model / Milestone …) | [`SYSTEM-DESIGN.md` · The language](SYSTEM-DESIGN.md#the-language) |
| Aggregates, tables and invariants | [`SYSTEM-DESIGN.md` · Domain model](SYSTEM-DESIGN.md#domain-model) |
| Service split, queue jobs and the LLM boundary | [`SYSTEM-DESIGN.md` · System design](SYSTEM-DESIGN.md#system-design) |
| The deliberately open questions | [`SYSTEM-DESIGN.md` · Open questions](SYSTEM-DESIGN.md#open-questions) |
| Backend hexagonal layers, configuration, LLM adapters | [`backend/README.md`](backend/README.md) |
| The schema as built, and the OpenAPI spec | [`backend/docs/db/schema.md`](backend/docs/db/schema.md) · [`backend/docs/api/`](backend/docs/api) |
| The three frontend stations, the rule engine, what maps to the backend | [`frontend/README.md`](frontend/README.md) · [`frontend/docs/API.md`](frontend/docs/API.md) |

Both implementations live in this repository: the backend in [`backend/`](backend), the
frontend in [`frontend/`](frontend).

## Technologies used

| Type | Technology / service | Purpose |
| --- | --- | --- |
| AI model | xAI Grok (`grok-4.6`, default) | Classify Profile signals, produce the Reports, score Role Model fit, produce the plan template, narrate the review |
| AI model | Anthropic Claude / local Ollama · vLLM (swappable) | As above; switched by the `LLM_ADAPTER` and `LLM_BASE_URL` environment variables, no code change |
| Frontend | React 19 · Next 16 (vinext) · Vite · TypeScript 5.9 | Web client — three pages for the three stations (`/`, `/plan`, `/ledger`) |
| Frontend | React Server Components · `snapshot-adapter` | All three stations read the backend in a server component, so the bearer token never reaches the browser; the adapter is the one place the two vocabularies meet |
| Frontend | mist design system (vendored CSS) · Vitest | Interface styling, plus the rule-engine, backend-mapping and server-render tests |
| Frontend | Cloudflare Workers · Wrangler | Frontend deployment, built to Worker-compatible ESM; in production the token is a Worker secret |
| Backend | Python 3.12 · FastAPI · Pydantic 2 · SQLAlchemy 2 (async) · Alembic | The three services' API and data access |
| Backend | ARQ · Redis 7 | Async job queues (`import.parse` / `profile.build` / `direction.run` / `plan.generate` / `reconcile.run` / `export.push`) |
| Backend | PostgreSQL 16 | Primary database, authoritative for all state |
| Backend | pypdf · python-docx · openpyxl · icalendar · BeautifulSoup · markdown-it-py | Upload parsing (7 parsers) |
| Backend | Cloudflare R2 / local filesystem (swappable) | Object storage for uploads, switched by `STORAGE_BACKEND` |
| Engineering | ruff · mypy --strict · import-linter · pytest | The hexagonal dependency direction is enforced in CI — a reverse import fails the build |
| Deployment | Docker Compose · DigitalOcean Droplet · Caddy | One image; the entrypoint decides which role it runs |
| External | Google OAuth 2.0 · Google Calendar API | Sign-in, and calendar import / export |

## Installation and running

Requires Python 3.12, [uv](https://docs.astral.sh/uv/), Node.js ≥ 22.13, PostgreSQL and Redis.
Backend and frontend live in this one repository; `backend/` and `frontend/` build independently.

```bash
git clone git@github.com:Quasar-Gang/life-guru.git && cd life-guru
```

```bash
# 1 · Backend, backend/
cd backend
uv sync
cp .env.example .env                  # set DATABASE_URL / REDIS_URL / LLM_API_KEY / GOOGLE_CLIENT_ID
uv run alembic upgrade head           # create the schema
uv run python -m cmd.seed_role_models # load the six Role Model templates
make check                            # ruff → mypy --strict → import-linter → pytest

# four processes
uv run python -m cmd.api_server      # HTTP, port 8000
uv run python -m cmd.api_worker      # import.parse · export.push
uv run python -m cmd.engine_worker   # profile.build · direction.run · plan.generate · reconcile.run
uv run python -m cmd.catalog_server  # HTTP, port 8001

# or the whole stack in containers (build + up + migrate + seed)
make deploy env=local                 # postgres / redis publish on 5433 / 6380 to avoid collisions
make deploy-smoke env=local           # the whole loop end to end
make deploy-down env=local            # tear the stack down
```

```bash
# 2 · Frontend, frontend/
cd frontend
npm install
cp .env.example .env.local            # GURU_API_BASE_URL=http://127.0.0.1:8000
                                      # GURU_API_TOKEN=<bearer token from POST /v1/auth/google>
npm run dev                           # the dev URL is printed in the terminal
npm test                              # language check → typecheck → lint → build → vitest
```

> **Both variables are server-side only**, which is why neither carries a `NEXT_PUBLIC_`
> prefix — that would ship the token to the browser. `vite.config.ts` passes them to the
> Worker as vars and every station reads its data in a server component; a real deployment
> uses a Worker secret instead.
>
> `.env.local` may stay empty: with no backend configured the frontend runs on its built-in
> demonstration dataset and all three stations stay fully interactive — that is the demo
> path, not a broken state. `GURU_API_BASE_URL` must be an absolute `http(s)` origin;
> anything else is ignored and the fixture is used.
> For a local token, set `ALLOW_FAKE_LOGIN=1` in `backend/.env` and sign in with
> `{"code": "fake:<email>"}` (**local only**).
>
> The backend's `LLM_ADAPTER` defaults to `fake`, so the whole loop runs without a key. To
> reach a real provider set it to `openai_compat` and fill in `LLM_API_KEY` (for the
> container path, put those in `backend/deployment/local/.env.local`).

## Demo

- Demo URL: <https://wu0h9625-boop.github.io/guru-intake-prototype/> (all three stations: intake and direction, goal-tree draft, quarterly reconciliation)
- Judging video: <https://www.youtube.com/watch?v=vP_6S0C3R7Q>

## Limitations and future work

**Current limitations**

- **No sign-in.** The token is configured server-side, so the app reads one account; adding OAuth is a workflow, not a field
- **No "done, but no effect" row against live data.** The backend models no capability baseline and no retest, and without two measurements there is nothing to compare — the cell is left empty rather than faked, because faking it would fake the one outcome this product exists to show
- **Every live branch reads as an anchor gap**, because the backend models no anchors. The criterion is working; the prescriptions beside it are still fixture
- **What else does not map stays fixture**: alternative paths, the coach's challenges, free-text Role Model input, Apple Health import, the horizon's quarter arithmetic, and the reconciliation narrative (`POST /v1/reconciliations` is ready, but no control on these pages asks for the decision it carries) — each one named in [`frontend/docs/API.md`](frontend/docs/API.md)
- **Only the sign-off writes back**: accepting the draft activates the plan (`PUT /v1/plans/{id}/status`); deletions and the weekly proofread are frontend state
- **One render costs eleven reads** against a backend that rate-limits at 60 requests a minute, so the assembled snapshot is held for 30 seconds; each section falls back on its own and logs it, so one unavailable read never blanks a page
- **The money dimension has no source**: card-statement import is out of scope for this round, so the dispatch money axis is declared by the user
- **The constraint criterion needs two to three quarters of history** before it has anything to compare, so it is shown as unavailable rather than hidden
- **Export is Google Calendar only**: the Google Sheets scope is requested but the export is not implemented, and neither is Notion
- **The Reviewer still fires on the review date stamped on the Hypothesis**; behaviour-drift detection is not implemented — that question and five other deliberately open ones are listed in [`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md)
- **The three published prototype pages run on frontend fixtures**, and are a separate artefact from the now backend-connected `frontend/`

**Future work**

- Add the capability baseline and retest, and an anchor model, to the backend — those two together are what let "done, but no effect" and the anchor prescriptions run on live data
- Add a sign-in flow, so the app is no longer limited to the one server-configured account
- Move the Reviewer from a fixed review date to behaviour-drift detection, so it appears only when it is actually needed
- Give the frontend a UI for user-authored Role Model templates — the backend's `POST /v1/role-models` already supports them, on the same terms and with the same cost rule
- Card-statement import, to cover the money dimension
- Calendar change detection and automatic rescheduling
- Vision and the five-year layer: intake produces a one-year hypothesis today, and the longer scales are not covered

## Third-party services, data and assets

| Item | Source / link | Licence / usage |
|---|---|---|
| xAI Grok API | <https://x.ai/api> | Commercial API; key supplied via the `LLM_API_KEY` environment variable, never committed |
| Anthropic Claude API | <https://www.anthropic.com/api> | As above, a swappable alternative provider |
| Google OAuth 2.0 · Calendar API | <https://developers.google.com/calendar> | Accessed with user consent; refresh tokens stored encrypted, the frontend never touches a Google token |
| Cloudflare Workers · R2 | <https://developers.cloudflare.com/> | Deployment and object storage |
| PostgreSQL | <https://www.postgresql.org/> | PostgreSQL License |
| Redis | <https://redis.io/> | RSALv2 / SSPLv1 |
| FastAPI · SQLAlchemy · Alembic · ARQ | Their respective repositories | MIT / BSD-family open source licences |
| React · Next.js · Vite · Tailwind CSS | Their respective repositories | MIT |
| Plus Jakarta Sans · Noto Sans TC (prototype fonts) | <https://fonts.google.com/> | SIL Open Font License 1.1 |
| Prototype demo data | Written by the team; a fictional user | Not real personal data |

> No keys, tokens or personal data are committed to the repositories; every credential comes
> from an environment variable, and `.env.example` lists field names only.

## Team

| Name | Email | Role |
| --- | --- | --- |
| Yoshi (橋本高佳) | <yoshi4868686@gmail.com> | Backend / Architecture |
| Steven | <gummy789j@gmail.com> | Infra / Architecture |
| Lynn | <wu0h9625@gmail.com> | UI / UX |
| Freddie (周森翔) | <remix0622@gmail.com> | Business Insight |
| 小測 (陳鈺培) | <hosailei711@gmail.com> | Consultant |

## License

Proprietary. Copyright (c) 2026 Quasar-Gang, all rights reserved. No licence to use, copy,
modify or distribute is granted without written permission.

See [`LICENSE`](./LICENSE) at the repository root for the full terms.
