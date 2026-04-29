# Stage 2.4 Extension Points

## 概述

此 repo 的「擴充點」有兩個層次：
1. **新增技能**（主要擴充點）
2. **技能的可自訂部分**（次要擴充點）

---

## 主要擴充點：新增技能

新增技能完全不需要修改現有程式碼，只需在根目錄新增一個技能目錄即可：

```bash
mkdir my-new-skill/
touch my-new-skill/SKILL.md
```

**最小合法 SKILL.md 結構**（來自 `write-a-skill/SKILL.md`）：
```yaml
---
name: my-new-skill
description: One-sentence capability. Use when [specific triggers].
---

# My New Skill

## Quick start

[Minimal working example]
```

### 新增技能的設計規則

| 規則 | 說明 |
|------|------|
| `name` 必須與目錄名稱一致 | Claude Code 的技能載入機制要求 |
| `description` ≤ 1024 字元 | 硬性限制，超過會被截斷 |
| SKILL.md ≤ 100 行 | 超過時抽取到輔助文件 |
| 不含時效性資訊 | skill 不應有過期日期 |
| 術語一致性 | 保持 repo 內術語一致 |
| 包含具體 triggers | "Use when..." 是必要的 |

---

## 次要擴充點：技能的可自訂部分

### git-guardrails-claude-code — 可自訂的危險指令清單

`SKILL.md` 第 4 步驟明確提到：

> "Ask if user wants to add or remove any patterns from the blocked list. Edit the copied script accordingly."

使用者可以修改 `block-dangerous-git.sh` 中的 `DANGEROUS_PATTERNS` 陣列：

```bash
DANGEROUS_PATTERNS=(
  "git push"      # 可移除（若允許 push）
  "git reset --hard"
  "git clean -fd"
  # 可新增自訂模式：
  # "git stash drop"
  # "git tag -d"
)
```

### git-guardrails-claude-code — 作用域選擇

安裝時可選擇作用域（`SKILL.md` 步驟 1）：
- **Project** (`.claude/settings.json`) — 僅此專案
- **Global** (`~/.claude/settings.json`) — 所有專案

### setup-pre-commit — Package manager 偵測

`SKILL.md` 步驟 1 說明了自動偵測邏輯：
- `package-lock.json` → npm
- `pnpm-lock.yaml` → pnpm
- `yarn.lock` → yarn
- `bun.lockb` → bun

並且步驟 4 說明若 `typecheck` 或 `test` script 不存在，會省略對應的 hook 行。

### obsidian-vault — Vault 路徑

`SKILL.md` 中 hardcode 了一個特定路徑：
```
/mnt/d/Obsidian Vault/AI Research/
```

⚠️ 這是 Matt Pocock 個人的 Obsidian vault 路徑，其他使用者需要修改此技能才能使用。這是 repo 中最不通用的一個技能。

---

## 技能組合模式（Composition Pattern）

部分技能可組合使用，形成更強大的工作流：

```
/github-triage
    └──呼叫──→ /domain-model (深入分析 issue)
    └──可選──→ /design-an-interface (設計實作介面)

/improve-codebase-architecture
    └──整合──→ CONTEXT.md (由 /domain-model 維護)
    └──整合──→ docs/adr/ (由 /domain-model 維護)
    └──可啟動──→ /design-an-interface (透過 INTERFACE-DESIGN.md)

/to-prd
    └──後續──→ /to-issues (將 PRD 拆解為 issues)
    └──後續──→ /tdd (實作每個 issue)
```

### 推薦工作流組合

**完整功能開發流程**：
```
/grill-me → 澄清設計
/to-prd → 建立 PRD
/to-issues → 拆解為 issues
/tdd → 實作每個 issue
```

**Bug 修復流程**：
```
/triage-issue → 根因分析
/tdd → TDD 修復
```

**架構改善流程**：
```
/domain-model → 建立/更新領域術語
/improve-codebase-architecture → 找出深化機會
/design-an-interface → 設計新介面
```

---

## 擴充限制

| 限制 | 說明 |
|------|------|
| 無 plugin registry | 沒有「技能發現」機制，安裝靠 `npx skills@latest add` |
| 無版本管理 | 此 repo 無 semver，更新靠重新安裝 |
| 無技能間程式呼叫 | 技能只能透過「指令中提到其他技能名稱」來組合，無 API |
| 無 runtime hook system | hook 機制（git-guardrails）是 Claude Code 平台功能，非此 repo 自建 |
| `obsidian-vault` 路徑 hardcode | 需手動修改才能使用 |
