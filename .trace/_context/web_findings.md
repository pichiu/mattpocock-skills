# Stage 1 Web Search 發現

## 搜尋摘要

| 搜尋查詢 | 關鍵發現 |
|---------|---------|
| mattpocock skills Claude Code npm 2026 | 確認 18k+ GitHub stars，`skills` CLI 由 Vercel Labs 維護 |
| npx skills latest add mattpocock CLI documentation | 確認 skills.sh 官方網站，支援 50+ agent 平台 |
| Matt Pocock aihero.dev newsletter Claude Code | 發現 aihero.dev 部落格，grill-me 爆紅文章 |
| Claude Code SKILL.md format disable-model-invocation | 確認 frontmatter 規格，Anthropic 官方文件 |

---

## 關鍵連結與摘要

### 官方資源

- **GitHub repo**: https://github.com/mattpocock/skills
  - 18,000+ stars，1,500+ forks
  - README 列出所有技能與安裝指令

- **aihero.dev** (Matt Pocock 個人網站/電子報):
  - https://www.aihero.dev/5-agent-skills-i-use-every-day — 5 個每日必用技能說明
  - https://www.aihero.dev/my-grill-me-skill-has-gone-viral — grill-me 爆紅文章
  - https://www.aihero.dev/cohorts/claude-code-for-real-engineers-2026-04 — Claude Code for Real Engineers 課程
  - https://www.aihero.dev/real-world-feature-build-with-claude-code — 實際功能開發案例

- **skills CLI（Vercel Labs 維護）**:
  - https://github.com/vercel-labs/skills — CLI 原始碼
  - https://www.npmjs.com/package/skills — npm 套件
  - https://skills.sh — 官方網站
  - https://skills.sh/mattpocock/skills — Matt Pocock skills 頁面

- **Anthropic 官方文件**:
  - https://code.claude.com/docs/en/skills — Claude Code skills 官方說明

### 社群資源

- https://www.welcomedeveloper.com/posts/the-10-claude-code-skills/ — 10 個實用 Claude Code skills 推薦
- https://medium.com/@unicodeveloper/10-must-have-skills-for-claude-and-any-coding-agent-in-2026-b5451b013051 — 2026 年必備技能
- https://dev.to/toyama0919/managing-ai-agent-skills-with-npx-skills-a-practical-guide-2an8 — npx skills 管理指南
- https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/ — Claude Agent Skills 深度解析

---

## 重要技術發現

### SKILL.md Frontmatter 規格（來源：Anthropic 官方文件）

```yaml
---
name: skill-name              # 必須與目錄名稱一致
description: >                # ≤1024 字元，決定 Claude 何時自動載入
  Brief description. Use when [specific triggers].
disable-model-invocation: true  # 可選：禁止 Claude 自動載入
---
```

- `description` 是 Claude 判斷是否自動載入 skill 的**唯一依據**
- `disable-model-invocation: true` 強制使用者透過 `/skill-name` 手動呼叫
- 當 `disable-model-invocation: true` 時，skill 不會出現在 Claude 自動決策中

### skills CLI 安裝機制

```bash
# 安裝單一技能
npx skills@latest add mattpocock/skills/tdd

# 列出可用技能
npx skills add mattpocock/skills --list

# 安裝特定 agent（Claude Code, Cursor 等）
npx skills add mattpocock/skills/tdd -a claude-code
```

技能安裝後會複製到專案的 `.claude/skills/` 或全域 `~/.claude/skills/` 目錄。

### 定位與競品

- **類似收藏**：VoltAgent/awesome-agent-skills（1000+ 社群技能）
- **平台支援**：Claude Code, OpenCode, Codex, Cursor, VS Code Copilot 等 50+ 個 agent 平台
- **Matt Pocock 的差異化**：強調 "real engineering"（非 vibe coding）、TDD、DDD 整合

### grill-me 的爆紅

Matt Pocock 在部落格上描述 `/grill-me` 的使用案例：
- 在一次課程視頻編輯器功能設計中，Claude 問了 16 個問題
- 有些複雜功能的 session 可長達 30-50 個問題
- 這個技能的核心理念：「先把所有決策解決，再寫程式碼」

### 架構相關

- 技能可 cross-reference 其他技能（例如 `github-triage` 呼叫 `/domain-model`）
- `improve-codebase-architecture` 內嵌了 Ousterhout《A Philosophy of Software Design》的概念
- `tdd` 技能內嵌了對 "horizontal slicing"（反模式）的明確警告

---

## 搜尋未找到的資訊

- `skills` CLI 的具體安裝目標路徑（`.claude/skills/` vs 其他）— 推測但未確認
- 是否有 CI/CD 流程自動發布到 skills.sh
- 技能版本管理機制（此 repo 無 semver）
