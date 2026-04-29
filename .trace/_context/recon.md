# Stage 1 偵察報告 (Reconnaissance)

## 1. 專案基本資訊

| 項目 | 內容 |
|------|------|
| 專案名稱 | mattpocock-skills |
| 作者 | Matt Pocock (Total TypeScript 作者) |
| 授權 | MIT (Copyright © 2026 Matt Pocock) |
| 目的 | 供真正工程師使用的 Claude Code Agent Skills 集合 |
| 口號 | "Agent Skills For Real Engineers" — not vibe coding |
| 分發方式 | `npx skills@latest add mattpocock/skills/<skill-name>` |
| GitHub 星數 | 18,000+ stars（⚠️ 來自 web search，非程式碼驗證） |
| 訂閱 | aihero.dev 電子報 (~60,000 訂閱者) |

## 2. 技術棧

| 類別 | 技術 | 說明 |
|------|------|------|
| 主要格式 | Markdown (.md) | 所有 SKILL.md 與參考文件 |
| 前置資料格式 | YAML frontmatter | 每個 SKILL.md 的元資料 |
| Scripting | Bash (.sh) | 僅一個 hook script |
| 分發工具 | `skills` npm CLI | 由 Vercel Labs 維護，`npx skills@latest` |
| 目標平台 | Claude Code | Anthropic 的官方 CLI 工具 |
| 其他支援平台 | OpenCode, Codex, Cursor 等 49 個 agent | ⚠️ 未驗證 |
| 版本控管 | Git | MIT 授權 repo |

**無 package.json、無 build system、無 runtime code（除一個 bash script）。**

## 3. 目錄結構（3 層深度 Annotated Tree）

```
mattpocock-skills/
├── README.md                              # 專案說明 + 所有 skill 列表
├── LICENSE                                # MIT 授權
│
├── caveman/                               # 技能：超壓縮溝通模式
│   └── SKILL.md
│
├── design-an-interface/                   # 技能：平行 sub-agent 介面設計
│   └── SKILL.md
│
├── domain-model/                          # 技能：DDD 領域模型訪談 + CONTEXT.md 維護
│   ├── SKILL.md
│   ├── ADR-FORMAT.md                      # ADR 格式規範
│   └── CONTEXT-FORMAT.md                  # CONTEXT.md 格式規範
│
├── edit-article/                          # 技能：文章編輯與改善
│   └── SKILL.md
│
├── git-guardrails-claude-code/            # 技能：設定危險 git 指令攔截 hook
│   ├── SKILL.md
│   └── scripts/
│       └── block-dangerous-git.sh         # 唯一的可執行 bash script
│
├── github-triage/                         # 技能：GitHub Issue 分類狀態機
│   ├── SKILL.md
│   ├── AGENT-BRIEF.md                     # 如何撰寫 agent brief
│   └── OUT-OF-SCOPE.md                    # .out-of-scope/ 知識庫說明
│
├── grill-me/                              # 技能：無情訪談計畫/設計
│   └── SKILL.md
│
├── improve-codebase-architecture/         # 技能：找出架構深化機會
│   ├── SKILL.md
│   ├── DEEPENING.md                       # 深化候選項目的依賴分類
│   ├── INTERFACE-DESIGN.md               # 平行 sub-agent 介面設計整合
│   └── LANGUAGE.md                        # 共用架構詞彙表
│
├── migrate-to-shoehorn/                   # 技能：測試中 `as` 遷移至 shoehorn
│   └── SKILL.md
│
├── obsidian-vault/                        # 技能：管理 Obsidian 筆記庫
│   └── SKILL.md
│
├── qa/                                    # 技能：互動式 QA 會話 + 自動建立 Issue
│   └── SKILL.md
│
├── request-refactor-plan/                 # 技能：建立重構計畫並提交為 GitHub Issue
│   └── SKILL.md
│
├── scaffold-exercises/                    # 技能：建立課程練習目錄結構
│   └── SKILL.md
│
├── setup-pre-commit/                      # 技能：設定 Husky pre-commit hooks
│   └── SKILL.md
│
├── tdd/                                   # 技能：測試驅動開發 (TDD)
│   ├── SKILL.md
│   ├── deep-modules.md                    # 深度模組概念 (Ousterhout)
│   ├── tests.md                           # 好測試 vs 壞測試範例
│   ├── mocking.md                         # Mock 使用時機與介面設計
│   ├── interface-design.md               # 可測試介面設計原則
│   └── refactoring.md                    # 重構候選項目清單
│
├── to-issues/                             # 技能：將計畫拆解為 GitHub Issues
│   └── SKILL.md
│
├── to-prd/                                # 技能：將對話轉為 PRD + GitHub Issue
│   └── SKILL.md
│
├── triage-issue/                          # 技能：Bug 根因分析 + TDD 修復計畫
│   └── SKILL.md
│
├── ubiquitous-language/                   # 技能：萃取 DDD 通用語言術語表
│   └── SKILL.md
│
├── write-a-skill/                         # 技能：建立新 skill 的元技能
│   └── SKILL.md
│
└── zoom-out/                              # 技能：請 agent 提升抽象層次
    └── SKILL.md
```

**總計：21 個技能目錄、32 個檔案（不含 .git）**

## 4. 架構模式

**Plugin-based flat collection（扁平外掛集合）**：
- 每個技能是一個獨立目錄（plugin）
- 核心格式：`SKILL.md` (YAML frontmatter + Markdown 指令)
- 輔助文件：放在同一目錄下（如 `tdd/tests.md`、`domain-model/ADR-FORMAT.md`）
- 無相互依賴（各 skill 彼此獨立，但部分 skill 在指令中 cross-reference 其他 skill）

## 5. SKILL.md 格式規格

每個 SKILL.md 都有 YAML frontmatter：
```yaml
---
name: skill-name           # 必須與目錄名稱一致
description: >             # Claude 判斷何時載入此 skill 的唯一依據（≤1024 字元）
  ...
disable-model-invocation: true  # 可選：禁止 Claude 自動載入，只允許使用者 /slash 呼叫
---
```

| 欄位 | 必填 | 說明 |
|------|------|------|
| `name` | 建議 | 必須與目錄名稱一致 |
| `description` | 建議 | Claude 自動載入的唯一判斷依據，≤1024 字元 |
| `disable-model-invocation` | 選填 | `true` = 只允許手動 `/skill-name` 呼叫 |

## 6. disable-model-invocation 技能清單

| 技能 | 原因 |
|------|------|
| `domain-model` | 有副作用（更新 CONTEXT.md、建立 ADR），需使用者主動觸發 |
| `ubiquitous-language` | 有副作用（寫入 UBIQUITOUS_LANGUAGE.md），需使用者主動觸發 |
| `zoom-out` | 行為改變性（讓 agent 提升抽象層次），需手動觸發 |

## 7. 技能分類

### Planning & Design（規劃與設計）
| 技能 | 核心功能 |
|------|---------|
| `to-prd` | 從對話 context 生成 PRD，提交為 GitHub Issue |
| `to-issues` | 將計畫/PRD 拆解為 tracer bullet 垂直切片 GitHub Issues |
| `grill-me` | 無情訪談，解決決策樹的每個分支 |
| `design-an-interface` | 平行 sub-agents 生成 3+ 種截然不同的介面設計 |
| `request-refactor-plan` | 建立詳細重構計畫（tiny commits），提交為 GitHub Issue |

### Development（開發）
| 技能 | 核心功能 |
|------|---------|
| `tdd` | TDD red-green-refactor 循環，垂直切片 |
| `triage-issue` | 根因分析 + TDD 修復計畫 GitHub Issue |
| `improve-codebase-architecture` | 找出淺模組深化機會，DDD 術語一致 |
| `migrate-to-shoehorn` | 將測試中的 `as` 替換為 `@total-typescript/shoehorn` |
| `scaffold-exercises` | 建立課程練習目錄結構（ai-hero.dev 用途） |

### Tooling & Setup（工具與設定）
| 技能 | 核心功能 |
|------|---------|
| `setup-pre-commit` | Husky + lint-staged + Prettier pre-commit hooks |
| `git-guardrails-claude-code` | PreToolUse hook 攔截危險 git 指令 |

### Writing & Knowledge（寫作與知識）
| 技能 | 核心功能 |
|------|---------|
| `write-a-skill` | 建立新 skill 的元技能（progressive disclosure 結構） |
| `edit-article` | 重構文章章節，改善清晰度與流暢性 |
| `ubiquitous-language` | 萃取 DDD 通用語言，儲存至 UBIQUITOUS_LANGUAGE.md |
| `domain-model` | 領域模型訪談 + CONTEXT.md/ADR 維護 |
| `obsidian-vault` | 搜尋、建立、管理 Obsidian 筆記（wikilinks）|

### Interaction Mode（互動模式）
| 技能 | 核心功能 |
|------|---------|
| `caveman` | 超壓縮溝通模式，削減 ~75% token 用量 |
| `zoom-out` | 請 agent 提升抽象層次，給出更高視角 |
| `qa` | 互動式 QA 會話，對話式報告 bug 並自動建立 GitHub Issues |
| `github-triage` | GitHub Issue 分類狀態機（label-based state machine）|

## 8. 既有文件掃描

### 既有文件位置
- `README.md` — 頂層說明，列出所有技能與安裝指令
- `domain-model/ADR-FORMAT.md` — ADR 格式規範（供 domain-model、improve-codebase-architecture 使用）
- `domain-model/CONTEXT-FORMAT.md` — CONTEXT.md 格式規範
- `github-triage/AGENT-BRIEF.md` — 如何撰寫 agent brief（供 github-triage 使用）
- `github-triage/OUT-OF-SCOPE.md` — .out-of-scope/ 知識庫說明
- `improve-codebase-architecture/LANGUAGE.md` — 架構詞彙表
- `improve-codebase-architecture/DEEPENING.md` — 深化流程說明
- `improve-codebase-architecture/INTERFACE-DESIGN.md` — 介面設計子流程
- `tdd/deep-modules.md` — 深度模組概念
- `tdd/tests.md` — 好/壞測試範例
- `tdd/mocking.md` — Mock 指南
- `tdd/interface-design.md` — 可測試介面設計
- `tdd/refactoring.md` — 重構候選清單

### 文件與程式碼落差

**無發現落差**：此 repo 本身就是文件，不存在「文件說 X，程式碼實際是 Y」的情況。唯一的可執行程式碼是 `git-guardrails-claude-code/scripts/block-dangerous-git.sh`，其行為與 `git-guardrails-claude-code/SKILL.md` 描述完全一致。

## 9. Git 歷史摘要

```
90ea8ee Revise README.md to clarify the purpose of agent skills
77b06d1 Update definition of 'Order' in CONTEXT-FORMAT.md
60aa99c Enhance DEEPENING.md and INTERFACE-DESIGN.md; introduce LANGUAGE.md
949472a Added DDD-awareness to improve-codebase-architecture
1186cf6 Update terminology: replace 'grilling session' with '/domain-model session'
3e251ea Clarify guidance on CONTEXT.md updates
8868f54 Rename write-a-prd to to-prd and prd-to-issues to to-issues
ab45d5e Added domain-model, updates to github-triage, and caveman
1f25956 Add AI disclaimer requirement for GitHub issue comments
651eab0 Add agent brief and out-of-scope documentation
98fecc7 Refactor glossary tables and enhance example dialogue
```

主要演進方向：DDD 整合（CONTEXT.md、ADR、domain-model skill）、術語一致性、github-triage 完善。
