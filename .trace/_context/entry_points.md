# Stage 2.1 Entry Points

## 概述

此 repo **沒有傳統意義的程式啟動點**。整個專案是純文件集合（Markdown + YAML + Bash），無 main function、無 server、無 CLI entrypoint（在此 repo 本身內）。

「啟動點」的概念需從**技能的使用方式**角度理解：

## 技能的啟動方式

```
使用者輸入 ──→ Claude Code ──→ 讀取 SKILL.md ──→ 執行指令
     │                              │
     │ (1) 自動觸發                   │  (2) 手動 /slash 觸發
     │ Claude 偵測到符合              │  使用者輸入 /skill-name
     │ description 的情境             │
     ▼                              ▼
description frontmatter         name frontmatter
(自動載入判斷依據)              (slash command 名稱)
```

### 啟動路徑 1：自動觸發（Auto-invocation）

Claude Code 讀取所有已安裝 skill 的 `description` frontmatter，判斷當前對話是否符合觸發條件。

觸發範例：
- 使用者說「grill me on this design」→ Claude 載入 `grill-me/SKILL.md`
- 使用者說「let's do TDD」→ Claude 載入 `tdd/SKILL.md`
- 使用者說「less tokens」→ Claude 載入 `caveman/SKILL.md`

### 啟動路徑 2：手動觸發（Manual slash command）

使用者輸入 `/skill-name`，Claude Code 直接載入對應 SKILL.md。

**必須手動觸發的技能**（`disable-model-invocation: true`）：
- `/domain-model` — 有副作用（修改 CONTEXT.md、建立 ADR 文件）
- `/ubiquitous-language` — 有副作用（寫入 UBIQUITOUS_LANGUAGE.md）
- `/zoom-out` — 行為改變模式，需使用者明確意圖

## Initialization 流程

### 技能層級的初始化

每個 SKILL.md 載入後，Claude 在執行指令前通常會：

1. **探索 codebase**（部分技能）
   - `github-triage`: 讀取 `.out-of-scope/` 目錄、git remote 判斷 repo
   - `improve-codebase-architecture`: 讀取 `CONTEXT.md`、`docs/adr/`（若存在）
   - `domain-model`: 讀取 `CONTEXT.md` 或 `CONTEXT-MAP.md`

2. **收集 context**
   - `triage-issue`: 使用 `Agent(subagent_type=Explore)` 深入調查 codebase
   - `qa`: 在背景啟動 `Agent(subagent_type=Explore)` 學習領域語言

3. **詢問使用者**（部分技能有前置訪談）
   - `design-an-interface`: 先收集需求（問題、呼叫者、限制、操作）
   - `request-refactor-plan`: 詢問詳細問題描述
   - `write-a-skill`: 詢問技能涵蓋範圍、使用案例

### 唯一的可執行 Initialization（bash script）

`git-guardrails-claude-code/scripts/block-dangerous-git.sh` — 這是 repo 中唯一的可執行腳本。

**觸發條件**：Claude Code 的 `PreToolUse` hook，在每次呼叫 `Bash` 工具前執行。

**初始化流程**（當 hook 被安裝後）：
```bash
INPUT=$(cat)                          # 1. 從 stdin 讀取 JSON tool_input
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')  # 2. 解析 command 欄位
# 3. 逐一比對危險指令模式
for pattern in "${DANGEROUS_PATTERNS[@]}"; do
  if echo "$COMMAND" | grep -qE "$pattern"; then
    echo "BLOCKED: ..." >&2           # 4a. 發現危險指令 → 輸出 BLOCKED 到 stderr
    exit 2                            # 4b. exit 2 = 通知 Claude Code 拒絕執行
  fi
done
exit 0                                # 5. 無危險模式 → 允許執行
```

**被攔截的指令清單**（`block-dangerous-git.sh`：第 8-17 行）：
```bash
DANGEROUS_PATTERNS=(
  "git push"
  "git reset --hard"
  "git clean -fd"
  "git clean -f"
  "git branch -D"
  "git checkout \."
  "git restore \."
  "push --force"
  "reset --hard"
)
```

## 技能間的 Cross-reference 關係

部分技能在執行中會「啟動」其他技能（非程式呼叫，而是指令中的文字參照）：

```
github-triage ──呼叫──→ /domain-model session
improve-codebase-architecture ──參照──→ /design-an-interface (INTERFACE-DESIGN.md)
triage-issue ──使用──→ Agent(subagent_type=Explore)
qa ──使用──→ Agent(subagent_type=Explore) [背景]
design-an-interface ──使用──→ Agent tool（平行 sub-agents）
improve-codebase-architecture ──使用──→ Agent(subagent_type=Explore)
```

## 技能的「輸出副作用」（Side Effects）

部分技能在執行後會修改目標 codebase 的檔案：

| 技能 | 副作用 |
|------|-------|
| `domain-model` | 建立/更新 `CONTEXT.md`、建立 `docs/adr/` ADR 文件 |
| `ubiquitous-language` | 寫入 `UBIQUITOUS_LANGUAGE.md` |
| `github-triage` | 建立 `.out-of-scope/*.md` 文件 |
| `git-guardrails-claude-code` | 建立 `.claude/hooks/block-dangerous-git.sh`、修改 `.claude/settings.json` |
| `setup-pre-commit` | 建立 `.husky/pre-commit`、`.lintstagedrc`、`.prettierrc`，修改 `package.json` |
| `scaffold-exercises` | 建立整個練習目錄結構 |
| `triage-issue` | 建立 GitHub Issue |
| `qa` | 建立 GitHub Issue(s) |
| `to-prd` | 建立 GitHub Issue |
| `to-issues` | 建立多個 GitHub Issues |
| `request-refactor-plan` | 建立 GitHub Issue |
