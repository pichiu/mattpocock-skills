# 2.1 Entry Points（進入點）

## 這個 Repo 的「啟動」概念

這不是傳統的應用程式，沒有 `main()` 函式或 server 啟動。「進入點」指的是**使用者如何觸發一個 skill**。

## Skill 的兩種觸發方式

### 方式一：在 Claude Code 內輸入 `/skill-name`
使用者在 Claude Code 的 chat 介面輸入 `/tdd`、`/grill-me` 等斜線指令，Claude Code 自動載入對應 SKILL.md 並注入 context。

**技術路徑**：
1. 使用者輸入 `/skill-name`
2. Claude Code runtime 在 `~/.claude/skills/` 或 `.claude/skills/` 目錄下查找對應 skill
3. 讀取 `SKILL.md` 的 frontmatter（取得 `name` 和 `description`）
4. 將 SKILL.md 全文內容注入到 model context
5. 若有 `disable-model-invocation: true`，只注入文字，不觸發 model 新的回應

### 方式二：安裝時的「載入」
```bash
npx skills@latest add mattpocock/skills/<skill-name>
```
這個指令將 skill 目錄複製到本機的 Claude Code skills 目錄（通常是 `.claude/skills/` 或 `~/.claude/skills/`）。

## Skill 載入的「初始化」等效行為

### frontmatter 解析（`name` + `description`）
所有 skills 的 SKILL.md 必須包含：
```yaml
---
name: <skill-name>
description: <一句話描述，最多 1024 字元>
---
```
`description` 是 agent 用來判斷「何時觸發此 skill」的關鍵。`write-a-skill/SKILL.md` 明確說明：「description is the only thing your agent sees when deciding which skill to load」。

### `disable-model-invocation: true`（特殊初始化）
三個 skills 設定了此 flag，表示觸發時只注入文字到 context，不發起新的 model 推理：
- `ubiquitous-language/SKILL.md:4`
- `zoom-out/SKILL.md:4`
- `domain-model/SKILL.md:4`

效果：使用者觸發後，skill 的文字內容會出現在 context 中，由使用者的後續輸入繼續驅動對話。

## Bundled Reference Files 的載入

有些 skills 在 SKILL.md 中透過相對連結引用同目錄下的補充文件：

```markdown
See [tests.md](tests.md) for examples       ← tdd/SKILL.md:16
See [DEEPENING.md](DEEPENING.md)            ← improve-codebase-architecture/SKILL.md
```

這些連結表示 agent 在需要時可以（或應該）讀取這些文件。**Claude Code 不會自動預載這些文件**，而是在 agent 執行時根據情境決定是否讀取。

## 唯一的可執行入口點

`git-guardrails-claude-code/scripts/block-dangerous-git.sh` — 這是唯一的可執行腳本（bash）。

**觸發時機**：不是由使用者直接調用，而是作為 Claude Code 的 `PreToolUse` hook：
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "<path>/block-dangerous-git.sh" }]
      }
    ]
  }
}
```

**腳本邏輯**（`block-dangerous-git.sh:1-26`）：
1. 從 stdin 讀取 JSON（格式：`{"tool_input": {"command": "..."}}`）
2. 用 `jq` 解析出 `command` 欄位
3. 比對 DANGEROUS_PATTERNS 陣列
4. 匹配到危險模式 → `exit 2`（阻止執行）
5. 無匹配 → `exit 0`（允許執行）

攔截的模式：`git push`、`git reset --hard`、`git clean -fd`、`git clean -f`、`git branch -D`、`git checkout .`、`git restore .`、`push --force`、`reset --hard`。

## Skill 安裝後的目錄結構（推測）

```
~/.claude/skills/
├── tdd/
│   ├── SKILL.md
│   ├── tests.md
│   ├── mocking.md
│   └── ...
├── grill-me/
│   └── SKILL.md
└── ...
```

⚠️ 未驗證：Claude Code 實際的 skills 目錄位置依照官方文件，可能是 `~/.claude/skills/` 或專案本地的 `.claude/skills/`。
