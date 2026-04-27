# 2.6 設定與環境（Configuration）

## Skill 本身的設定機制

### SKILL.md YAML Frontmatter（唯一的設定格式）

每個 skill 的設定完全由 `SKILL.md` 的 YAML frontmatter 定義：

```yaml
---
name: <skill-name>                  # 必填：唯一識別子，對應斜線指令名稱
description: <一句話描述>            # 必填：最多 1024 字元，agent 選 skill 的依據
disable-model-invocation: true      # 選填：只注入文字，不觸發 model
---
```

**沒有環境變數、沒有 .env、沒有外部 config 檔案**。所有設定都嵌入在 SKILL.md 中。

### `disable-model-invocation` 旗標

| Skill | 旗標位置 | 效果 |
|-------|----------|------|
| `ubiquitous-language` | `SKILL.md:4` | 觸發後直接注入文字，不發起 model 推理 |
| `zoom-out` | `SKILL.md:4` | 同上 |
| `domain-model` | `SKILL.md:4` | 同上 |

---

## Git Guardrails 的 Claude Code Hook 設定

`git-guardrails-claude-code` skill 的安裝會產生以下設定（`SKILL.md:41-79`）：

### 專案層級設定（`.claude/settings.json`）
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

### 全域層級設定（`~/.claude/settings.json`）
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

**優先順序**：專案層級 vs 全域層級由使用者在安裝時選擇（`SKILL.md:24`：「install for this project only or all projects?」）。

---

## Pre-Commit Hook 設定（`setup-pre-commit`）

### `.husky/pre-commit`（生成的設定）
```
npx lint-staged
npm run typecheck
npm run test
```

### `.lintstagedrc`（生成的設定）
```json
{
  "*": "prettier --ignore-unknown --write"
}
```

### `.prettierrc`（若無現有設定則生成）
```json
{
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 80,
  "singleQuote": false,
  "trailingComma": "es5",
  "semi": true,
  "arrowParens": "always"
}
```

---

## 套件管理器偵測（設定推斷）

`setup-pre-commit/SKILL.md:18-20` 定義了偵測邏輯：

```
鎖定檔案               → 使用的套件管理器
package-lock.json      → npm
pnpm-lock.yaml         → pnpm
yarn.lock              → yarn
bun.lockb              → bun
（以上都沒有）         → 預設 npm
```

---

## Obsidian Vault 路徑設定

`obsidian-vault/SKILL.md:9`：
```
/mnt/d/Obsidian Vault/AI Research/
```

這是 **硬編碼的個人路徑**，使用者需要直接修改 SKILL.md 來適應自己的環境。⚠️ 這是一個需要注意的環境差異點。

---

## Scaffold Exercises 的隱含環境需求

`scaffold-exercises/SKILL.md:48-64` 需要：
- `pnpm ai-hero-cli internal lint` — 這個 CLI 只存在於 aihero.dev 課程 repo
- 練習目錄必須在 `exercises/` 子目錄下
- 特定的命名慣例（`XX-section-name/`、`XX.YY-exercise-name/`）

---

## Feature Flags

這個 repo **沒有 feature flags**。所有功能都是靜態的 Markdown 文字，沒有條件邏輯（除了 `block-dangerous-git.sh` 中的 pattern 比對）。

---

## Secrets 管理

這個 repo **沒有任何 secrets**。
- GitHub 認證由 `gh` CLI 自行管理（與本 repo 無關）
- 沒有 API key、沒有環境變數需求
- `migrate-to-shoehorn/SKILL.md:114`：使用 `npm i @total-typescript/shoehorn` 安裝公開套件，不需認證

---

## 設定優先順序摘要

```
Claude Code Hook 設定:
  ~/.claude/settings.json (全域) < .claude/settings.json (專案)

Skill 安裝位置:
  ~/.claude/skills/ (全域) 或 .claude/skills/ (專案)  ⚠️ 未驗證確切路徑

套件管理器:
  bun.lockb > yarn.lock > pnpm-lock.yaml > package-lock.json > npm（預設）

Obsidian 路徑:
  obsidian-vault/SKILL.md 中的硬編碼值（需手動修改）
```
