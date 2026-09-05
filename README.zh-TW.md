<div align="center">

# guru

[English](README.md) · **繁體中文**（本頁）

**找到你的資料本來就支持的那個形狀，以及一個用來驗證它的實驗。**

沒有人回答得出「你的願景是什麼」——那是一道要你無中生有的題。所以這個系統從不問。
它讀你已經有的資料，給你六個借來的人生形狀，說出你的行為實際上支持哪一個，
再交還給你一個能在一季內被推翻的方向。

![狀態](https://img.shields.io/badge/status-design-D97706)
[![站 1](https://img.shields.io/badge/station%201-prototype--verified-0F9D58)](https://wu0h9625-boop.github.io/guru-intake-prototype/)
![站 2-3](https://img.shields.io/badge/stations%202--3-designed-4169E1)
![授權](https://img.shields.io/badge/license-proprietary-A31515)

[為什麼這樣設計](#為什麼這樣設計) · [六個形狀](#六個形狀) ·
[規格書](SYSTEM-DESIGN.md) · [試用原型](https://wu0h9625-boop.github.io/guru-intake-prototype/)

</div>

---

<img src="assets/guru-design.png" alt="整個系統：User 上傳個人資料給 Uploader，建立唯一一份 Profile；Analyzer 讀取後產出各面向的 Report；Recommender 讀這些 Report 並推薦六個 Role Model 樣板；User 選定其中一個後，Plan Engine 建立 Milestone、由此產生 Task，並排程這些 Task">

<div align="center"><sub><b>整個系統。</b>上傳 → Profile → Reports → 六個 Role Model → Milestone、Task、Schedule。<br/>每一步的規格都在 <a href="SYSTEM-DESIGN.md">SYSTEM-DESIGN.md</a>（英文）。</sub></div>

---

## 為什麼這樣設計

四個想法。設計裡其餘的一切都是從這裡長出來的。

**1 · 不問，改讀。** 你已經留下好幾年的痕跡——時間花在哪、履歷重複寫著什麼、
什麼事你悄悄停掉了。它不會說出你**想要**什麼，但會說出你現在長什麼形狀，
這就足夠借一個來測試。一個人聲稱重視什麼，和他的時間與金錢實際流向哪裡，
這中間的落差**就是**診斷。

**2 · 每一個形狀都標出自己的代價。** 沒有代價的樣板只是一場人氣投票，
而贏的永遠是講起來最好聽的那一個。

**3 · 一個便宜的測試，不是五年計畫。** 沒方向的人需要的是一個實驗——
一季做完、結果明確、失敗不痛。失敗代價承受得起的測試，才會真的被執行。

**4 · 產出的是假設，不是願景。** 有日期、有來源，而且**永不覆寫**——
因為可以悄悄改掉的假設，最後一定會被改寫成你剛好做到的樣子，然後什麼也沒學到。
一季之後系統會主動拿出來問：不是催你執行，是問這個形狀還算不算數。

它不做任何選擇。它只負責讀、攤開證據、標出代價，然後把決定交還給你。

## 六個形狀

借來的樣板，不是職業——同一個職稱可以長成不同形狀。使用者也能自己寫一個。

| | 形狀 | 它標出的代價 |
|---|---|---|
| **S-1** | **深耕的專家**——在一件事上做到很深，被同行認得 | 深度越厚，換軌成本越高 |
| **S-2** | **從零到一的建造者**——一直在把不存在的東西做出來 | 很少東西做到成熟。履歷會看起來跳 |
| **S-3** | **獨立經營者**——時間自己安排，收入自己撐得起來 | 收入不穩，沒有槓桿，雜事全歸自己 |
| **S-4** | **帶人的人**——透過別人把事情放大，而不是自己做完 | 自己動手的技藝會退化，成果變得間接 |
| **S-5** | **穩定的支柱**——工作可預測，重心放在關係與身體 | 職涯天花板會來得比較早，收入成長趨緩 |
| **S-6** | **跨界的連結者**——站在領域之間做翻譯 | 每個領域都不是最深的，得一直解釋自己在做什麼 |

選了不等於承諾。它只是給下一步一個可以對照的基準。

## 接著看

| | |
|---|---|
| [**試用原型**](https://wu0h9625-boop.github.io/guru-intake-prototype/) | 可操作的上傳流程——六個畫面，第五個之前不問任何問題 |
| [**`SYSTEM-DESIGN.md`**](SYSTEM-DESIGN.md) | 規格書：概念模型、統一語彙、三個站、領域模型與不變量、服務、佇列工作、LLM 邊界（英文） |
| [`excalidraw/guru_core_concept.excalidraw`](excalidraw/guru_core_concept.excalidraw) | 上面那張白板的可編輯原始檔 |

## 授權

Proprietary. Copyright (c) 2026 Quasar-Gang, all rights reserved.
版權所有，未經書面同意不得使用、重製、修改或散布。
