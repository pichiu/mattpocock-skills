# Stage 2.3 核心領域邏輯

## 核心抽象：Skill（技能）

整個 repo 的「心臟」是 **Skill** 這個 abstraction：

```
Skill = SKILL.md 文件
      = YAML frontmatter (元資料)
      + Markdown 指令 (Claude 的執行指南)
      + 可選輔助文件 (reference materials)
```

### SKILL.md 的結構哲學

從 `write-a-skill/SKILL.md` 提取的設計原則：

1. **Description 即介面**：`description` frontmatter 是 skill 的唯一對外介面。Claude 只看 description 決定是否載入。
2. **Progressive disclosure（漸進揭露）**：主要指令在 SKILL.md 本體（≤100 行），複雜細節抽取到獨立參考文件。
3. **Utility scripts 節省 tokens**：確定性操作（驗證、格式化）用 script 實作，而非每次生成程式碼。
4. **Triggers 明確化**：description 的第二句必須是 "Use when [specific triggers]"。

---

## 核心模式 1：垂直切片（Tracer Bullet）

這是最核心的工程理念，貫穿 `tdd`、`to-issues`、`qa` 三個技能：

```
垂直切片 = 通過所有整合層的薄薄一條路徑
         ≠ 水平切片（先完成某一整個層）

好的垂直切片：
  test → route → validation → business logic → persistence → response

壞的水平切片（明確被 tdd/SKILL.md 禁止）：
  寫所有測試 → 寫所有實作
```

**來源文件**：
- `tdd/SKILL.md`：「DO NOT write all tests first, then all implementation. This is 'horizontal slicing'」
- `to-issues/SKILL.md`：「Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)」

---

## 核心模式 2：深度模組（Deep Module）

來自 Ousterhout《A Philosophy of Software Design》，在 `tdd`、`improve-codebase-architecture` 中被引用：

```
深度模組 = 小介面 + 大量實作（高槓桿）
淺層模組 = 大介面 + 薄薄實作（低槓桿，應避免）

          深度                        淺層
┌──────────────┐              ┌───────────────────────┐
│ 小介面        │              │      大介面            │
├──────────────┤              ├───────────────────────┤
│              │              │ 薄薄實作               │
│              │              └───────────────────────┘
│  大量實作     │
│              │
│              │
└──────────────┘
```

**删除測試（Deletion Test）**：「想像刪掉這個模組。如果複雜度消失了，它只是個傳遞者。如果複雜度重新出現在 N 個呼叫者中，它在發揮作用。」

---

## 核心模式 3：Grilling Loop（無情訪談）

`grill-me` 和 `domain-model` 共享的核心邏輯，也被 `github-triage` 和 `improve-codebase-architecture` 引用：

```
啟動 → 遍歷決策樹的每個分支
    → 對每個問題：
        1. 提出建議答案
        2. 等待使用者反饋
        3. 若問題可透過探索 codebase 回答 → 直接探索，不問使用者
    → 持續直到達成共識
    → 副作用（domain-model 版本）：
        - 立即更新 CONTEXT.md（不批量）
        - 視需要建立 ADR
```

**差異**：
- `grill-me`：純訪談，無文件副作用
- `domain-model`：訪談 + 即時更新 CONTEXT.md + 選擇性建立 ADR

---

## 核心模式 4：Label-based State Machine（github-triage）

`github-triage/SKILL.md` 定義了一個明確的狀態機：

```
狀態：
  unlabeled → needs-triage
  unlabeled → ready-for-agent（已充分規格化）
  unlabeled → ready-for-human
  unlabeled → wontfix
  needs-triage → needs-info
  needs-triage → ready-for-agent
  needs-triage → ready-for-human
  needs-triage → wontfix
  needs-info → needs-triage（reporter 回覆後）

每個狀態轉換都有：
  - 觸發者（維護者 vs Skill 自動）
  - 對應動作（label 變更 + comment + 可能的文件）
```

**Label 設計**：
- Category labels（分類）：`bug`、`enhancement`
- State labels（狀態）：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`
- 每個 Issue 恰好有一個 category label + 一個 state label

---

## 核心模式 5：依賴注入（Dependency Injection）設計理念

貫穿 `tdd/mocking.md`、`tdd/interface-design.md`、`improve-codebase-architecture/DEEPENING.md`：

```
好（可測試）：
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

壞（不可測試）：
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

**Adapter 分類**（`DEEPENING.md`）：
1. In-process — 純計算，可直接測試
2. Local-substitutable — 有本地替代品（PGLite、in-memory FS）
3. Remote but owned — 自己的服務，定義 Port 介面
4. True external — 第三方服務，Mock

---

## 核心模式 6：Durable Documentation（耐久性文件）

貫穿 `triage-issue`、`qa`、`request-refactor-plan`、`github-triage/AGENT-BRIEF.md`：

**原則**：文件應在大規模重構後仍然有效。

```
❌ 壞的：
"Open src/types/skill.ts and add a schedule field on line 42"
"The function around line 150 has the issue"

✅ 好的：
"The SkillConfig type should accept an optional schedule field of type CronExpression"
"When a user runs /triage with no arguments, they should see a summary..."
```

**規則**：
- 不引用檔案路徑（會過期）
- 不引用行號（會過期）
- 描述行為與契約，而非實作
- 使用 codebase 的領域語言

---

## 特殊機制：disable-model-invocation

`domain-model` 和 `ubiquitous-language` 使用 `disable-model-invocation: true` 的原因：

1. **副作用性**：這些技能會修改 codebase 文件（CONTEXT.md、ADR、UBIQUITOUS_LANGUAGE.md）
2. **時機重要性**：使用者需要在對的時機主動觸發，而非 Claude 自動決定
3. **影響範圍**：文件的改變會影響整個 team 的共識，需要謹慎

---

## 技能的 Scaffolding 哲學

`write-a-skill/SKILL.md` 的 100-line 限制揭示了一個重要設計決定：

```
SKILL.md 限制在 100 行以內
     │
     │ 超過時 → 抽取到獨立文件
     ▼
skill-name/
├── SKILL.md           (≤100 行，主要流程)
├── REFERENCE.md       (詳細技術細節)
├── EXAMPLES.md        (使用範例)
└── scripts/           (確定性操作的 bash/js script)
    └── helper.js
```

這種結構確保了「description → 觸發 → 核心指令」的路徑儘可能短，同時仍可包含複雜細節。
