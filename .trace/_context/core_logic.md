# 2.3 核心領域邏輯

## 這個 Repo 的「心臟」

這個 repo 不是傳統意義的程式碼，它的核心是**精心設計的 prompt 工程**，體現在以下幾個關鍵設計原則上。

---

## 核心設計原則一：漸進揭露（Progressive Disclosure）

`write-a-skill/SKILL.md:11-37` 明確定義了這個架構：

```
SKILL.md（主要，簡短）
├── 100 行以內
├── 包含快速開始與主要流程
└── 連結到 reference files

REFERENCE.md / EXAMPLES.md（次要，可選）
└── 詳細文件，非日常所需
```

這讓 agent 的 context 不被無關資訊佔滿，同時在需要時可以按需讀取深度文件。

**體現最好的案例**：`tdd/SKILL.md` 只有 108 行，但連結到 5 個深度參考文件（tests.md、mocking.md、deep-modules.md、interface-design.md、refactoring.md）。

---

## 核心設計原則二：行為優先，而非實作（Behavioral, not Procedural）

所有涉及 GitHub issue 的 skills 都強調：
- **不要**引用檔案路徑或行號
- **要**描述行為、介面、契約

這個原則在 `github-triage/AGENT-BRIEF.md:7-20` 中最完整表述：
> "Durability over precision. The codebase will change. Write the brief so it stays useful."

**體現最好的案例**：`triage-issue/SKILL.md:54-55`：
> "Describe modules, behaviors, and contracts instead. The issue should remain useful even after major refactors."

---

## 核心設計原則三：垂直切片（Vertical Slices / Tracer Bullets）

多個 skills 反覆強調垂直切片而非水平切片：

```
WRONG (horizontal):
  RED:   test1, test2, test3
  GREEN: impl1, impl2, impl3

RIGHT (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
```

這個模式在 `tdd/SKILL.md:21-40` 有最完整說明，並在 `to-issues/SKILL.md:23-28` 用於 issue 分解：
> "Each slice delivers a narrow but COMPLETE path through every layer"

---

## 核心設計原則四：深模組（Deep Modules）

`improve-codebase-architecture` 的整套 skill group 圍繞這個概念（來源：John Ousterhout 的《A Philosophy of Software Design》）：

**深模組** = 小介面 + 大量實作（高槓桿）
**淺模組** = 大介面 + 薄實作（應避免）

核心測試方法（`improve-codebase-architecture/LANGUAGE.md:37`）：
> "Deletion test: imagine deleting the module. If complexity vanishes, it was a pass-through."

---

## 核心設計原則五：最小問題數（Minimal Questions）

多個 skills 強調減少不必要的使用者交互：

- `triage-issue/SKILL.md:8`：「This is a mostly hands-off workflow - minimize questions to the user.」
- `triage-issue/SKILL.md:12-14`：「ask ONE question」、「Do NOT ask follow-up questions yet. Start investigating immediately.」
- `qa/SKILL.md:13`：「Ask at most 2-3 short clarifying questions」

---

## 核心抽象概念

### Skill（`write-a-skill/SKILL.md` 定義）
- 最小單位：一個 `SKILL.md` 文件
- 必要元素：YAML frontmatter（name + description）
- 可選元素：reference files、utility scripts
- 核心約束：description 最多 1024 字元

### Agent Brief（`github-triage/AGENT-BRIEF.md` 定義）
```
類型: bug / enhancement
當前行為: <描述>
期望行為: <描述>
關鍵介面: <TypeScript 型別或函式簽名>
驗收條件: [ ] 可測試的清單
範疇外: <明確排除>
```

### CONTEXT.md（`domain-model/CONTEXT-FORMAT.md` 定義）
DDD 術語表，記錄領域語言、關係、範例對話、模糊術語。

### ADR（`domain-model/ADR-FORMAT.md` 定義）
架構決策記錄，記錄「為什麼」而非「什麼」，三個觸發條件：難以撤銷、出乎意料、真實 trade-off。

---

## 唯一的程式碼邏輯

`git-guardrails-claude-code/scripts/block-dangerous-git.sh` 是 repo 中唯一有真正邏輯的程式碼：

```bash
# 核心邏輯（block-dangerous-git.sh:6-23）
DANGEROUS_PATTERNS=("git push" "git reset --hard" ...)
for pattern in "${DANGEROUS_PATTERNS[@]}"; do
  if echo "$COMMAND" | grep -qE "$pattern"; then
    echo "BLOCKED: ..." >&2
    exit 2   # exit 2 = Claude Code 解讀為「阻止執行」
  fi
done
exit 0       # 允許
```

Pattern：簡單的字串比對，無複雜邏輯。輸入格式是 Claude Code 的 PreToolUse JSON（`{"tool_input": {"command": "..."}}`）。

---

## Skills 之間的知識依賴關係

```
A Philosophy of Software Design (Ousterhout)
├── deep-modules.md（tdd/）
├── improve-codebase-architecture/（DEEPENING, LANGUAGE, INTERFACE-DESIGN）
└── design-an-interface/SKILL.md（「Design It Twice」原則）

DDD（Domain-Driven Design）
├── domain-model/SKILL.md
├── domain-model/CONTEXT-FORMAT.md
├── domain-model/ADR-FORMAT.md
├── ubiquitous-language/SKILL.md
└── improve-codebase-architecture/（引用 CONTEXT.md）

Martin Fowler（Refactoring）
└── request-refactor-plan/SKILL.md（「tiny commits」引述）

Michael Feathers（Working Effectively with Legacy Code）
└── improve-codebase-architecture/LANGUAGE.md（「Seam」術語來源）
```
