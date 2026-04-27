# 2.2 Data Flow（資料流）

## 代表性使用案例：`/tdd` Skill 的完整執行流程

從使用者輸入到最終產出的端對端追蹤。

```
使用者輸入: "/tdd"
      │
      ▼
┌─────────────────────────────────────────────────────────┐
│  Claude Code Runtime                                     │
│  1. 解析斜線指令 "/tdd"                                   │
│  2. 查找 tdd/SKILL.md（frontmatter: name=tdd）           │
│  3. 讀取 SKILL.md 全文 + 任何 bundled reference files    │
│  4. 注入 context（"你是 TDD 專家，請遵照以下指令..."）     │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│  tdd/SKILL.md 指令執行（tdd/SKILL.md:44-108）            │
│                                                          │
│  步驟 1: Planning                                        │
│  ├── Agent 詢問使用者介面變更需求                          │
│  ├── 確認要測試的行為                                      │
│  └── 取得使用者批准                                        │
│                                                          │
│  步驟 2: Tracer Bullet（tdd/SKILL.md:62-69）             │
│  ├── RED: 寫第一個測試 → 確認失敗                          │
│  └── GREEN: 寫最小實作 → 確認通過                          │
│                                                          │
│  步驟 3: Incremental Loop（tdd/SKILL.md:71-85）          │
│  ├── 對每個剩餘行為重複 RED → GREEN                        │
│  └── 每次一個測試                                          │
│                                                          │
│  步驟 4: Refactor（tdd/SKILL.md:87-96）                  │
│  ├── 參考 refactoring.md（重構候選模式）                   │
│  └── 每次重構後執行測試                                    │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
                    產出: 有測試覆蓋的程式碼變更
```

## 代表性使用案例 2：`/github-triage` 的資料流

這個 skill 涉及最複雜的資料流，包含外部 API 互動：

```
使用者: "Show me anything that needs my attention"
      │
      ▼
┌───────────────────────────────────────────────────────┐
│  Skill Context 載入                                    │
│  ├── github-triage/SKILL.md（主流程）                  │
│  ├── AGENT-BRIEF.md（agent brief 規範）                │
│  └── OUT-OF-SCOPE.md（知識庫說明）                     │
└────────────────────┬──────────────────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────────────────┐
│  資料蒐集階段（github-triage/SKILL.md:79-115）         │
│  ├── gh issue list → 讀取所有 issues                   │
│  ├── 篩選：unlabeled / needs-triage / needs-info       │
│  ├── 讀取 .out-of-scope/*.md（先前被拒絕的功能）        │
│  └── 探索 codebase（了解 domain）                      │
└────────────────────┬──────────────────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────────────────┐
│  推薦呈現（github-triage/SKILL.md:87-105）             │
│  ├── 分類推薦（bug / enhancement）                     │
│  ├── 狀態推薦（ready-for-agent / needs-info 等）       │
│  └── 相似 out-of-scope 記錄比對                        │
└────────────────────┬──────────────────────────────────┘
                     │ 使用者決策
                     ▼
┌───────────────────────────────────────────────────────┐
│  Apply Outcome（github-triage/SKILL.md:119-131）       │
│  ├── gh issue edit --add-label <label>                 │
│  ├── gh issue comment --body "<agent-brief>"           │
│  └── 寫入 .out-of-scope/<concept>.md（wontfix 時）     │
└───────────────────────────────────────────────────────┘
```

## Skill 的通用資料流模型

```
輸入層                 處理層                  輸出層
─────────────────────────────────────────────────────
使用者訊息             SKILL.md 指令            GitHub issue
     │                 （+ reference files）         │
對話 context   ──►    Agent reasoning        ──►  程式碼變更
     │                                              │
Codebase              子 agent（若需要）        CONTEXT.md / ADR
（可用 bash 讀取）     （parallel sub-agents）   本機設定檔
```

## 轉換層次

| 層次 | 輸入 | 輸出 | 代表 Skill |
|------|------|------|-----------|
| 收集層 | 使用者描述 / GitHub issue | 結構化需求 | `grill-me`、`domain-model` |
| 規劃層 | 結構化需求 | 執行計畫（issue / PRD） | `to-prd`、`to-issues`、`request-refactor-plan` |
| 執行層 | 計畫 + codebase | 程式碼變更 | `tdd`、`triage-issue` |
| 驗證層 | 程式碼變更 | 通過的測試 | `tdd`（內建） |
| 文件層 | 對話 context | 文件檔案 | `ubiquitous-language`、`obsidian-vault` |
| 設定層 | 使用者偏好 | 設定檔 | `setup-pre-commit`、`git-guardrails-claude-code` |

## 狀態機：`github-triage` 的 Label State Machine

這是 repo 中唯一明確定義狀態機的 skill（`github-triage/SKILL.md:26-51`）：

```
unlabeled ──► needs-triage ──► ready-for-agent
    │              │           ──► ready-for-human
    │              │           ──► needs-info ──► needs-triage（回流）
    │              └──────────► wontfix
    └────────────────────────► ready-for-agent / ready-for-human / wontfix
```
