<div align="center">

# guru

**繁體中文**（本頁） · [English](README.en.md)

**找到你的資料本來就支持的那個形狀，以及一個用來驗證它的實驗。**

![狀態](https://img.shields.io/badge/status-design-D97706)
[![原型](https://img.shields.io/badge/prototype-live-0F9D58)](https://wu0h9625-boop.github.io/guru-intake-prototype/)
![授權](https://img.shields.io/badge/license-proprietary-A31515)

</div>

---

## 問題與目標

**沒有人回答得出「你的願景是什麼」——那是一道要你無中生有的題。**

市面上每一個目標管理產品都從「你想要什麼」開始問。對已經知道答案的少數人來說沒問題；
對其他人來說，那是第一個畫面就撞上的牆，而且再怎麼會問也繞不過去，
因為這個問題預設了一個還不存在的答案。

**目標使用者**是那些「知道現在不太對，但說不出想去哪裡」的人——
有工作、有行事曆、有履歷，但沒有方向感，也沒有立場去寫五年計畫。

所以這個系統從不問你要什麼。它**讀你已經留下的痕跡**（時間怎麼花、履歷重複寫著什麼），
給你六個標好代價的人生形狀，說出你的行為實際上支持哪一個，
再交還給你一個**能在一季之內被推翻的假設**，而不是一份願景。

**預期影響**：把「我不知道我要什麼」變成一個可以往下走的答案。
選錯的代價從五年縮短成一季，而且每一季都會用真實行為重新對帳一次。

## 核心功能

系統由兩條主要流程構成。

**1 · 探索 Role Model（`explore role model`）**

- **上傳既有資料**——Google 行事曆與履歷 PDF；不要求使用者回想或開始記錄
- **建立 Profile**——每位使用者唯一一份，代表系統對「你現在是誰」的讀法
- **產生多面向 Report**——工作／運動／社交／學習／容量等面向，未分類時間也是第一級結果
- **推薦 6 個 Role Model 樣板**——每一個都標出自己的代價，讓使用者自己選
- **產生 Milestone 樹與 Task**——Milestone 可巢狀，Task 在 Milestone 底下維持單層
- **排程**——把 Task 放到真實日期上

**2 · 檢視任務進度（`review task progress`）**

- **Reviewer 定期檢視**——每週／每月／每季讀取使用者的任務進度
- **低於門檻就重新分析**——不是等使用者主動求助，而是由進度數字觸發
- **重新推薦 Role Model**——用最新的行為資料重跑 Report 與 Recommender
- **換角色後重新規劃**——使用者選了新的 Role Model，Plan Engine 重建 Milestone、Task 並重新排程

> **設計上的關鍵決定：**Recommender 不直接看 Profile，而是看多面向的 Report。
> 借用 CoT（chain of thought）的想法：先產生中間、可檢視的證據再推理，
> 不但推論精準度更好，也讓每一個建議都**可以被使用者反駁**。

## 系統架構

### 流程一 · 探索 Role Model

<img src="assets/explore_role_model.png" alt="探索 Role Model 流程：User 上傳個人資料給 Uploader，建立唯一一份 Profile；Analyzer 讀取後產出各面向 Report；Recommender 讀 Report 並推薦六個 Role Model 樣板；User 選定後由 Plan Engine 建立 Milestone、產生 Task 並排程">

### 流程二 · 檢視任務進度

<img src="assets/review_task_progress.png" alt="檢視任務進度流程：Reviewer 定期讀取 Task 進度，低於門檻時觸發 Analyzer 重新分析，重新產生 Report 與 Role Model 推薦；User 選擇新角色後，Plan Engine 建立新的 Milestone、Task 並重新排程">

### 各層如何協作

| 層 | 元件 | 職責 |
|---|---|---|
| **前端** | `guru-app` | 上傳介面、Report 檢視、Role Model 選擇、每日任務與進度回報 |
| **後端 · API** | API Service | 認證、OAuth、檔案上傳與解析、所有對前端的 endpoint、工作派送 |
| **後端 · 規劃** | Plan Engine | Milestone 樹、Task 產生、**決定性**排程與重新排程 |
| **後端 · 推薦** | Role Model Service | Role Model 樣板查詢與 LLM 推薦 |
| **AI 模型** | LLM（xAI Grok） | 只做判斷：分析 Report、產生 Role Model 建議、產生計畫模板 |
| **資料庫** | PostgreSQL · Redis | 前者存所有狀態，後者做佇列與快取 |
| **外部服務** | Google Calendar · Cloudflare R2 | 行事曆匯入與匯出、上傳檔案儲存 |

**LLM 只負責判斷，不負責算數。** 排程、額度計算與進度比對都是決定性的程式碼——
同樣的輸入必須得到同樣的結果，否則「假設是否被推翻」就是在跟雜訊比較。

完整的領域模型、不變量、佇列工作與 LLM 邊界，見 [`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md)。

## 使用技術

| 類型 | 技術／服務 | 用途 |
| --- | --- | --- |
| AI 模型 | xAI Grok（`grok-4.6`，預設） | 分析 Profile 產生 Report、推薦 Role Model、產生計畫模板 |
| AI 模型 | Anthropic Claude／本地 Ollama · vLLM（可切換） | 同上；透過 `LLM_ADAPTER` 環境變數切換，不需改動程式碼 |
| 前端 | React 19 · Next 16（vinext）· Vite · Tailwind v4 | 網頁客戶端 |
| 前端 | Cloudflare Workers | 前端部署 |
| 後端 | Python 3.12 · FastAPI · SQLAlchemy 2（async）· Alembic | API 服務與資料存取 |
| 後端 | ARQ · Redis 7 | 非同步工作佇列（解析、產生計畫、重新規劃、匯出） |
| 後端 | PostgreSQL 16 | 主要資料庫 |
| 後端 | Cloudflare R2 | 上傳檔案物件儲存 |
| 外部服務 | Google OAuth 2.0 · Google Calendar API | 登入，以及行事曆匯入／匯出 |
| Sponsor 技術 | _TODO：請填入實際使用的 Sponsor 技術與用途_ | |

## 安裝與執行

需要 Python 3.12、[uv](https://docs.astral.sh/uv/)、Node.js ≥ 22.13、PostgreSQL 與 Redis。

```bash
# 1 · 後端 guru-core
git clone git@github.com:Quasar-Gang/guru-core.git && cd guru-core
uv sync
cp .env.example .env                  # 設定 DATABASE_URL / REDIS_URL / LLM_API_KEY / GOOGLE_CLIENT_ID
uv run alembic upgrade head           # 建立資料庫 schema
uv run python -m cmd.seed_role_models # 載入 Role Model 樣板
make check                            # ruff → mypy --strict → import-linter → pytest

# 啟動四個 process
uv run python -m cmd.api_server          # HTTP, port 8000
uv run python -m cmd.api_worker          # import.parse · export.push
uv run python -m cmd.plan_engine_worker  # plan.generate · continue · revise
uv run python -m cmd.role_model_server   # HTTP, port 8001

# 或用容器一次啟動（build + up + migrate + seed）
make deploy env=local
```

```bash
# 2 · 前端 guru-app
git clone git@github.com:Quasar-Gang/guru-app.git && cd guru-app
npm install
cp .env.example .env.local            # NEXT_PUBLIC_API_BASE_URL=http://localhost:8000
npm run dev                           # http://localhost:3100
```

> 未設定後端時，前端會退回內建的示範資料，互動仍可操作。

## 作品展示

- 作品展示網址：<https://wu0h9625-boop.github.io/guru-intake-prototype/>（流程一的上傳與方向原型）
- 評選影片：_TODO：請填入影片連結_

## 限制與未來工作

**目前限制**

- 流程一的**上傳與方向**已由原型完整驗證；**流程二（檢視任務進度）目前是設計，尚未經原型驗證**
- 原型使用示範資料，尚未串接真實的 Google 行事曆與履歷解析
- 兩個程式碼儲存庫目前實作的是較早的「先問目標」模型，尚未收斂到本文件描述的設計
- Reviewer 的觸發門檻尚未定案（見 [`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md) 的待決問題）
- 匯出目前只支援 Google Calendar 與 Markdown；Notion、Google Sheets 尚未實作

**後續方向**

- 把 Reviewer 從固定週期改為行為偏移偵測，讓它在真正需要時才出現
- 讓使用者自己撰寫 Role Model 樣板，而不只從六個內建的挑
- 信用卡帳單匯入，補上金錢流向這一個面向
- 行事曆變更偵測與自動重新排程

## 第三方服務、資料與素材

| 項目 | 來源／連結 | 授權／使用方式 |
|---|---|---|
| xAI Grok API | <https://x.ai/api> | 商用 API，金鑰以環境變數 `LLM_API_KEY` 提供，未進版控 |
| Anthropic Claude API | <https://www.anthropic.com/api> | 同上，可切換的替代供應商 |
| Google OAuth 2.0 · Calendar API | <https://developers.google.com/calendar> | 使用者授權後存取；refresh token 加密存放，前端不接觸 Google token |
| Cloudflare Workers · R2 | <https://developers.cloudflare.com/> | 部署與物件儲存 |
| PostgreSQL | <https://www.postgresql.org/> | PostgreSQL License |
| Redis | <https://redis.io/> | RSALv2 / SSPLv1 |
| FastAPI · SQLAlchemy · Alembic · ARQ | 各自官方儲存庫 | MIT／BSD 系列開源授權 |
| React · Next.js · Vite · Tailwind CSS | 各自官方儲存庫 | MIT |
| Plus Jakarta Sans · Noto Sans TC（原型字體） | <https://fonts.google.com/> | SIL Open Font License 1.1 |
| 原型示範資料 | 團隊自行撰寫的虛構使用者 | 非真實個人資料 |

> 儲存庫內未提交任何金鑰、Token 或個人資料；所有憑證皆由環境變數提供，
> `.env.example` 只列出欄位名稱。

## 團隊成員

| 姓名 | 分工 |
| --- | --- |
| _TODO_ | _TODO_ |

## License

Proprietary. Copyright (c) 2026 Quasar-Gang, all rights reserved.
版權所有，未經書面同意不得使用、重製、修改或散布。

> _TODO：請在儲存庫根目錄加入 `LICENSE` 檔案。_
