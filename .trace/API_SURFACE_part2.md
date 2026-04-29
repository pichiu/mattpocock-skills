# API_SURFACE_part2.md — 技能 API 參考（Part 2：Tooling + Writing + Interaction）

> 此文件為 [API_SURFACE.md](API_SURFACE.md) 的延續，涵蓋 Tooling & Setup、Writing & Knowledge、Interaction Mode 技能。

## 總覽表格（Part 2）

| 技能 | 分類 | 觸發條件 | disable-model-invocation | 主要輸出 |
|------|------|---------|--------------------------|---------|
| `/setup-pre-commit` | Tooling | "pre-commit hooks", "husky", "lint-staged" | 否 | `.husky/`, `.lintstagedrc`, `package.json` 修改 |
| `/git-guardrails-claude-code` | Tooling | "block git push", "dangerous git", "git safety hooks" | 否 | hook script + `settings.json` 修改 |
| `/write-a-skill` | Writing | "create a skill", "write a skill", "build a new skill" | 否 | 新技能目錄 + SKILL.md |
| `/edit-article` | Writing | "edit article", "revise article", "improve article" | 否 | 改寫後的文章 |
| `/ubiquitous-language` | Writing | "domain terms", "glossary", "ubiquitous language", "DDD" | **是** | `UBIQUITOUS_LANGUAGE.md` |
| `/domain-model` | Writing | （僅手動）"domain model", "stress-test plan" | **是** | `CONTEXT.md`, `docs/adr/*.md` |
| `/obsidian-vault` | Writing | "obsidian", "find notes", "create note" | 否 | Obsidian 筆記 |
| `/caveman` | Interaction | "caveman mode", "less tokens", "be brief", /caveman | 否 | 超壓縮對話模式 |
| `/zoom-out` | Interaction | （僅手動）不熟悉的程式碼區域 | **是** | 模組/呼叫者 map |
| `/qa` | Interaction | "QA session", "report bugs", "file issues conversationally" | 否 | 多個 GitHub Issues |
| `/github-triage` | Interaction | "triage issues", "manage issues", "issue workflow" | 否 | Issue labels + comments + `.out-of-scope/` |

---

## Tooling & Setup 技能

---

### `/setup-pre-commit`

**分類**：Tooling & Setup

**安裝**：
```bash
npx skills@latest add mattpocock/skills/setup-pre-commit
```

**觸發條件**：
- 自動觸發：「pre-commit hooks」、「set up husky」、「configure lint-staged」、「add commit-time formatting」
- 手動呼叫：`/setup-pre-commit`

**工作流程**：
1. 偵測 package manager（`package-lock.json`→npm / `pnpm-lock.yaml`→pnpm / `yarn.lock`→yarn / `bun.lockb`→bun）
2. 安裝：`husky lint-staged prettier`（devDependencies）
3. 初始化 Husky：`npx husky init`
4. 建立 `.husky/pre-commit`（含 lint-staged、typecheck、test）
5. 建立 `.lintstagedrc`
6. 建立 `.prettierrc`（若不存在）
7. 驗證：`npx lint-staged`
8. Commit：`Add pre-commit hooks (husky + lint-staged + prettier)`

**`.husky/pre-commit` 模板**：
```
npx lint-staged
npm run typecheck  # 若 package.json 無此 script 則省略
npm run test       # 若 package.json 無此 script 則省略
```

**`.lintstagedrc` 模板**：
```json
{"*": "prettier --ignore-unknown --write"}
```

**`.prettierrc` 預設值**：
```json
{
  "useTabs": false, "tabWidth": 2, "printWidth": 80,
  "singleQuote": false, "trailingComma": "es5", "semi": true, "arrowParens": "always"
}
```

**輸出 / 副作用**：`.husky/pre-commit`、`.lintstagedrc`、`.prettierrc`（若無）、修改 `package.json` 的 `prepare` script

**依賴外部工具**：npm/pnpm/yarn/bun、`npx husky`

---

### `/git-guardrails-claude-code`

**分類**：Tooling & Setup

**安裝**：
```bash
npx skills@latest add mattpocock/skills/git-guardrails-claude-code
```

**觸發條件**：
- 自動觸發：「block git push」、「prevent dangerous git」、「add git safety hooks」、「git guardrails」
- 手動呼叫：`/git-guardrails-claude-code`

**工作流程**：
1. 詢問作用域：Project (`.claude/settings.json`) 或 Global (`~/.claude/settings.json`)
2. 複製 `scripts/block-dangerous-git.sh` 到目標位置，`chmod +x`
3. 將 hook 設定合併到 settings.json（不覆寫現有設定）
4. 詢問是否自訂攔截清單
5. 驗證：`echo '{"tool_input":{"command":"git push origin main"}}' | <path-to-script>`

**攔截的危險指令**（`block-dangerous-git.sh`）：
```
git push, git reset --hard, git clean -fd, git clean -f,
git branch -D, git checkout ., git restore ., push --force, reset --hard
```

**Hook 輸入/輸出**：
- 輸入（stdin）：`{"tool_input":{"command":"<command>"}}`
- 允許執行：exit 0
- 攔截：exit 2 + stderr 訊息

**Settings.json 片段**（Project 作用域）：
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{"type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-dangerous-git.sh"}]
    }]
  }
}
```

**輸出 / 副作用**：hook script 文件 + `.claude/settings.json` 或 `~/.claude/settings.json` 修改

---

## Writing & Knowledge 技能

---

### `/write-a-skill`

**分類**：Writing & Knowledge

**安裝**：
```bash
npx skills@latest add mattpocock/skills/write-a-skill
```

**觸發條件**：
- 自動觸發：「create a new skill」、「write a skill」、「build a skill」
- 手動呼叫：`/write-a-skill`

**工作流程**：
1. 收集需求（任務/領域、使用案例、是否需要 script、參考材料）
2. 起草技能（SKILL.md + 輔助文件 + scripts/）
3. 與使用者確認（涵蓋所有用例？缺少什麼？）

**SKILL.md 結構規則**：
- `description`：1024 字元以內，三人稱，第一句說能力，第二句說「Use when [triggers]」
- SKILL.md 本體：≤100 行；超過時抽取到獨立文件
- scripts/：用於確定性操作（驗證、格式化），節省 token

**何時新增 scripts**：操作是確定性的、重複生成同樣程式碼、錯誤需要明確處理

**Review 清單**：description 含觸發詞、SKILL.md ≤100 行、無時效性資訊、術語一致、包含具體範例、引用一層深

**輸出 / 副作用**：新技能目錄 + SKILL.md（+ 可選輔助文件）

---

### `/edit-article`

**分類**：Writing & Knowledge

**安裝**：
```bash
npx skills@latest add mattpocock/skills/edit-article
```

**觸發條件**：
- 自動觸發：「edit this article」、「revise my draft」、「improve this article」
- 手動呼叫：`/edit-article`

**工作流程**：
1. 將文章依標題分割為章節，思考各章節的主要論點
2. 以資訊有向無環圖（DAG）排列章節，確保依賴順序正確
3. 確認章節結構與使用者
4. 對每個章節：改寫以改善清晰度、連貫性、流暢性（每段 ≤240 字元）

**輸出 / 副作用**：改寫後的文章（對話中呈現）

---

### `/ubiquitous-language`

**分類**：Writing & Knowledge

**安裝**：
```bash
npx skills@latest add mattpocock/skills/ubiquitous-language
```

**觸發條件**：
- **僅手動呼叫**（`disable-model-invocation: true`）：`/ubiquitous-language`
- 也觸發於：使用者提到「domain terms」、「define terminology」、「ubiquitous language」、「DDD」

**工作流程**：
1. 掃描對話，找出領域相關名詞、動詞、概念
2. 識別問題：同詞不同意義（歧義）、不同詞同義（同義詞）、模糊詞彙
3. 提出規範術語表（有主見的選擇）
4. 寫入 `UBIQUITOUS_LANGUAGE.md`
5. 在對話中輸出摘要

**`UBIQUITOUS_LANGUAGE.md` 格式**：
- 依領域分組的術語表格（Term / Definition / Aliases to avoid）
- Relationships 章節（基數關係）
- Example dialogue（3-5 交換的開發者 vs 領域專家對話）
- Flagged ambiguities（衝突術語）

**重新執行行為**：讀取既有 `UBIQUITOUS_LANGUAGE.md`，合併新術語，更新定義，重新標注歧義。

**輸出 / 副作用**：建立或更新 `UBIQUITOUS_LANGUAGE.md`

---

### `/domain-model`

**分類**：Writing & Knowledge

**安裝**：
```bash
npx skills@latest add mattpocock/skills/domain-model
```

**觸發條件**：
- **僅手動呼叫**（`disable-model-invocation: true`）：`/domain-model`

**工作流程**（無情訪談 + 即時文件更新）：
1. 探索既有文件（CONTEXT.md 或 CONTEXT-MAP.md、docs/adr/）
2. 無情訪談，解決計畫的每個決策分支
3. 副作用（即時發生，不批量）：
   - 術語確立 → 立即更新 `CONTEXT.md`
   - 術語與 CONTEXT.md 衝突 → 立即指出
   - 決策符合 ADR 標準（難以逆轉 + 令人驚訝 + 有真實 trade-off）→ 提供建立 ADR
4. 用具體場景壓力測試，用程式碼交叉驗證

**文件結構（目標 codebase）**：
- 單一 context：根目錄 `CONTEXT.md`
- 多 context：`CONTEXT-MAP.md` + 各 `src/<context>/CONTEXT.md`
- ADR：`docs/adr/NNNN-slug.md`（延遲建立）

**輸出 / 副作用**：建立或更新 `CONTEXT.md`、選擇性建立 `docs/adr/*.md`

---

### `/obsidian-vault`

**分類**：Writing & Knowledge

**安裝**：
```bash
npx skills@latest add mattpocock/skills/obsidian-vault
```

**⚠️ 路徑 Hardcode 警告**：Vault 路徑 hardcode 為 `/mnt/d/Obsidian Vault/AI Research/`（Matt Pocock 的 WSL2 路徑），其他使用者需修改 SKILL.md 才能使用。

**觸發條件**：
- 自動觸發：「obsidian」、「find note」、「create note」、「manage notes」
- 手動呼叫：`/obsidian-vault`

**工作流程**：
- 搜尋筆記：`find` + `grep -rl`
- 建立筆記：Title Case 命名、加 wikilinks、加索引連結
- 找相關：`grep -rl "[[Note Title]]"`

**命名慣例**：Title Case、無資料夾（扁平結構）、Index notes（`*Index.md`）

**輸出 / 副作用**：建立或修改 Obsidian 筆記（`.md` 文件）

---

## Interaction Mode 技能

---

### `/caveman`

**分類**：Interaction Mode

**安裝**：
```bash
npx skills@latest add mattpocock/skills/caveman
```

**觸發條件**：
- 自動觸發：「caveman mode」、「talk like caveman」、「use caveman」、「less tokens」、「be brief」
- 手動呼叫：`/caveman`
- 關閉：「stop caveman」、「normal mode」

**行為**：
- **一旦啟動，每個 response 都維持**（不自動停用）
- 刪除：冠詞(a/an/the)、填充詞(just/really/basically)、客套語(sure/certainly/happy to)、猶豫語
- 保留：所有技術術語、程式碼區塊、精確錯誤訊息
- 縮寫：DB/auth/config/req/res/fn/impl
- 因果以箭頭表示：X → Y

**例外**：安全警告、不可逆操作確認、多步驟序列易混淆時 → 暫時恢復正常，完成後回到 caveman 模式

**輸出 / 副作用**：改變對話溝通模式（無文件副作用）

---

### `/zoom-out`

**分類**：Interaction Mode

**安裝**：
```bash
npx skills@latest add mattpocock/skills/zoom-out
```

**觸發條件**：
- **僅手動呼叫**（`disable-model-invocation: true`）：`/zoom-out`

**行為**：
指示 Claude「提升一層抽象層次，給出所有相關模組和呼叫者的地圖」。

這是一個極簡技能（SKILL.md 僅一行指令），用於使用者對某段程式碼不熟悉、需要理解它如何融入大局時。

**輸出 / 副作用**：提升抽象層次的 codebase 解說（對話中）

---

### `/qa`

**分類**：Interaction Mode

**安裝**：
```bash
npx skills@latest add mattpocock/skills/qa
```

**觸發條件**：
- 自動觸發：「QA session」、「report bugs」、「file issues conversationally」
- 手動呼叫：`/qa`

**工作流程**（每個回報的問題）：
1. 聆聽並少量澄清（最多 2-3 個短問題：預期 vs 實際、重現步驟、是否一致）
2. 在背景啟動 `Agent(subagent_type=Explore)` 了解相關 codebase 區域和領域語言
3. 評估：單一 Issue 或需要拆解？
4. 建立 GitHub Issue（不詢問預覽）
5. 繼續（每個問題獨立）

**拆解時機**：修復跨多個獨立區域、有清晰可分離的關注點、可以平行處理

**GitHub Issue 規則**：
- 無檔案路徑/行號（會過期）
- 使用專案領域語言
- 描述行為而非程式碼
- 重現步驟必填
- 30 秒內可讀完

**GitHub Issue 模板（單一 Issue）**：
```
## What happened
## What I expected
## Steps to reproduce
## Additional context
```

**輸出 / 副作用**：多個 GitHub Issues（含阻塞關係摘要）

**依賴外部工具**：Claude Code Agent tool（背景 Explore）、`gh issue create`

---

### `/github-triage`

**分類**：Interaction Mode

**安裝**：
```bash
npx skills@latest add mattpocock/skills/github-triage
```

**觸發條件**：
- 自動觸發：「triage issues」、「manage issue workflow」、「review incoming bugs」
- 手動呼叫：`/github-triage`

**工作流程**（自然語言驅動）：

使用者描述意圖，skill 解讀並執行：
- 「Show me anything that needs attention」→ 概覽三個 bucket
- 「Let's look at #42」→ 處理特定 Issue
- 「Move #42 to ready-for-agent」→ 快速狀態覆寫

**Label 系統**：

| Label | 類型 | 說明 |
|-------|------|------|
| `bug` | Category | 某樣東西壞了 |
| `enhancement` | Category | 新功能或改善 |
| `needs-triage` | State | 維護者需要評估 |
| `needs-info` | State | 等待報告者提供更多資訊 |
| `ready-for-agent` | State | 已充分規格化，可給 AFK agent |
| `ready-for-human` | State | 需要人類實作 |
| `wontfix` | State | 不予處理 |

**狀態機轉換**：
```
unlabeled → needs-triage / ready-for-agent / ready-for-human / wontfix
needs-triage → needs-info / ready-for-agent / ready-for-human / wontfix
needs-info → needs-triage（reporter 回覆後）
```

**AI 免責聲明（強制）**：每個 comment 第一行必須是：
```
> *This was generated by AI during triage.*
```

**參考文件**：
- `github-triage/AGENT-BRIEF.md` — Agent Brief 撰寫指引（Durability 原則）
- `github-triage/OUT-OF-SCOPE.md` — `.out-of-scope/` 知識庫格式

**輸出 / 副作用**：Issue label 變更、Issue comments、可能建立 `.out-of-scope/*.md`、可能呼叫 `/domain-model`

**依賴外部工具**：`gh issue list/view/edit/comment/create`、`git remote`（推斷 repo）
