# API_SURFACE.md — 技能 API 參考（Part 1：Planning & Design + Development）

> 完整參考分為兩部分：本文件涵蓋 Planning & Design、Development 技能。
> Part 2 請見 [API_SURFACE_part2.md](API_SURFACE_part2.md)。

## 總覽表格

| 技能 | 分類 | 觸發條件 | disable-model-invocation | 主要輸出 |
|------|------|---------|--------------------------|---------|
| `/to-prd` | Planning | "create a PRD", "write a PRD" | 否 | GitHub Issue (PRD) |
| `/to-issues` | Planning | "convert to issues", "break into issues" | 否 | 多個 GitHub Issues |
| `/grill-me` | Planning | "grill me", "stress-test", "design" | 否 | 對話（問答） |
| `/design-an-interface` | Planning | "design API", "design interface", "design it twice" | 否 | 介面設計對比報告 |
| `/request-refactor-plan` | Planning | "refactor plan", "plan a refactor" | 否 | GitHub Issue (重構計畫) |
| `/tdd` | Development | "TDD", "red-green-refactor", "test-driven" | 否 | 程式碼 + 測試 |
| `/triage-issue` | Development | "triage", "investigate bug", "root cause" | 否 | GitHub Issue (修復計畫) |
| `/improve-codebase-architecture` | Development | "improve architecture", "refactoring opportunities" | 否 | 架構建議 + CONTEXT.md |
| `/migrate-to-shoehorn` | Development | "shoehorn", "replace `as`", "partial test data" | 否 | 修改後的測試檔案 |
| `/scaffold-exercises` | Development | "scaffold exercises", "create exercise stubs" | 否 | 練習目錄結構 |

---

## Planning & Design 技能

---

### `/to-prd`

**分類**：Planning & Design

**安裝**：
```bash
npx skills@latest add mattpocock/skills/to-prd
```

**觸發條件**：
- 自動觸發：使用者提到「PRD」、「create a product requirements document」、「write a spec」
- 手動呼叫：`/to-prd`
- `disable-model-invocation: false`

**工作流程**：
1. 探索 codebase 了解現狀（若尚未探索）
2. 列出需要建立或修改的主要模組（找深度模組機會）
3. 與使用者確認模組與測試計畫
4. 按模板撰寫 PRD，提交為 GitHub Issue

**GitHub Issue 模板結構**：
```
## Problem Statement
## Solution
## User Stories（格式：As a <actor>, I want <feature>, so that <benefit>）
## Implementation Decisions（模組、介面、架構決策）
## Testing Decisions
## Out of Scope
## Further Notes
```

**輸出 / 副作用**：建立 GitHub Issue（PRD），不詢問使用者預覽。

**依賴外部工具**：`gh issue create`

---

### `/to-issues`

**分類**：Planning & Design

**安裝**：
```bash
npx skills@latest add mattpocock/skills/to-issues
```

**觸發條件**：
- 自動觸發：「break this into issues」、「create implementation tickets」、「convert to issues」
- 手動呼叫：`/to-issues`（可傳入 GitHub Issue 號碼作為參數）

**工作流程**：
1. 取得 context（對話或 `gh issue view <number>`）
2. 可選：探索 codebase
3. 擬定 tracer bullet 垂直切片（HITL / AFK 分類）
4. 向使用者確認切片粒度、依賴關係、HITL/AFK 標記
5. 按依賴順序建立 GitHub Issues

**GitHub Issue 模板結構**：
```
## Parent（若來源為 GitHub Issue）
## What to build（端對端行為描述）
## Acceptance criteria（可驗證的清單）
## Blocked by（依賴關係）
```

**關鍵設計原則**：
- 垂直切片 = 通過所有整合層的完整路徑（schema + API + UI + tests）
- 偏好多個細薄 issue，而非少數粗糙 issue
- AFK（無人監督）優先於 HITL（需人類介入）

**輸出 / 副作用**：多個 GitHub Issues（按依賴順序建立）

**依賴外部工具**：`gh issue create`、`gh issue view`

---

### `/grill-me`

**分類**：Planning & Design

**安裝**：
```bash
npx skills@latest add mattpocock/skills/grill-me
```

**觸發條件**：
- 自動觸發：「grill me」、「stress-test this plan」、「challenge my design」
- 手動呼叫：`/grill-me`

**工作流程**：
1. 遍歷計畫/設計的每個決策分支
2. 對每個問題：提出建議答案
3. 等待使用者逐一回應（一次問一個問題）
4. 若問題可透過 codebase 探索回答 → 直接探索，不問使用者
5. 持續直到達成共識

**關鍵行為**：
- **一次問一個問題**（不批量）
- 為每個問題提供建議答案
- 可自動探索 codebase 解答技術問題

**輸出 / 副作用**：純對話（無文件副作用）。這是最常被單獨使用的技能。

---

### `/design-an-interface`

**分類**：Planning & Design

**安裝**：
```bash
npx skills@latest add mattpocock/skills/design-an-interface
```

**觸發條件**：
- 自動觸發：「design an API」、「design this interface」、「explore interface options」、「design it twice」
- 手動呼叫：`/design-an-interface`

**工作流程**：
1. 收集需求（問題、呼叫者、操作、限制、隱藏 vs 暴露的邊界）
2. 平行啟動 3+ 個 sub-agent，每個獲得不同設計約束：
   - Agent 1：「最小化介面（1-3 個方法）」
   - Agent 2：「最大化靈活性」
   - Agent 3：「最佳化最常見用例」
   - Agent 4（可選）：「以特定 paradigm/library 為靈感」
3. 依序展示每個設計（型別、使用範例、隱藏的複雜度）
4. 比較設計（介面簡單度、通用性、實作效率、深度）
5. 合成最佳設計

**評估標準**（來自 Ousterhout《A Philosophy of Software Design》）：
- 介面簡單度、通用性、實作效率、深度（大介面 vs 薄實作 = 應避免）

**輸出 / 副作用**：純對話 + 介面設計建議（無文件副作用）

**依賴外部工具**：Claude Code Agent tool（平行 sub-agents）

---

### `/request-refactor-plan`

**分類**：Planning & Design

**安裝**：
```bash
npx skills@latest add mattpocock/skills/request-refactor-plan
```

**觸發條件**：
- 自動觸發：「plan a refactor」、「create a refactoring RFC」、「break refactor into steps」
- 手動呼叫：`/request-refactor-plan`

**工作流程**：
1. 詢問問題的詳細描述與解決方案想法
2. 探索 codebase 驗證斷言
3. 提出替代選項並詢問
4. 詳細訪談實作細節
5. 確認精確範圍（改什麼 / 不改什麼）
6. 檢查測試覆蓋率
7. 拆解為 tiny commits 計畫
8. 建立 GitHub Issue

**GitHub Issue 模板結構**：
```
## Problem Statement
## Solution
## Commits（最小 commit 計畫，Martin Fowler 原則）
## Decision Document（模組、介面、架構決策 — 無路徑/程式碼）
## Testing Decisions
## Out of Scope
## Further Notes
```

**輸出 / 副作用**：GitHub Issue（重構計畫）

**依賴外部工具**：`gh issue create`

---

## Development 技能

---

### `/tdd`

**分類**：Development

**安裝**：
```bash
npx skills@latest add mattpocock/skills/tdd
```

**觸發條件**：
- 自動觸發：「TDD」、「test-driven development」、「red-green-refactor」、「write tests first」
- 手動呼叫：`/tdd`

**工作流程**：
```
1. Planning:    確認介面變更、要測試的行為、深度模組機會、可測試介面設計
2. Tracer Bullet: RED（寫第一個測試）→ GREEN（最小程式碼通過）
3. 增量 Loop:   對每個行為重複 RED → GREEN
4. Refactor:    提取重複、深化模組、SOLID（僅在全部 GREEN 後）
```

**明確反模式（水平切片）**：
```
❌ WRONG: RED: test1+test2+test3 → GREEN: impl1+impl2+impl3
✅ RIGHT: RED→GREEN: test1→impl1, RED→GREEN: test2→impl2, ...
```

**評估清單**（每個循環）：
- 測試描述行為，不描述實作
- 僅使用公開介面
- 測試能在內部重構後存活
- 程式碼為通過當前測試的最小量

**參考文件**：
- `tdd/tests.md` — 好/壞測試範例
- `tdd/mocking.md` — Mock 使用指南
- `tdd/deep-modules.md` — 深度模組概念
- `tdd/interface-design.md` — 可測試介面設計
- `tdd/refactoring.md` — 重構候選

**輸出 / 副作用**：修改後的程式碼 + 測試檔案（在目標 codebase 中）

---

### `/triage-issue`

**分類**：Development

**安裝**：
```bash
npx skills@latest add mattpocock/skills/triage-issue
```

**觸發條件**：
- 自動觸發：「triage this bug」、「investigate this issue」、「find root cause」、「file an issue」
- 手動呼叫：`/triage-issue`

**工作流程**（盡量少問使用者）：
1. 取得問題的簡要描述（若無則問：「What's the problem you're seeing?」）
2. 使用 `Agent(subagent_type=Explore)` 深入調查 codebase：
   - Bug 在哪裡發生？（entry points、API responses）
   - 涉及哪條 code path？
   - 為什麼失敗？（根因）
   - 有哪些相關測試和模式？
3. 確定修復方式（最小變更、受影響介面、需驗證的行為）
4. 設計 TDD 修復計畫（有序的 RED-GREEN 循環）
5. 建立 GitHub Issue（不詢問預覽）

**GitHub Issue 模板結構**：
```
## Problem（實際 vs 預期行為 + 重現步驟）
## Root Cause Analysis（行為與契約描述，無路徑/行號）
## TDD Fix Plan（有序的 RED-GREEN 循環列表）
## Acceptance Criteria（可驗證清單）
```

**Durability 原則**：描述行為與契約，不引用路徑/行號，Issue 在重構後仍有效。

**輸出 / 副作用**：GitHub Issue（修復計畫）+ 印出 URL 和根因摘要

**依賴外部工具**：Claude Code Agent tool（Explore）、`gh issue create`

---

### `/improve-codebase-architecture`

**分類**：Development

**安裝**：
```bash
npx skills@latest add mattpocock/skills/improve-codebase-architecture
```

**觸發條件**：
- 自動觸發：「improve architecture」、「find refactoring opportunities」、「make codebase more testable」、「AI-navigable」
- 手動呼叫：`/improve-codebase-architecture`

**工作流程**：
1. 探索（讀取 CONTEXT.md、docs/adr/，然後用 Explore agent 遍歷 codebase）
2. 找出架構摩擦點（淺模組、無局部性、緊耦合、難測試）
3. 展示候選深化機會（檔案、問題、解決方案、效益）
4. 使用者選擇候選項後 → Grilling loop（走設計樹）
5. 副作用（即時發生）：
   - 新術語 → 更新 `CONTEXT.md`
   - 使用者拒絕且有充分理由 → 建立 ADR
   - 需要探索介面選項 → 參見 `INTERFACE-DESIGN.md`

**核心詞彙**（`LANGUAGE.md`）：
- Module、Interface、Seam、Adapter、Depth、Leverage、Locality
- Deletion test：刪除模組後複雜度消失 = 傳遞者；複雜度重新出現 = 有價值

**參考文件**：
- `improve-codebase-architecture/LANGUAGE.md` — 共用詞彙
- `improve-codebase-architecture/DEEPENING.md` — 依賴分類（in-process/local-substitutable/remote/external）
- `improve-codebase-architecture/INTERFACE-DESIGN.md` — 介面設計子流程
- `domain-model/ADR-FORMAT.md`、`domain-model/CONTEXT-FORMAT.md`

**輸出 / 副作用**：建議報告 + 可能更新 `CONTEXT.md`、建立 ADR

**依賴外部工具**：Claude Code Agent tool（Explore）

---

### `/migrate-to-shoehorn`

**分類**：Development

**安裝**：
```bash
npx skills@latest add mattpocock/skills/migrate-to-shoehorn
```

**觸發條件**：
- 自動觸發：「shoehorn」、「replace `as` in tests」、「partial test data」
- 手動呼叫：`/migrate-to-shoehorn`

**工作流程**：
1. 詢問：哪些測試檔案有問題？需要傳入局部資料？需要故意傳錯誤型別？
2. 安裝：`npm i @total-typescript/shoehorn`
3. 搜尋：`grep -r " as [A-Z]" --include="*.test.ts" --include="*.spec.ts"`
4. 遷移：
   - `{...} as Type` → `fromPartial({...})`
   - `{...} as unknown as Type` → `fromAny({...})`
5. 新增 imports、執行型別檢查

**函式對照**：

| 函式 | 使用情境 |
|------|---------|
| `fromPartial()` | 傳入局部資料（通過型別檢查）|
| `fromAny()` | 傳入故意錯誤的資料（保留自動完成）|
| `fromExact()` | 強制完整物件 |

**限制**：僅用於測試程式碼，絕不用於生產程式碼。

**輸出 / 副作用**：修改後的測試檔案

**依賴外部工具**：`npm install`、TypeScript compiler

---

### `/scaffold-exercises`

**分類**：Development

**安裝**：
```bash
npx skills@latest add mattpocock/skills/scaffold-exercises
```

**觸發條件**：
- 自動觸發：「scaffold exercises」、「create exercise stubs」、「set up a new course section」
- 手動呼叫：`/scaffold-exercises`

**工作流程**：
1. 解析計畫（章節名、練習名、variant 類型）
2. 建立目錄（`mkdir -p`）
3. 建立 stub readmes
4. 執行 lint：`pnpm ai-hero-cli internal lint`
5. 修正錯誤直到 lint 通過

**目錄命名規則**：
- Section：`XX-section-name/`（在 `exercises/` 內）
- Exercise：`XX.YY-exercise-name/`（在 section 內）
- Variant：`problem/`、`solution/`、`explainer/`

**Lint 規則摘要**：每個 exercise 需至少一個 variant、readme.md 非空、無 broken links、無 `.gitkeep`。

**輸出 / 副作用**：練習目錄結構 + readme stubs

**依賴外部工具**：`pnpm ai-hero-cli internal lint`（⚠️ Matt Pocock 的個人工具，其他使用者可能無法使用）
