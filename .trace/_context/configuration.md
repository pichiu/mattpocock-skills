# Stage 2.6 設定與環境

## 概述

此 repo 本身沒有設定檔（無 `.env`、無 `config/`、無 feature flags）。

「設定」的概念分為兩個層次：
1. **技能的 frontmatter 設定**（技能本身的設定）
2. **技能安裝後在目標 codebase 的設定**

---

## 技能 Frontmatter 設定

每個 SKILL.md 頂部的 YAML frontmatter 是技能的設定：

```yaml
---
name: skill-name              # 技能識別符（必須與目錄名稱一致）
description: >                # Claude 自動載入的判斷依據（≤1024 字元）
  Capability description. Use when [triggers].
disable-model-invocation: true  # 可選：禁止 Claude 自動載入
---
```

### 各技能 frontmatter 設定一覽

| 技能目錄 | name | disable-model-invocation |
|---------|------|--------------------------|
| `caveman` | caveman | 無 |
| `design-an-interface` | design-an-interface | 無 |
| `domain-model` | domain-model | **true** |
| `edit-article` | edit-article | 無 |
| `git-guardrails-claude-code` | git-guardrails-claude-code | 無 |
| `github-triage` | github-triage | 無 |
| `grill-me` | grill-me | 無 |
| `improve-codebase-architecture` | improve-codebase-architecture | 無 |
| `migrate-to-shoehorn` | migrate-to-shoehorn | 無 |
| `obsidian-vault` | obsidian-vault | 無 |
| `qa` | qa | 無 |
| `request-refactor-plan` | request-refactor-plan | 無 |
| `scaffold-exercises` | scaffold-exercises | 無 |
| `setup-pre-commit` | setup-pre-commit | 無 |
| `tdd` | tdd | 無 |
| `to-issues` | to-issues | 無 |
| `to-prd` | to-prd | 無 |
| `triage-issue` | triage-issue | 無 |
| `ubiquitous-language` | ubiquitous-language | **true** |
| `write-a-skill` | write-a-skill | 無 |
| `zoom-out` | zoom-out | **true** |

---

## 安裝設定（技能安裝在目標 codebase 後）

### 安裝路徑

透過 `npx skills@latest add mattpocock/skills/<skill-name>` 安裝後：
- **專案範圍**：`.claude/skills/<skill-name>/`
- **全域範圍**：`~/.claude/skills/<skill-name>/`

⚠️ 確切安裝路徑未在此 repo 中說明，來自 `skills` CLI（Vercel Labs）的行為。

### git-guardrails-claude-code 的設定

**最重要的設定技能**，因為它會修改 Claude Code 的 hooks 設定。

**設定目標**（根據選擇的作用域）：

| 作用域 | Hook script 位置 | Settings 檔案 |
|--------|-----------------|-------------|
| Project | `.claude/hooks/block-dangerous-git.sh` | `.claude/settings.json` |
| Global | `~/.claude/hooks/block-dangerous-git.sh` | `~/.claude/settings.json` |

**Settings.json 設定片段**（`.claude/settings.json`）：
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

**合併原則**（`SKILL.md` 步驟 3）：
> "If the settings file already exists, merge the hook into existing `hooks.PreToolUse` array — don't overwrite other settings."

---

## 目標 Codebase 的期望設定（技能讀取的設定）

部分技能預期目標 codebase 中存在特定文件：

### domain-model / improve-codebase-architecture 預期

```
/ (repo root)
├── CONTEXT.md              # 單一 context 的術語表
├── CONTEXT-MAP.md          # 多 context repo 的 context 地圖
├── docs/
│   └── adr/               # 架構決策記錄
│       ├── 0001-*.md
│       └── 0002-*.md
└── src/
    ├── ordering/
    │   ├── CONTEXT.md      # 各 context 的術語表（多 context 模式）
    │   └── docs/adr/
    └── billing/
        ├── CONTEXT.md
        └── docs/adr/
```

**載入優先順序**（domain-model skill）：
1. 先讀 `CONTEXT-MAP.md`（若存在）→ 多 context 模式
2. 若無，讀根目錄 `CONTEXT.md` → 單一 context 模式
3. 若兩者皆無，在第一個術語被確立時**延遲建立** CONTEXT.md

### github-triage 預期

```
/ (repo root)
└── .out-of-scope/           # 拒絕的功能請求知識庫
    ├── dark-mode.md
    └── plugin-system.md
```

若不存在，技能會在需要時建立（wontfix enhancement 時）。

### obsidian-vault 預期（Hardcoded）

```
/mnt/d/Obsidian Vault/AI Research/  # Matt Pocock 的個人路徑
├── *.md                             # 扁平結構（不用資料夾）
└── *Index.md                        # index notes
```

⚠️ 此路徑對其他使用者無效，需修改 `obsidian-vault/SKILL.md`。

---

## Feature Flags

此 repo 無 feature flags 系統。唯一接近的機制是 `disable-model-invocation` frontmatter，用於控制技能的觸發方式。

---

## Secrets 管理

此 repo 不涉及任何 secrets 或 API keys。技能指引的外部服務（GitHub、Stripe 等）的認證由 Claude 執行環境或目標系統自行處理：
- GitHub：依賴 `gh` CLI 的既有認證
- 其他外部 API：由技能指引 Claude 使用目標 codebase 的設定

---

## 版本管理

此 repo **無明確的版本管理機制**：
- 無 `package.json`（無 npm semver）
- 更新技能靠重新執行 `npx skills@latest add`
- Git tag 機制⚠️ 未驗證是否存在
