# Claude 技能建置完整指南

## 目錄

- 介紹
- 第 1 章：基本概念
- 第 2 章：規劃與設計
- 第 3 章：測試與迭代
- 第 4 章：發布與分享
- 第 5 章：模式與疑難排解
- 第 6 章：資源與參考
- 附錄 A：快速檢查清單
- 附錄 B：YAML Frontmatter
- 附錄 C：完整技能範例

---

## 介紹

技能（Skill）是一組指令，會以簡單資料夾形式打包，用來教 Claude 如何處理特定任務或工作流程。技能是客製化 Claude 的最強方式之一。你不需要在每次對話都重講偏好、流程或領域知識，只要教一次，就能反覆使用。

當你有可重複的流程時，技能特別有價值，例如：依規格產生前端設計、用固定方法做研究、產出符合團隊風格文件、或協調多步驟流程。技能可搭配 Claude 內建能力（例如程式執行與文件建立）一起用。對有建置 MCP 整合的人來說，技能能再加上一層知識邏輯，讓原始工具存取變成穩定、可優化的流程。

本指南涵蓋從規劃、結構到測試、發布的完整實作重點。無論你是為自己、團隊或社群建立技能，都能找到可落地的模式與實戰範例。

### 你會學到

- 技能結構的技術要求與最佳實務
- 獨立技能與 MCP 增強型流程的常見模式
- 不同使用情境中可行的設計做法
- 技能的測試、迭代與發布方法

### 適合對象

- 想讓 Claude 穩定遵循特定流程的開發者
- 想把 Claude 用到更進階的重度使用者
- 想在組織內標準化 Claude 工作方式的團隊

### 兩條閱讀路徑

- 做獨立技能：優先看「基本概念」、「規劃與設計」與第 1-2 類用例。
- 做 MCP 增強：優先看「Skills + MCP」與第 3 類用例。

兩條路徑共用同一套技術要求，但你可依情境挑重點。

完成本指南後，你應能在單次工作時段做出可運作的技能。一般約 15-30 分鐘可完成第一版並測試（可搭配 `skill-creator`）。

---

## 第 1 章：基本概念

### 什麼是技能？

技能就是一個資料夾，內容通常包含：

- `SKILL.md`（必要）：以 Markdown 撰寫，含 YAML frontmatter
- `scripts/`（選用）：可執行程式（Python、Bash 等）
- `references/`（選用）：需要時才載入的參考文件
- `assets/`（選用）：產出時用的模板、字型、圖示

### 核心設計原則

#### 1. 漸進式揭露（Progressive Disclosure）

技能採三層載入：

- 第一層（YAML frontmatter）：固定載入到 Claude 系統提示，提供「何時該用此技能」的最小必要資訊。
- 第二層（`SKILL.md` 內文）：Claude 判定相關時才載入完整指令。
- 第三層（連結檔案）：技能資料夾內的其他檔案，僅在需要時才進一步讀取。

這能在保留專業能力的同時，降低 token 消耗。

#### 2. 可組合性（Composability）

Claude 可同時載入多個技能。你的技能應該能和其他技能共存，不要假設自己是唯一能力來源。

#### 3. 可攜性（Portability）

技能可跨 `Claude.ai`、`Claude Code`、`API` 使用。只要環境支援相依需求，一份技能可跨介面重用。

### 給 MCP 開發者：Skills + Connectors

> 若你只做獨立技能且不使用 MCP，可先跳到第 2 章。

若你已有可運作的 MCP 伺服器，最難的連線層已完成。技能是上層知識層：把你已知的流程與最佳實務固化，讓 Claude 能穩定套用。

#### 廚房比喻

- MCP 像專業廚房：提供設備、食材、工具。
- 技能像食譜：提供一步一步做法。
- 兩者結合：使用者不用自己摸索每一步，也能完成複雜任務。

#### MCP 與技能分工

- MCP（連通性）：讓 Claude 連到你的服務（如 Notion、Asana、Linear）
- 技能（知識）：教 Claude 如何高品質使用該服務
- MCP 提供即時資料與工具呼叫
- 技能封裝流程與最佳實務

#### 為何這對 MCP 使用者重要

沒有技能時：

- 使用者接上 MCP 後，不知道下一步
- 常收到「要怎麼做 X」的支援問題
- 每次對話都從零開始
- 提示語不一致造成結果不穩
- 問題其實在流程指引，卻被誤認為 connector 問題

有技能時：

- 需要時可自動啟用預建流程
- 工具使用更一致、可預期
- 每次互動都內建最佳實務
- 明顯降低學習門檻

---

## 第 2 章：規劃與設計

### 從使用情境開始

寫任何程式前，先定義 2-3 個具體用例。

範例：

```text
Use Case: Project Sprint Planning
Trigger: 使用者說「幫我規劃這個 sprint」或「建立 sprint 任務」
Steps:
1. 透過 MCP 從 Linear 取得專案現況
2. 分析團隊速度與容量
3. 建議任務優先順序
4. 在 Linear 建立任務並填入標籤與估點
Result: 完整 sprint 規劃完成
```

可自問：

- 使用者真正要完成什麼？
- 需要哪些多步驟流程？
- 需要哪些工具（內建或 MCP）？
- 哪些領域知識或最佳實務應內建？

### 常見技能用例分類

Anthropic 觀察到三大類：

#### 類別 1：文件與資產產製

用途：穩定產出高品質內容（文件、簡報、App、設計、程式等）。

技巧：

- 內建風格指南與品牌規範
- 使用模板提升一致性
- 交付前套用品質檢查清單
- 優先使用 Claude 內建能力，減少外部依賴

#### 類別 2：流程自動化

用途：需要一致方法論的多步驟流程，含跨 MCP 協作。

技巧：

- 分步流程 + 驗證關卡
- 常見結構模板化
- 內建檢視與改善建議
- 迭代修正循環

#### 類別 3：MCP 增強

用途：在既有 MCP 工具存取上，加上流程知識引導。

技巧：

- 按順序協調多次 MCP 呼叫
- 注入領域專業
- 補足使用者通常不會明確提供的上下文
- 對常見 MCP 錯誤做好處理

### 定義成功標準

如何判斷技能有效？

這些是方向性目標，不是絕對門檻。請追求嚴謹，但也接受早期會帶有主觀評估成分。

量化指標：

- 90% 相關查詢可正確觸發技能
  - 測法：跑 10-20 組應觸發查詢，記錄自動載入率
- 在 X 次工具呼叫內完成流程
  - 測法：比較「有技能 / 無技能」的工具呼叫與 token 用量
- 每個流程 0 次失敗 API 呼叫
  - 測法：看 MCP 日誌、重試率、錯誤碼

質化指標：

- 使用者不用一直提醒下一步
- 流程可完成且少需人工修正
- 跨多次對話結果一致

### 技術要求

#### 檔案結構

```text
your-skill-name/
├── SKILL.md                  # 必要
├── scripts/                  # 選用
│   ├── process_data.py
│   └── validate.sh
├── references/               # 選用
│   ├── api-guide.md
│   └── examples/
└── assets/                   # 選用
    └── report-template.md
```

#### 關鍵規則

`SKILL.md` 命名：

- 必須完全是 `SKILL.md`（區分大小寫）
- `SKILL.MD`、`skill.md` 都不行

技能資料夾命名：

- 必須用 kebab-case，例如 `notion-project-setup`
- 不可空白、底線、大寫

`README.md`：

- 技能資料夾內不要放 `README.md`
- 文件請放在 `SKILL.md` 或 `references/`
- 若放在 GitHub 發布，repo 根層仍建議有 README 給人類讀者

### YAML frontmatter：最重要的區塊

Claude 會用 frontmatter 判斷要不要載入技能。

#### 最小必要格式

```yaml
---
name: your-skill-name
description: 做什麼。當使用者提到 [特定語句] 時使用。
---
```

#### 欄位要求

`name`（必要）：

- 只能 kebab-case
- 不可空白或大寫
- 應與資料夾名一致

`description`（必要）：

- 必須同時寫出：
  - 技能做什麼
  - 何時使用（觸發條件）
- 長度小於 1024 字元
- 不可含 XML 標籤（`<`、`>`）
- 要包含使用者真實會說的任務語句
- 若有關聯檔案類型，也應寫出

`license`（選用）：

- 開源發布時建議填
- 常見為 `MIT`、`Apache-2.0`

`compatibility`（選用）：

- 1-500 字元
- 用來說明環境需求（目標產品、系統套件、網路需求等）

`metadata`（選用）：

- 可放任意 key-value
- 建議至少含 `author`、`version`、`mcp-server`

```yaml
metadata:
  author: ProjectHub
  version: 1.0.0
  mcp-server: projecthub
```

#### 安全限制

frontmatter 禁止：

- XML 角括號（`<`、`>`）
- 技能名稱含 `claude` 或 `anthropic`（保留字）

原因：frontmatter 會進系統提示，惡意內容可能造成指令注入。

### 如何寫出有效技能

#### description 欄位

建議結構：

```text
[做什麼] + [何時用] + [核心能力]
```

好範例（具體可觸發）：

```yaml
description: 分析 Figma 設計檔並產生交接文件。當使用者上傳 .fig、提到「design specs」「component documentation」或「design-to-code handoff」時使用。
```

壞範例（過於模糊）：

```yaml
description: 協助專案。
```

#### 主指令撰寫

frontmatter 後方用 Markdown 撰寫完整流程，建議包含：

- 分步驟流程
- 多個實例
- 常見錯誤與處理方式

範本：

```markdown
---
name: your-skill
description: [...]
---

# 技能名稱

## Instructions
### Step 1: [主要步驟]
說明、命令與預期結果

## Examples
### Example 1: [常見情境]
使用者會說什麼、執行哪些動作、結果是什麼

## Troubleshooting
### Error: [常見錯誤]
Cause: [原因]
Solution: [解法]
```

#### 指令最佳實務

- 具體且可執行，不要只寫抽象建議
- 明確加上錯誤處理步驟
- 清楚引用 `references/` 裡的檔案
- 核心指令留在 `SKILL.md`，長文細節放 `references/`

---

## 第 3 章：測試與迭代

技能測試可依需求選擇嚴謹度：

- `Claude.ai` 手動測試：快、零設定
- `Claude Code` 腳本化測試：可重複驗證
- Skills API 程式化測試：可建立系統性評估套件

品質要求不同，測試深度也不同。小團隊內用技能，和面向大量企業使用者的技能，標準不應相同。

> 專家建議：先把單一高難任務做到穩定，再擴到多案例。

### 建議測試法

#### 1. 觸發測試

目標：確認該觸發時觸發，不該觸發時不觸發。

- ✅ 明確任務可觸發
- ✅ 同義改寫可觸發
- ❌ 無關任務不觸發

#### 2. 功能測試

目標：確認輸出正確。

- 產出有效
- API 呼叫成功
- 錯誤處理正常
- 邊界情境有覆蓋

#### 3. 效能比較

目標：證明技能優於基準。

無技能：

- 每次都要重講指令
- 15 輪來回
- 3 次 API 失敗重試
- 約 12,000 tokens

有技能：

- 自動跑流程
- 僅 2 個釐清問題
- API 失敗 0 次
- 約 6,000 tokens

### 使用 `skill-creator`

`skill-creator` 可幫你快速產生、檢視與迭代技能。

可做：

- 由自然語言產生技能草稿
- 產出格式正確的 `SKILL.md`
- 建議觸發詞與結構
- 找出常見問題（描述模糊、漏觸發詞、結構不佳）
- 提供測試案例建議

注意：`skill-creator` 偏向設計與改善，不會直接代你跑完整自動化評估。

### 依回饋持續迭代

欠觸發訊號：

- 該觸發卻沒載入
- 使用者要手動啟用
- 常被問何時該用

解法：補強 description 的細節、語意與關鍵詞（尤其技術詞）。

過觸發訊號：

- 無關查詢也觸發
- 使用者常關閉技能
- 技能用途混淆

解法：加入負向觸發條件、收斂描述範圍。

執行問題訊號：

- 結果不一致
- API 失敗
- 需要大量人工修正

解法：改善步驟指令，補足錯誤處理。

---

## 第 4 章：發布與分享

技能可讓 MCP 整合更完整。使用者比較不同 connector 時，帶技能的方案通常更快產生價值。

### 現行發布模式（2026 年 1 月）

個人使用者安裝方式：

1. 下載技能資料夾
2. 視需要壓縮成 zip
3. 在 `Claude.ai > Settings > Capabilities > Skills` 上傳
4. 或放入 Claude Code 技能目錄

組織層級技能：

- 管理員可工作區層級部署（2025-12-18 上線）
- 支援自動更新
- 集中式管理

### 開放標準

`Agent Skills` 已作為開放標準發布。與 MCP 一樣，技能應可跨平台重用；但若技能依賴特定平台能力，可在 `compatibility` 欄位註明。

### 透過 API 使用技能

程式化場景（應用、代理、自動化流程）可用 API 直接管理與執行技能。

重點能力：

- `/v1/skills`：列出與管理技能
- 透過 `container.skills` 將技能加入 Messages API
- 可在 Claude Console 做版本管控
- 可與 Claude Agent SDK 搭配

何時用 API、何時用 Claude.ai：

- 人直接互動、開發中手動測試、臨時流程：`Claude.ai / Claude Code`
- 大規模生產部署、自動化管線：`API`

> API 中技能需搭配 Code Execution Tool beta。

### 目前建議做法

1. 在 GitHub 建公開 repo（含清楚 README、截圖、用法）。
2. 在 MCP 文件補上技能連結與搭配價值說明。
3. 提供安裝快速指南。

安裝範例：

```text
1. 下載技能（git clone 或 releases ZIP）
2. Claude.ai > Settings > Skills > Upload skill
3. 啟用技能，並確認 MCP 已連線
4. 用一句真實任務測試
```

### 技能定位（Positioning）

描述技能時，重點放在「成果」而非「技術細節」。

好：

> ProjectHub 技能可讓團隊在幾秒內建立完整專案工作區（頁面、資料庫、模板），不用再花 30 分鐘手動設定。

壞：

> ProjectHub 技能是一個含 YAML frontmatter 與 Markdown 指令、會呼叫 MCP 工具的資料夾。

也要講清楚 MCP + Skills 的組合價值。

---

## 第 5 章：模式與疑難排解

以下模式來自早期採用者與內部團隊的實務，不是硬性模板。

### 先問題導向還是先工具導向？

- 問題導向：使用者說「我要完成什麼」，技能決定工具序列。
- 工具導向：使用者先有工具存取，技能補上最佳流程知識。

多數技能會偏其中一邊，先選清楚再設計。

### 模式 1：序列式流程編排

適用：任務需要固定順序的多步驟。

技巧：

- 明確步驟排序
- 定義跨步驟相依
- 每步做驗證
- 失敗時提供回復（rollback）

### 模式 2：多 MCP 協作

適用：流程跨多個服務。

例：Figma 匯出設計 → Drive 存資產 → Linear 建任務 → Slack 通知。

技巧：

- 分階段清楚切割
- 定義資料如何跨 MCP 傳遞
- 下一階段前先驗證
- 集中式錯誤處理

### 模式 3：迭代精修

適用：品質需反覆改進才會好。

技巧：

- 先產初稿
- 用腳本驗證（如 `scripts/check_report.py`）
- 修正問題再重驗
- 設定停止迭代門檻

### 模式 4：情境感知工具選擇

適用：同一成果可能需依情境改用不同工具。

例：

- 大檔（>10MB）：雲端儲存 MCP
- 協作文件：Notion/Docs MCP
- 程式碼：GitHub MCP
- 臨時檔：本地儲存

技巧：

- 清楚決策條件
- 有 fallback 路徑
- 對使用者說明為何這樣選

### 模式 5：領域專業智慧

適用：技能的價值不只在呼叫工具，而在專業判斷。

例：金融合規付款流程。

技巧：

- 動作前先做合規檢核
- 內嵌規則與治理要求
- 保留完整稽核軌跡

### 疑難排解

#### 無法上傳技能

錯誤：`Could not find SKILL.md in uploaded folder`

- 原因：檔名不正確
- 解法：改成 `SKILL.md`，並用 `ls -la` 確認

錯誤：`Invalid frontmatter`

- 原因：YAML 格式錯誤
- 常見：漏 `---`、引號未關閉

錯誤：`Invalid skill name`

- 原因：名稱含空白或大寫
- 解法：改為 kebab-case

#### 技能不會觸發

- 現象：永不自動載入
- 解法：重寫 `description`，加入具體觸發語句與範圍
- 可問 Claude：「什麼情況你會用 [技能名]？」確認判斷依據

#### 技能觸發太頻繁

- 加入負向觸發條件
- 把描述寫得更具體
- 明確劃定範圍（in-scope / out-of-scope）

#### MCP 連線問題

- 確認 Extensions 顯示已連線
- 檢查 API 金鑰、權限、OAuth token
- 先不帶技能直接測 MCP 呼叫
- 確認工具名稱拼字與大小寫

#### 有載入但不照指令做

常見原因：

- 指令太冗長
- 關鍵規則埋太深
- 語意模糊

改善：

- 精簡、條列化
- 關鍵規則放前面（如 `## Important`）
- 對高風險驗證改用腳本實作，降低語言解讀不確定性

#### 大上下文問題

- 現象：變慢或品質下降
- 原因：技能太大、同時啟用太多、未做漸進式揭露
- 解法：
  - `SKILL.md` 控制在 5,000 字內
  - 詳細內容移到 `references/`
  - 同時啟用技能數量避免過多（例如超過 20-50）

---

## 第 6 章：資源與參考

若是第一次做技能，建議先看 Best Practices，再查 API 文件。

### 官方文件

- Best Practices Guide
- Skills Documentation
- API Reference
- MCP Documentation

### 文章

- Introducing Agent Skills
- Engineering Blog: Equipping Agents for the Real World
- Skills Explained
- How to Create Skills for Claude
- Building Skills for Claude Code
- Improving Frontend Design through Skills

### 範例技能

- GitHub：`anthropics/skills`
- 含 Anthropic 官方範例，可直接改成你的版本

### 工具與實用資源

`skill-creator`：

- 內建於 Claude.ai，也可用在 Claude Code
- 可由描述自動產生技能
- 可做檢視並給改善建議

### 取得支援

- 技術問題：Claude Developers Discord 社群
- 錯誤回報：`anthropics/skills/issues`
- 回報時請附：技能名稱、錯誤訊息、重現步驟

---

## 附錄 A：快速檢查清單

### 開始前

- 已定義 2-3 個具體用例
- 已確認工具（內建 / MCP）
- 已讀本指南與範例技能
- 已規劃資料夾結構

### 開發中

- 資料夾命名符合 kebab-case
- `SKILL.md` 存在且拼字正確
- YAML frontmatter 有 `---` 分隔
- `name` 無空白、無大寫
- `description` 同時說明 WHAT + WHEN
- 全文無 XML 標籤（`<`、`>`）
- 指令清楚可執行
- 已含錯誤處理
- 已附範例
- 參考檔案連結清楚

### 上傳前

- 已測試明確任務可觸發
- 已測試改寫語句可觸發
- 已驗證無關題目不會誤觸發
- 功能測試通過
- 工具整合正常（若有）
- 已壓縮為 `.zip`

### 上傳後

- 在真實對話中測試
- 監控欠觸發 / 過觸發
- 收集使用者回饋
- 迭代 description 與指令
- 在 metadata 更新版本

---

## 附錄 B：YAML Frontmatter

### 必填欄位

```yaml
---
name: skill-name-in-kebab-case
description: 這個技能做什麼、什麼時候使用。請包含具體觸發語句。
---
```

### 可選欄位

```yaml
name: skill-name
description: [required description]
license: MIT  # 選填：開源授權
allowed-tools: "Bash(python:*) Bash(npm:*) WebFetch"  # 選填：限制工具存取
metadata:  # 選填：自訂欄位
  author: Company Name
  version: 1.0.0
  mcp-server: server-name
  category: productivity
  tags: [project-management, automation]
  documentation: https://example.com/docs
  support: support@example.com
```

### 安全說明

允許：

- 標準 YAML 型別（字串、數字、布林、陣列、物件）
- 自訂 metadata 欄位
- 長描述（最多 1024 字元）

禁止：

- XML 角括號（`<`、`>`）
- 在 YAML 中執行程式碼（採安全解析）
- 名稱以前綴 `claude` 或 `anthropic` 命名的技能

---

## 附錄 C：完整技能範例

若想看可直接上線的完整範例，可參考：

- Document Skills：PDF、DOCX、PPTX、XLSX 產製
- Example Skills：多種工作流程模式
- Partner Skills Directory：Asana、Atlassian、Canva、Figma、Sentry、Zapier 等夥伴技能

這些 repo 會持續更新，涵蓋本指南以外的更多案例。可直接 clone 後依需求調整成你的版本。
