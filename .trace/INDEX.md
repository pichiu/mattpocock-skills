# mattpocock/skills — 專案總覽

## 一段話總結

這是 Matt Pocock（TypeScript 教育者、aihero.dev 創辦人）個人日常使用的 Claude Code agent skills 集合，涵蓋 TDD、架構設計、GitHub issue 管理、git 安全防護等 19 個 skill，透過 `npx skills@latest add` 安裝。目標是讓 AI coding agent 成為工程師開發流程的真正參與者，而非單純的程式碼生成工具。

---

## 技術棧總覽

| 類別 | 技術 | 版本 | 用途 |
|------|------|------|------|
| 格式 | Markdown + YAML frontmatter | — | Skill 定義文件（SKILL.md） |
| 腳本語言 | Bash | — | PreToolUse hook（block-dangerous-git.sh） |
| 外部 CLI | `gh` (GitHub CLI) | — | Issue/PR 管理，多個 skills 使用 |
| 外部 CLI | `git` | — | 版本控制操作 |
| 外部工具 | `jq` | — | JSON 解析（hook script 使用） |
| 安裝工具 | `npx skills@latest` | latest | Skill 安裝/管理（Vercel Labs 維護） |
| 套件管理 | npm / pnpm / yarn / bun | — | 由 `setup-pre-commit` skill 自動偵測 |
| 授權 | MIT | — | 開放使用 |

---

## 關鍵指令速查

### 安裝單一 Skill
```bash
npx skills@latest add mattpocock/skills/<skill-name>
```

### 常用安裝範例
```bash
# 核心工程 skills
npx skills@latest add mattpocock/skills/tdd
npx skills@latest add mattpocock/skills/grill-me
npx skills@latest add mattpocock/skills/git-guardrails-claude-code

# GitHub workflow
npx skills@latest add mattpocock/skills/github-triage
npx skills@latest add mattpocock/skills/triage-issue
npx skills@latest add mattpocock/skills/to-prd
npx skills@latest add mattpocock/skills/to-issues

# 架構改善
npx skills@latest add mattpocock/skills/improve-codebase-architecture
npx skills@latest add mattpocock/skills/design-an-interface
```

### 測試 Git Guardrails Hook
```bash
echo '{"tool_input":{"command":"git push origin main"}}' | .claude/hooks/block-dangerous-git.sh
# 預期：exit 2，BLOCKED 訊息
```

### 驗證 Pre-Commit Hook
```bash
npx lint-staged
```

---

## 文件地圖

| 文件 | 說明 |
|------|------|
| [INDEX.md](./INDEX.md) | 本文件，專案總覽與速查 |
| [CODEBASE_MAP.md](./CODEBASE_MAP.md) | 程式碼地圖與目錄說明 |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | 系統架構、設計決策、Mermaid 圖 |
| [DATA_MODEL.md](./DATA_MODEL.md) | Skill 資料模型與關係 |
| [API_SURFACE.md](./API_SURFACE.md) | 所有 skills 的完整參考 |
| [DEV_GUIDE.md](./DEV_GUIDE.md) | 開發者上手指南 |
| [DISCOVERY_LOG.md](./DISCOVERY_LOG.md) | 探索紀錄、落差分析、待解問題 |

---

## 專案術語表

| 術語 | 定義 |
|------|------|
| **Skill** | 一個包含 SKILL.md 的目錄，定義 agent 的特定工作流程 |
| **Frontmatter** | SKILL.md 頂部的 YAML 區塊（`---` 包圍），包含 name 和 description |
| **Agent Brief** | github-triage skill 產生的結構化規格，供 AFK agent 執行任務使用 |
| **AFK Agent** | 不需人工干預、可自動完成任務的 AI agent（Away From Keyboard） |
| **HITL** | Human-in-the-loop，需要人工決策的工作切片 |
| **Deep Module** | 小介面 + 大量實作的模組（來自 Ousterhout，`improve-codebase-architecture` 核心概念） |
| **Seam** | 可以在不直接修改程式碼的情況下改變行為的位置（來自 Michael Feathers） |
| **Tracer Bullet** | 端對端的最小可驗證切片，TDD 第一個測試/實作循環 |
| **Vertical Slice** | 跨越所有層（schema、API、UI、tests）的薄型功能切片，相對於水平切片 |
| **disable-model-invocation** | Skill frontmatter 旗標，觸發時只注入文字不發起新的 model 推理 |
| **PreToolUse** | Claude Code 的 hook 事件，在工具執行前觸發 |
| **Ubiquitous Language** | DDD 術語，整個團隊共用的統一領域語言 |
| **ADR** | Architecture Decision Record，架構決策記錄 |
| **Deepening** | 將淺模組重構為深模組的過程（`improve-codebase-architecture` 的主要目標） |

---

## Skill 分類索引

### Planning & Design（規劃與設計）
- **to-prd** — 對話轉 PRD，提交為 GitHub issue
- **to-issues** — 計畫/PRD 拆分為 GitHub issues（垂直切片）
- **grill-me** — 無情審問設計直到所有分支解決
- **design-an-interface** — 平行 sub-agent 生成多種介面設計
- **request-refactor-plan** — 重構計畫訪談，產出 tiny commits 計畫
- **domain-model** — DDD 領域模型 grilling，更新 CONTEXT.md 和 ADR

### Development（開發）
- **tdd** — 紅綠重構 TDD 循環（含 5 個參考文件）
- **triage-issue** — Bug triage + TDD 修復計畫，提交為 GitHub issue
- **improve-codebase-architecture** — 尋找深化機會，改善可測試性
- **migrate-to-shoehorn** — TypeScript 測試斷言從 `as` 遷移到 shoehorn
- **scaffold-exercises** — 練習目錄結構生成（aihero.dev 課程用）
- **qa** — 互動 QA session，conversational bug 回報轉 GitHub issue

### Tooling & Setup（工具設定）
- **setup-pre-commit** — Husky + lint-staged + prettier 設置
- **git-guardrails-claude-code** — 危險 git 指令攔截 hook

### Writing & Knowledge（寫作與知識）
- **write-a-skill** — 建立新 skill 的完整指南
- **edit-article** — 文章編輯改寫（按段落重組）
- **ubiquitous-language** — DDD 術語表提取，儲存到 UBIQUITOUS_LANGUAGE.md
- **obsidian-vault** — Obsidian 筆記搜尋、建立、管理

### Utility（工具型）
- **caveman** — 超壓縮溝通模式，減少約 75% token 用量
- **zoom-out** — 請 agent 上移一層抽象，給出模組全貌
- **github-triage** — GitHub issue 狀態機管理（含 agent brief、out-of-scope 機制）
