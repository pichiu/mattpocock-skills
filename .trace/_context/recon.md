# Stage 1 偵察結果

## 專案基本資訊

- **專案名稱**: mattpocock/skills
- **授權**: MIT (Copyright © 2026 Matt Pocock)
- **作者**: Matt Pocock（TypeScript 教育者，aihero.dev 創辦人）
- **定位**: Claude Code 的 agent skill 集合，作者個人日常使用
- **目標受眾**: 使用 Claude Code 的工程師
- **安裝方式**: `npx skills@latest add mattpocock/skills/<skill-name>`

## 技術棧識別

| 類別 | 技術 | 用途 |
|------|------|------|
| 格式 | Markdown (SKILL.md) | Skill 主要指令文件 |
| 前端配置 | YAML frontmatter | Skill metadata（name、description） |
| 腳本 | Bash | git-guardrails 的危險命令攔截 |
| 外部工具 | `gh` CLI | GitHub issue 操作 |
| 外部工具 | `git` | 版本控制操作 |
| 外部工具 | `npx skills` | Skill 安裝工具（Vercel Labs 維護） |
| 圖表 | Mermaid（建議用途）| 不在本 repo，在 skill 指令內提及 |

## 架構模式

**Plugin-based / Content Library**：沒有 runtime 程式碼（除了一個 bash script）。整個 repo 是一個「指令內容庫」，每個子目錄是一個獨立的 skill plugin。

## 目錄結構（3層深）

```
mattpocock-skills/
├── README.md                             ← 所有 skills 的索引與安裝說明
├── LICENSE                               ← MIT
│
├── caveman/
│   └── SKILL.md                         ← 超壓縮溝通模式
│
├── design-an-interface/
│   └── SKILL.md                         ← 平行 sub-agent 介面設計
│
├── domain-model/
│   ├── SKILL.md                         ← DDD 領域模型 grilling
│   ├── ADR-FORMAT.md                    ← ADR 格式規範
│   └── CONTEXT-FORMAT.md               ← CONTEXT.md 格式規範
│
├── edit-article/
│   └── SKILL.md                         ← 文章編輯改寫
│
├── git-guardrails-claude-code/
│   ├── SKILL.md                         ← 安裝危險 git 指令攔截鉤
│   └── scripts/
│       └── block-dangerous-git.sh       ← PreToolUse 鉤的實際 bash script
│
├── github-triage/
│   ├── SKILL.md                         ← GitHub issue 狀態機
│   ├── AGENT-BRIEF.md                   ← agent brief 寫作規範
│   └── OUT-OF-SCOPE.md                 ← .out-of-scope/ 知識庫說明
│
├── grill-me/
│   └── SKILL.md                         ← 無情審問設計
│
├── improve-codebase-architecture/
│   ├── SKILL.md                         ← 尋找深化機會
│   ├── DEEPENING.md                     ← 深化策略指南
│   ├── INTERFACE-DESIGN.md              ← 平行介面設計（用於架構 skill）
│   └── LANGUAGE.md                      ← 架構術語表
│
├── migrate-to-shoehorn/
│   └── SKILL.md                         ← TypeScript 測試斷言遷移
│
├── obsidian-vault/
│   └── SKILL.md                         ← Obsidian 筆記管理
│
├── qa/
│   └── SKILL.md                         ← 互動 QA session
│
├── request-refactor-plan/
│   └── SKILL.md                         ← 重構計畫 RFC
│
├── scaffold-exercises/
│   └── SKILL.md                         ← 練習目錄結構生成
│
├── setup-pre-commit/
│   └── SKILL.md                         ← Husky + lint-staged 設置
│
├── tdd/
│   ├── SKILL.md                         ← TDD 主流程（紅綠重構）
│   ├── deep-modules.md                  ← 深模組概念
│   ├── interface-design.md              ← 可測試介面設計
│   ├── mocking.md                       ← Mock 使用指南
│   ├── refactoring.md                   ← 重構候選模式
│   └── tests.md                         ← 好測試 vs 壞測試範例
│
├── to-issues/
│   └── SKILL.md                         ← 計畫轉 GitHub issue
│
├── to-prd/
│   └── SKILL.md                         ← 對話轉 PRD
│
├── triage-issue/
│   └── SKILL.md                         ← Bug triage + TDD 計畫
│
├── ubiquitous-language/
│   └── SKILL.md                         ← DDD 術語表提取
│
├── write-a-skill/
│   └── SKILL.md                         ← 新 skill 建立指南
│
└── zoom-out/
    └── SKILL.md                         ← 抽象層上移請求
```

## Skill 類型分類

### 1. 一般 Skill（觸發後執行多步驟 workflow）
大多數 skills。觸發後 agent 根據 SKILL.md 執行。

### 2. `disable-model-invocation: true` Skill（純文字注入）
不發起新的 model call，只將指令注入 context：
- `ubiquitous-language/SKILL.md`
- `zoom-out/SKILL.md`
- `domain-model/SKILL.md`

### 3. 包含 Bundled Reference Files 的 Skill
SKILL.md 連結到同目錄下的補充文件：
- `tdd/` — 5 個參考文件（deep-modules, interface-design, mocking, refactoring, tests）
- `improve-codebase-architecture/` — 3 個（DEEPENING, INTERFACE-DESIGN, LANGUAGE）
- `github-triage/` — 2 個（AGENT-BRIEF, OUT-OF-SCOPE）
- `domain-model/` — 2 個（ADR-FORMAT, CONTEXT-FORMAT）

### 4. 包含可執行腳本的 Skill
- `git-guardrails-claude-code/scripts/block-dangerous-git.sh` — Bash script，作為 Claude Code PreToolUse hook

## 既有文件掃描

唯一的 top-level 文件是 `README.md`，作為所有 skills 的索引，無額外 `docs/` 目錄。每個 skill 的 `SKILL.md` 即為其自身的說明文件。

## 既有文件與實際程式碼的落差

| 落差描述 | 說明 |
|----------|------|
| README 未列出 `domain-model`、`caveman`、`qa`、`github-triage`、`zoom-out` | 這些 skills 在 README 中找不到，但目錄存在 |
| `scaffold-exercises` 提及 `pnpm ai-hero-cli internal lint` | 這是特定於 aihero.dev 課程 repo 的工具，不在本 repo 中 |
| `obsidian-vault` 使用硬編碼路徑 `/mnt/d/Obsidian Vault/AI Research/` | 這是 Matt Pocock 個人機器的路徑，直接使用者需修改 |

## 統計

- 總檔案數：36（不含 .git）
- Skill 目錄數：19 個（含 `domain-model`、未列於 README 的 5 個）
- 純 Markdown 檔案：34
- Bash script：1
- 平均每個 skill 的檔案數：約 1.9 個
