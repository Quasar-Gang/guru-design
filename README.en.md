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
| **Frontend** | `guru-app` | Upload UI, Report views, Role Model selection, daily tasks and progress check-in |
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

The full domain model, invariants, queue jobs and LLM boundary are in
[`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md).

## Technologies used

| Type | Technology / service | Purpose |
| --- | --- | --- |
| AI model | xAI Grok (`grok-4.6`, default) | Analyze the Profile into Reports, recommend Role Models, produce the plan template |
| AI model | Anthropic Claude / local Ollama · vLLM (swappable) | As above; switched by the `LLM_ADAPTER` environment variable, no code change |
| Frontend | React 19 · Next 16 (vinext) · Vite · Tailwind v4 | Web client |
| Frontend | Cloudflare Workers | Frontend deployment |
| Backend | Python 3.12 · FastAPI · SQLAlchemy 2 (async) · Alembic | API service and data access |
| Backend | ARQ · Redis 7 | Async job queues (parse, generate, revise, export) |
| Backend | PostgreSQL 16 | Primary database |
| Backend | Cloudflare R2 | Object storage for uploads |
| External | Google OAuth 2.0 · Google Calendar API | Sign-in, and calendar import / export |
| Sponsor tech | _TODO: fill in the sponsor technologies actually used, and what for_ | |

## Installation and running

Requires Python 3.12, [uv](https://docs.astral.sh/uv/), Node.js ≥ 22.13, PostgreSQL and Redis.

```bash
# 1 · Backend, guru-core
git clone git@github.com:Quasar-Gang/guru-core.git && cd guru-core
uv sync
cp .env.example .env                  # set DATABASE_URL / REDIS_URL / LLM_API_KEY / GOOGLE_CLIENT_ID
uv run alembic upgrade head           # create the schema
uv run python -m cmd.seed_role_models # load the Role Model templates
make check                            # ruff → mypy --strict → import-linter → pytest

# four processes
uv run python -m cmd.api_server          # HTTP, port 8000
uv run python -m cmd.api_worker          # import.parse · export.push
uv run python -m cmd.plan_engine_worker  # plan.generate · continue · revise
uv run python -m cmd.role_model_server   # HTTP, port 8001

# or the whole stack in containers (build + up + migrate + seed)
make deploy env=local
```

```bash
# 2 · Frontend, guru-app
git clone git@github.com:Quasar-Gang/guru-app.git && cd guru-app
npm install
cp .env.example .env.local            # NEXT_PUBLIC_API_BASE_URL=http://localhost:8000
npm run dev                           # http://localhost:3100
```

> With no backend configured the frontend falls back to built-in demo data and stays interactive.

## Demo

- Demo URL: <https://wu0h9625-boop.github.io/guru-intake-prototype/> (the intake-and-direction prototype for Flow 1)
- Judging video: _TODO: add the video link_

## Limitations and future work

**Current limitations**

- Flow 1's **intake and direction** is verified end to end by the prototype; **Flow 2 (review task progress) is designed, not yet prototype-verified**
- The prototype runs on demo data — real Google Calendar and résumé parsing are not wired in
- Both code repositories (`guru-core` / `guru-app`) currently implement an earlier goal-first model and have not yet converged on the design described here
- The Reviewer's trigger threshold — and how it tells "task too big" from "not enough time" from "the goal needs adjusting" — is undecided; see the open questions in [`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md)
- Export supports Google Calendar and Markdown only; Notion and Google Sheets are not implemented

**Future work**

- Move the Reviewer from a fixed period to behaviour-drift detection, so it appears only when it is actually needed
- Let users write their own Role Model templates rather than only picking from the six
- Card-statement import, to cover the money dimension
- Calendar change detection and automatic rescheduling

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
