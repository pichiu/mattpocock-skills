# 2.4 Extension Points（擴充點）

## 主要擴充點：新增 Skill

整個 repo 的核心 extension point 就是「新增一個 skill 目錄」。`write-a-skill/SKILL.md` 是這個 extension point 的完整說明文件。

### 新增 Skill 的最小步驟

1. 建立目錄 `<skill-name>/`
2. 建立 `SKILL.md`（必須含 frontmatter）
3. （選用）新增 reference files、utility scripts

### SKILL.md frontmatter 的可擴充欄位

目前已知的欄位：
```yaml
---
name: skill-name                    # 必填，唯一識別子
description: "..."                  # 必填，最多 1024 字元
disable-model-invocation: true      # 選填，僅注入文字不觸發 model
---
```

`disable-model-invocation` 是一個輕量的行為開關，讓 skill 從「互動式」變成「文字注入式」。使用此 flag 的 skills：`ubiquitous-language`、`zoom-out`、`domain-model`。

---

## 擴充類型一：新增 Reference Files

當 `SKILL.md` 超過約 100 行，可以將詳細內容拆到同目錄的額外文件。

**最佳範例**：`tdd/` 目錄（`write-a-skill/SKILL.md:42-43`）

```
tdd/
├── SKILL.md          ← 主流程（108 行，在閾值附近）
├── tests.md          ← 好/壞測試範例
├── mocking.md        ← Mock 使用指南
├── deep-modules.md   ← 深模組概念
├── interface-design.md ← 可測試介面設計
└── refactoring.md    ← 重構候選模式
```

SKILL.md 內使用相對連結引用：
```markdown
See [tests.md](tests.md) for examples       ← tdd/SKILL.md:16
See [mocking.md](mocking.md)                ← tdd/SKILL.md:16
```

---

## 擴充類型二：Bundled Utility Scripts

當某些操作需要確定性執行（不依賴 LLM 生成代碼），可以加入 bash script。

**唯一範例**：`git-guardrails-claude-code/scripts/block-dangerous-git.sh`

```
git-guardrails-claude-code/
├── SKILL.md    ← 說明如何安裝 hook
└── scripts/
    └── block-dangerous-git.sh   ← 實際執行的 hook
```

Skill 內引用方式（`git-guardrails-claude-code/SKILL.md:28`）：
```markdown
The bundled script is at: [scripts/block-dangerous-git.sh](scripts/block-dangerous-git.sh)
```

`write-a-skill/SKILL.md:93-98` 明確說明何時加入 script：
> "Add utility scripts when: operation is deterministic, same code would be generated repeatedly, errors need explicit handling"

---

## 擴充類型三：Sub-Agent 平行化模式

多個 skills 建立了「派發 sub-agent 平行執行」的模式，這是一個軟性的 extension point：

**`design-an-interface/SKILL.md:27-41`**：
```
sub-agent 1: "Minimize method count"
sub-agent 2: "Maximize flexibility"
sub-agent 3: "Optimize for most common case"
sub-agent 4: "Take inspiration from [paradigm]"
```

**`improve-codebase-architecture/INTERFACE-DESIGN.md:18-27`**：
派發 3+ 個 sub-agent，每個有不同的設計約束。

**`qa/SKILL.md:22-27`**：
在背景派發 `subagent_type=Explore` 探索 codebase。

**`triage-issue/SKILL.md:20`**：
使用 `Agent tool with subagent_type=Explore` 深入調查。

這個模式讓 skills 可以在不修改核心的情況下，透過 agent 工具的組合擴展能力。

---

## 擴充類型四：Skill 互相引用（Composability）

多個 skills 明確引用其他 skills，形成可組合的 workflow：

```
github-triage/SKILL.md:119 → "/domain-model session"
github-triage/SKILL.md:125 → "[AGENT-BRIEF.md](AGENT-BRIEF.md)"
improve-codebase-architecture/SKILL.md:28 → "[CONTEXT-FORMAT.md](../domain-model/CONTEXT-FORMAT.md)"
improve-codebase-architecture/SKILL.md:77 → "[INTERFACE-DESIGN.md](INTERFACE-DESIGN.md)"
```

**`github-triage` 調用 `domain-model`**（`github-triage/SKILL.md:118`）：
> "If the issue needs to be fleshed out, interview the maintainer. Use the /domain-model skill."

這是 skill-as-composable-unit 最清晰的範例。

---

## 外部 Extension Point：`.out-of-scope/` 知識庫

`github-triage/OUT-OF-SCOPE.md` 定義了一個可擴充的知識庫系統：

```
.out-of-scope/
├── dark-mode.md         ← 概念名稱
├── plugin-system.md
└── graphql-api.md
```

每個概念一個檔案，記錄拒絕原因與歷史 issue 連結。新的 issue 可以匹配到已有記錄，避免重複討論。這是 skill 定義的「持久化知識擴充點」。

---

## 如何新增功能而不動到核心

1. **新 skill 目錄** — 完全獨立，不影響任何現有 skill
2. **新 reference file** — 加入現有 skill 目錄，SKILL.md 添加連結
3. **擴充 DANGEROUS_PATTERNS 陣列** — `block-dangerous-git.sh:6-16`
4. **新增 sub-agent 約束** — 在 `design-an-interface` 或 `improve-codebase-architecture` 的 sub-agent prompt 加入新的設計視角
5. **新增 `.out-of-scope/` 條目** — 在任何使用 `github-triage` 的 repo 中
