# mattpocock-skills — 專案總覽

## 一句話總結

**mattpocock-skills** 是 Matt Pocock（Total TypeScript 作者）開源的 Claude Code Agent Skills 集合，提供 21 個可安裝的技能，幫助真正的工程師以結構化、可重複的方式使用 AI coding agent — 強調 TDD、DDD、垂直切片，而非「vibe coding」。

---

## 技術棧總覽

| 類別 | 技術 | 版本 | 用途 |
|------|------|------|------|
| 主要格式 | Markdown (.md) | — | 所有 SKILL.md 與參考文件 |
| 前置資料格式 | YAML frontmatter | — | 技能元資料（name, description, disable-model-invocation）|
| Scripting | Bash (.sh) | — | git 危險指令攔截 hook |
| 分發工具 | `skills` npm CLI | latest | `npx skills@latest add mattpocock/skills/<name>` |
| 目標平台 | Claude Code | — | Anthropic 官方 AI coding CLI |
| 版本控管 | Git | — | MIT 授權，GitHub 公開 repo |

---

## 關鍵指令速查

### 安裝技能（在目標專案中執行）

```bash
# 安裝單一技能
npx skills@latest add mattpocock/skills/tdd

# 安裝多個技能
npx skills@latest add mattpocock/skills/tdd
npx skills@latest add mattpocock/skills/grill-me
npx skills@latest add mattpocock/skills/improve-codebase-architecture

# 列出所有可用技能
npx skills@latest add mattpocock/skills --list
```

### 在 Claude Code 中使用技能

```bash
# 手動呼叫（所有技能皆可）
/tdd
/grill-me
/github-triage
/domain-model   # 必須手動呼叫（disable-model-invocation: true）
/zoom-out        # 必須手動呼叫

# 或在對話中自然觸發（Claude 自動偵測）
"let's do TDD for this feature"
"grill me on this design"
"set up pre-commit hooks"
```

### 開發此 repo（新增/修改技能）

```bash
# 複製 repo
git clone https://github.com/mattpocock/skills
cd skills

# 新增技能
mkdir my-new-skill
cat > my-new-skill/SKILL.md << 'EOF'
---
name: my-new-skill
description: Description here. Use when [triggers].
---

# My New Skill

[Instructions here]
EOF

# 無需 build 步驟，直接 commit + push
git add my-new-skill/
git commit -m "Add my-new-skill"
git push
```

---

## 文件地圖

| 文件 | 說明 |
|------|------|
| [INDEX.md](INDEX.md) | 本文件：專案總覽、快速參考 |
| [CODEBASE_MAP.md](CODEBASE_MAP.md) | 程式碼地圖：目錄結構、「我想改 X 要看哪裡？」速查表 |
| [ARCHITECTURE.md](ARCHITECTURE.md) | 系統架構：技能結構、Mermaid 圖、設計決策 |
| [DATA_MODEL.md](DATA_MODEL.md) | 資料模型：SKILL.md 格式、frontmatter 欄位、文件約定 |
| [API_SURFACE.md](API_SURFACE.md) | API 與介面 Part 1：Planning & Design + Development 技能參考 |
| [API_SURFACE_part2.md](API_SURFACE_part2.md) | API 與介面 Part 2：Tooling + Writing & Knowledge + Interaction 技能參考 |
| [DEV_GUIDE.md](DEV_GUIDE.md) | 開發者指南：環境建置、新增技能、貢獻流程 |
| [DISCOVERY_LOG.md](DISCOVERY_LOG.md) | 探索紀錄：發現、待解問題、技術債 |

---

## 術語表

| 術語 | 定義 |
|------|------|
| **Skill（技能）** | 一個 SKILL.md 文件（含 frontmatter + 指令），告訴 Claude 如何執行特定工作流 |
| **Agent Skill** | 可被 Claude Code 等 AI coding agent 載入的技能 |
| **Slash command** | 使用者輸入 `/skill-name` 手動觸發技能 |
| **Auto-invocation** | Claude 根據 `description` frontmatter 自動判斷並載入技能 |
| **disable-model-invocation** | Frontmatter 欄位，設為 `true` 時禁止 Claude 自動載入 |
| **Tracer bullet** | 通過所有整合層的薄薄一條垂直切片（對應 TDD 中的一個 RED-GREEN 循環）|
| **Deep module** | 介面小、實作複雜的模組（高槓桿），來自 Ousterhout《A Philosophy of Software Design》|
| **Grilling** | 無情訪談模式，逐一解決決策樹的每個分支 |
| **CONTEXT.md** | 領域術語表文件，由 `/domain-model` 和 `/ubiquitous-language` 維護 |
| **ADR** | Architecture Decision Record，架構決策記錄 |
| **Agent brief** | 給 AFK agent 執行的詳細任務說明（在 GitHub Issue 中）|
| **AFK agent** | 無人監督自動執行的 AI agent |
| **HITL** | Human-In-The-Loop，需要人類介入的任務 |
| **Seam** | 可以改變行為的介面位置，不需要在原地修改（來自 Michael Feathers）|
| **Adapter** | 滿足介面（seam）的具體實作 |
| **State machine（github-triage）** | Issue 的 label-based 狀態機：unlabeled → needs-triage → ready-for-agent 等 |
| **Vibe coding** | 隨意、無結構地使用 AI coding（此 repo 明確反對的模式）|

---

## 技能分類速查

### Planning & Design（規劃與設計）
`/to-prd` `/to-issues` `/grill-me` `/design-an-interface` `/request-refactor-plan`

### Development（開發）
`/tdd` `/triage-issue` `/improve-codebase-architecture` `/migrate-to-shoehorn` `/scaffold-exercises`

### Tooling & Setup（工具與設定）
`/setup-pre-commit` `/git-guardrails-claude-code`

### Writing & Knowledge（寫作與知識）
`/write-a-skill` `/edit-article` `/ubiquitous-language` `/domain-model` `/obsidian-vault`

### Interaction Mode（互動模式）
`/caveman` `/zoom-out` `/qa` `/github-triage`
