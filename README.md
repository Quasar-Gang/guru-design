<div align="center">

# Life Guru

**繁體中文**（本頁） · [English](README.en.md)

**讓人生目標有依據，讓每天行動有方向。**

一款個人成長 AI 教練，幫助有成長意願、卻難以持續行動的人，
找到值得投入的方向，把目標轉換成符合自身能力與時間的行動計畫，並透過持續回饋調整步調。

![狀態](https://img.shields.io/badge/status-design-D97706)
[![原型](https://img.shields.io/badge/prototype-live-0F9D58)](https://wu0h9625-boop.github.io/guru-intake-prototype/)
![授權](https://img.shields.io/badge/license-proprietary-A31515)

[後端 `backend/`](backend) · [前端 `frontend/`](frontend) · [原型](https://wu0h9625-boop.github.io/guru-intake-prototype/) · [影片](https://www.youtube.com/watch?v=vP_6S0C3R7Q)

</div>

---

## 問題與目標

Life Guru 要降低兩種成本：

| | 成本 | 長什麼樣子 |
|---|---|---|
| **1** | **不知道方向的成本** | 反覆規劃，到頭來沒開始，原地踏步 |
| **2** | **計畫中斷的成本** | 開始之後，因為計畫不符合生活型態而停下來 |

**目標使用者**是有成長意願、卻難以持續行動的人——有工作、有行事曆、有履歷，
知道現在不太對，但說不出想去哪裡，也沒有立場去寫一份五年計畫。

問題的根源在於：**「你的願景是什麼」是一道要人無中生有的題。**
市面上每一個目標管理產品都從這裡開始問；對已經知道答案的少數人來說沒問題，
對其他人來說，那是第一個畫面就撞上的牆。

所以 Life Guru 不從提問開始，而是**從使用者已經留下的資料開始**，
讓使用者先看見自己的現況，再用簡短問答補足資料無法解釋的部分。
這些資訊共同構成後續規劃的依據，**減少使用者從零整理自己的負擔**。

> **產品的核心原則：**每一項建議都應該讓使用者知道
> **為什麼值得嘗試、需要付出什麼，以及今天可以如何開始。**

**預期影響**：把「我不知道我要什麼」變成一個可以往下走的答案，
並且讓計畫能隨生活變動而延續，而不是中斷。

## 核心功能

系統由兩條主要流程構成，對應上面那兩種成本。

**1 · 探索 Role Model（`explore role model`）——解決「不知道方向」**

- **從既有資料開始**——匯入 Google 行事曆與履歷 PDF，不要求使用者回想或開始記錄
- **先看見現況**——建立每人唯一一份 Profile，並產生多面向 Report
  （工作／運動／社交／學習／容量），未分類的時間也是第一級結果
- **簡短問答補足缺口**——補上資料無法解釋的部分：不願接受的條件、
  願意分配到這件事的時間；每一題都可以跳過
- **用 Role Model 具體比較不同的生活與工作方式**——每個方向都呈現
  **需要累積的能力、投入與取捨**，並對照個人資料說明
  **目前有哪些基礎、還有哪些缺口**
- **使用者保有主導權**——建議的理由是攤開的，使用者可以補充或修正 AI 的判斷

**2 · 檢視任務進度（`review task progress`）——解決「計畫中斷」**

- **把方向拆成里程碑與具體任務**——Milestone 可巢狀，Task 在 Milestone 底下維持單層
- **依可用時間安排每日行動**——排到真實日期上，並尊重使用者願意投入的程度
- **完成紀錄與定期回顧**——Reviewer 每週／每月／每季讀取任務進度
- **診斷卡關的來源**——判斷卡關是因為**任務太大**、**時間不足**，
  還是**目標本身需要調整**
- **重新規劃**——需要調整方向時重跑分析與推薦；使用者選了新的 Role Model 之後，
  Plan Engine 重建 Milestone、Task 並重新排程

> **設計上的關鍵決定：**Recommender 不直接看 Profile，而是看多面向的 Report。
> 借用 CoT（chain of thought）的想法：先產生中間、可檢視的證據再推理，
> 不但推論精準度更好，也讓每一個建議都**可以被使用者反駁**——
> 這正是「使用者保有主導權」在技術上的實作方式。

## 系統架構

### 流程一 · 探索 Role Model

<img src="assets/explore_role_model.png" alt="探索 Role Model 流程：User 上傳個人資料給 Uploader，建立唯一一份 Profile；Analyzer 讀取後產出各面向 Report；Recommender 讀 Report 並推薦六個 Role Model 樣板；User 選定後由 Plan Engine 建立 Milestone、產生 Task 並排程">

### 流程二 · 檢視任務進度

<img src="assets/review_task_progress.png" alt="檢視任務進度流程：Reviewer 定期讀取 Task 進度，低於門檻時觸發 Analyzer 重新分析，重新產生 Report 與 Role Model 推薦；User 選擇新角色後，Plan Engine 建立新的 Milestone、Task 並重新排程">

### 各層如何協作

| 層 | 元件 | 職責 |
|---|---|---|
| **前端** | Web App（[`frontend/`](frontend)） | 上傳介面、Report 檢視、Role Model 選擇、每日任務與進度回報 |
| **後端 · API** | API Service | 認證、OAuth、檔案上傳與解析、所有對前端的 endpoint、工作派送 |
| **後端 · 規劃** | Plan Engine | Milestone 樹、Task 產生、**決定性**排程與重新排程 |
| **後端 · 回顧** | Reviewer | 定期讀取任務進度，診斷卡關來源並在需要時觸發重新分析 |
| **後端 · 推薦** | Role Model Service | Role Model 樣板查詢與 LLM 推薦 |
| **AI 模型** | LLM（xAI Grok） | 只做判斷：分析 Report、產生 Role Model 建議與理由、產生計畫模板 |
| **資料庫** | PostgreSQL · Redis | 前者存所有狀態，後者做佇列與快取 |
| **外部服務** | Google Calendar · Cloudflare R2 | 行事曆匯入與匯出、上傳檔案儲存 |

**LLM 只負責判斷，不負責算數。** 排程、時間計算與進度比對都是決定性的程式碼——
同樣的輸入必須得到同樣的結果，否則「這次卡關是因為任務太大還是時間不足」
就是在跟雜訊比較。

### 深入技術細節

README 講到「為什麼這樣做」與「長什麼樣子」為止；架構底下的完整規格寫在
**[`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md)**——領域模型、不變量、佇列工作與 LLM 的邊界都在那裡。

| 想知道什麼 | 讀哪裡 |
|---|---|
| 兩條流程逐步展開 | [`SYSTEM-DESIGN.md` · The two flows](SYSTEM-DESIGN.md#the-two-flows) |
| 名詞的精確定義（Profile／Report／Role Model／Milestone…） | [`SYSTEM-DESIGN.md` · The language](SYSTEM-DESIGN.md#the-language) |
| 聚合、資料表與不變量 | [`SYSTEM-DESIGN.md` · Domain model](SYSTEM-DESIGN.md#domain-model) |
| 服務切分、佇列工作與 LLM 邊界 | [`SYSTEM-DESIGN.md` · System design](SYSTEM-DESIGN.md#system-design) |
| 刻意未決的問題 | [`SYSTEM-DESIGN.md` · Open questions](SYSTEM-DESIGN.md#open-questions) |
| 後端六邊形架構、設定檔與 LLM adapter | [`backend/README.md`](backend/README.md) |
| 實際 schema 與 OpenAPI 規格 | [`backend/docs/db/schema.md`](backend/docs/db/schema.md) · [`backend/docs/api/`](backend/docs/api) |
| 前端三個站點、規則引擎與後端對映 | [`frontend/README.md`](frontend/README.md) · [`frontend/docs/API.md`](frontend/docs/API.md) |

實作都在這個儲存庫裡：後端在 [`backend/`](backend)，前端在 [`frontend/`](frontend)。

## 使用技術

| 類型 | 技術／服務 | 用途 |
| --- | --- | --- |
| AI 模型 | xAI Grok（`grok-4.6`，預設） | 分類 Profile 訊號、產生 Report、評分 Role Model 契合度、產生計畫模板、敘述回顧結果 |
| AI 模型 | Anthropic Claude／本地 Ollama · vLLM（可切換） | 同上；以 `LLM_ADAPTER`、`LLM_BASE_URL` 環境變數切換，不需改動程式碼 |
| 前端 | React 19 · Next 16（vinext）· Vite · TypeScript 5.9 | 網頁客戶端，三個頁面對應三個站點（`/`、`/plan`、`/ledger`） |
| 前端 | React Server Components · `snapshot-adapter` | 三個站點都在 server component 讀後端 API，bearer token 只留在伺服器端；adapter 是兩套詞彙唯一交會的地方 |
| 前端 | mist 設計系統（vendored CSS）· Vitest | 介面樣式，以及規則引擎、後端對映與伺服器渲染測試 |
| 前端 | Cloudflare Workers · Wrangler | 前端部署，建置為 Worker 相容的 ESM；正式環境的 token 是 Worker secret |
| 後端 | Python 3.12 · FastAPI · Pydantic 2 · SQLAlchemy 2（async）· Alembic | 三個服務的 API 與資料存取 |
| 後端 | ARQ · Redis 7 | 非同步工作佇列（`import.parse`／`profile.build`／`direction.run`／`plan.generate`／`reconcile.run`／`export.push`） |
| 後端 | PostgreSQL 16 | 主要資料庫，所有狀態的唯一真相 |
| 後端 | pypdf · python-docx · openpyxl · icalendar · BeautifulSoup · markdown-it-py | 上傳檔案解析（7 種 parser） |
| 後端 | Cloudflare R2／本地檔案系統（可切換） | 上傳檔案物件儲存，以 `STORAGE_BACKEND` 切換 |
| 工程品質 | ruff · mypy --strict · import-linter · pytest | 六邊形架構的依賴方向由 import-linter 在 CI 擋下，反向 import 直接建置失敗 |
| 部署 | Docker Compose · DigitalOcean Droplet · Caddy | 單一映像檔，由 entrypoint 決定跑哪一個角色 |
| 外部服務 | Google OAuth 2.0 · Google Calendar API | 登入，以及行事曆匯入／匯出 |

## 安裝與執行

需要 Python 3.12、[uv](https://docs.astral.sh/uv/)、Node.js ≥ 22.13、PostgreSQL 與 Redis。
後端與前端都在這個儲存庫裡，`backend/` 與 `frontend/` 各自獨立建置。

```bash
git clone git@github.com:Quasar-Gang/life-guru.git && cd life-guru
```

```bash
# 1 · 後端 backend/
cd backend
uv sync
cp .env.example .env                  # 設定 DATABASE_URL / REDIS_URL / LLM_API_KEY / GOOGLE_CLIENT_ID
uv run alembic upgrade head           # 建立資料庫 schema
uv run python -m cmd.seed_role_models # 載入六個 Role Model 樣板
make check                            # ruff → mypy --strict → import-linter → pytest

# 啟動四個 process
uv run python -m cmd.api_server      # HTTP, port 8000
uv run python -m cmd.api_worker      # import.parse · export.push
uv run python -m cmd.engine_worker   # profile.build · direction.run · plan.generate · reconcile.run
uv run python -m cmd.catalog_server  # HTTP, port 8001

# 或用容器一次啟動（build + up + migrate + seed）
make deploy env=local                 # postgres／redis 發佈在 5433 / 6380，避免與本機衝突
make deploy-smoke env=local           # 端到端跑完整條流程
make deploy-down env=local            # 收掉整個 stack
```

```bash
# 2 · 前端 frontend/
cd frontend
npm install
cp .env.example .env.local            # GURU_API_BASE_URL=http://127.0.0.1:8000
                                      # GURU_API_TOKEN=<POST /v1/auth/google 取得的 bearer token>
npm run dev                           # 開發網址見終端機輸出
npm test                              # 語言檢查 → typecheck → lint → build → vitest
```

> **兩個變數都只在伺服器端使用**，因此都沒有 `NEXT_PUBLIC_` 前綴——加了就會把 token 送進瀏覽器。
> `vite.config.ts` 把它們當 var 傳給 Worker，每個站點都在 server component 讀資料；正式部署則用 Worker secret。
>
> `.env.local` 可以留空：沒有設定後端時，前端會使用內建的示範資料集，三個站點的互動仍然完整可操作——那是展示路徑，不是壞掉的狀態。
> `GURU_API_BASE_URL` 必須是絕對的 `http(s)` 位址，其他值一律忽略並改用示範資料。
> 本機要拿 token，可在 `backend/.env` 設 `ALLOW_FAKE_LOGIN=1` 後以 `{"code": "fake:<email>"}` 登入（**僅限本機**）。
>
> 後端的 `LLM_ADAPTER` 預設是 `fake`，不需要金鑰就能跑完整條流程；要接真的供應商，
> 改成 `openai_compat` 並填入 `LLM_API_KEY`（容器模式則寫在 `backend/deployment/local/.env.local`）。

## 作品展示

- 作品展示網址：<https://wu0h9625-boop.github.io/guru-intake-prototype/>（三個站點的互動原型：上傳與方向、目標樹草案、季度對帳）
- 評選影片：<https://www.youtube.com/watch?v=vP_6S0C3R7Q>

## 限制與未來工作

**目前限制**

- **沒有登入畫面**：token 由伺服器端設定，整個 app 只讀一個帳號；補上 OAuth 是一段流程，不是一個欄位
- **接真實資料時不會出現「做了卻沒效果」**：後端沒有能力重測的基準與重測，少了兩次測量就無從比較——這一格刻意留白，造一個假的等於造假這個產品唯一存在的理由
- **每一條 live 分支都判為缺少錨點**：後端沒有錨點模型。準則本身是對的，旁邊的處方仍是示範資料
- **其他未對映、維持示範資料的部分**：替代路徑、教練追問、Role Model 自由輸入、Apple Health 匯入、Horizon 的季度換算，以及對帳敘述（`POST /v1/reconciliations` 已可用，但頁面上沒有一個控制項在要求那個決定）——逐項列在 [`frontend/docs/API.md`](frontend/docs/API.md)
- **只有簽核會寫回後端**：接受草案會把計畫轉為 active（`PUT /v1/plans/{id}/status`），刪除與週檢查仍只是前端狀態
- **一次渲染要讀 11 次**，後端限流 60 rpm，因此組好的 snapshot 快取 30 秒；每一段各自 fallback 並記錄在伺服器日誌，避免一段掛掉就白掉整頁
- **金錢面向沒有資料來源**：信用卡帳單匯入不在這一輪範圍，調度的金錢軸目前由使用者自行宣告
- **約束準則需要兩到三季的歷史**才有東西可比，因此顯示為「尚無法判斷」，而不是藏起來
- **匯出只有 Google Calendar**：Google Sheets 的授權範圍已申請但尚未實作，Notion 也未實作
- **Reviewer 仍以 Hypothesis 上標記的回顧日期觸發**，行為偏移偵測尚未實作；這一項與另外五項刻意未決的問題列在 [`SYSTEM-DESIGN.md`](SYSTEM-DESIGN.md) 的待決問題
- **展示用的三頁原型跑的是前端假資料**，與已接上後端的 `frontend/` 是兩份東西

**後續方向**

- 在後端補上能力重測（基準＋重測）與錨點模型——這兩項一補，「做了卻沒效果」與錨點處方就能對到真實資料
- 補上登入流程，讓 app 不再只讀伺服器端設定的那一個帳號
- 把 Reviewer 從固定的回顧日期改為行為偏移偵測，讓它在真正需要時才出現
- 前端補上自寫 Role Model 樣板的介面——後端的 `POST /v1/role-models` 已經支援，且同樣要求寫出代價
- 信用卡帳單匯入，補上金錢流向這一個面向
- 行事曆變更偵測與自動重新排程
- 願景與五年層級：目前的上傳與方向只產出一年期的方向假設，更長的時間尺度尚未涵蓋

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

| 姓名 | Email | 分工 |
| --- | --- | --- |
| 橋本高佳（Yoshi） | <yoshi4868686@gmail.com> | 後端 / 架構 |
| Steven | <gummy789j@gmail.com> | 基礎架構 / 架構 |
| Lynn | <wu0h9625@gmail.com> | UI / UX |
| 周森翔（Freddie） | <remix0622@gmail.com> | 商業洞察 |
| 陳鈺培（小測） | <hosailei711@gmail.com> | 顧問 |

## License

Proprietary. Copyright (c) 2026 Quasar-Gang, all rights reserved.
版權所有，未經書面同意不得使用、重製、修改或散布。

完整條款請見儲存庫根目錄的 [`LICENSE`](./LICENSE)。
