<div align="center">

# guru

**English**（this page） · [繁體中文](README.md)

**Find the shape your own data already supports, and one experiment to test it.**

![Status](https://img.shields.io/badge/status-design-D97706)
[![Prototype](https://img.shields.io/badge/prototype-live-0F9D58)](https://wu0h9625-boop.github.io/guru-intake-prototype/)
![License](https://img.shields.io/badge/license-proprietary-A31515)

</div>

---

## Problem and goal

**Nobody can answer *"what is your vision?"* — it asks you to invent something from nothing.**

Every goal product starts by asking what you want. For the minority who already know, that
works. For everyone else it is a wall on screen one, and no amount of clever prompting gets
past it, because the question presumes an answer that does not exist yet.

**The target user** is someone who knows something is off but cannot say where they want to
go — they have a job, a calendar and a résumé, but no sense of direction and no standing to
write a five-year plan.

So the system never asks what you want. It **reads the traces you have already left** — where
your hours went, what your résumé keeps repeating — offers six life shapes each with its cost
stated, says which one your behaviour actually supports, and hands back **a hypothesis you can
disprove inside one quarter** rather than a vision.

**Intended impact:** turn "I don't know what I want" into an answer you can walk forward
from. The cost of choosing wrong drops from five years to one season, and every season it is
reconciled against real behaviour.

## Core features

The system is two flows.

**1 · Explore role model**

- **Upload what already exists** — Google Calendar and a résumé PDF; no recall, no new tracking
- **Build a Profile** — one per user, the system's read of who you are now
- **Produce multi-dimension Reports** — work / exercise / social / learning / capacity; unclassified time is a first-class result
- **Recommend 6 Role Model templates** — each states its own cost, and the user chooses
- **Generate a Milestone tree and Tasks** — Milestones nest; Tasks stay single-level beneath one
- **Schedule** — place the Tasks on real dates

**2 · Review task progress**

- **The Reviewer runs on a period** — weekly / monthly / quarterly, it reads the user's task progress
- **Under the threshold, it re-analyzes** — triggered by the numbers, not by the user asking for help
- **Re-recommend Role Models** — Reports and Recommender re-run on the newest behaviour
- **Replan after a role change** — the user picks a new Role Model and the Plan Engine rebuilds Milestones, Tasks and the schedule

> **The design decision that matters:** the Recommender never sees the raw Profile — it reads
> the multi-dimension Reports. Borrowed from chain-of-thought: producing intermediate,
> inspectable evidence before reasoning improves precision, and makes every recommendation
> something the user can **argue with**.

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
| **Backend · recommendation** | Role Model Service | Role Model template queries and LLM-backed recommendation |
| **AI model** | LLM (xAI Grok) | Judgement only: analyze Reports, recommend Role Models, produce the plan template |
| **Database** | PostgreSQL · Redis | All state in the former; queues and cache in the latter |
| **External services** | Google Calendar · Cloudflare R2 | Calendar import and export, uploaded-file storage |

**The LLM does judgement, never arithmetic.** Scheduling, quota maths and progress
comparison are deterministic code — the same inputs must give the same result, or "was the
hypothesis disproved?" is a comparison against noise.

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
- Both code repositories currently implement an earlier goal-first model and have not yet converged on the design described here
- The Reviewer's trigger threshold is undecided — see the open questions in [`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md)
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

| Name | Role |
| --- | --- |
| _TODO_ | _TODO_ |

## License

Proprietary. Copyright (c) 2026 Quasar-Gang, all rights reserved. No licence to use, copy,
modify or distribute is granted without written permission.

> _TODO: add a `LICENSE` file at the repository root._
