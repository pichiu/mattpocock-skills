# Stage 1 線上搜尋結果

## 搜尋摘要

### 1. GitHub 官方 Repo
- **URL**: https://github.com/mattpocock/skills
- **Key takeaway**: 這是 Matt Pocock 個人的 `.claude` 目錄 skills，標榜「for real engineers, not vibe coding」。截至搜尋時有大量 star 與 fork。

### 2. Vercel Labs `skills` 工具
- **URL**: https://github.com/vercel-labs/skills
- **Key takeaway**: `npx skills@latest` 是由 Vercel Labs 維護的開放標準工具，不是 Matt Pocock 本人維護。安裝指令格式：`npx skills@latest add <github-user>/skills/<skill-name>`。

### 3. Claude Code 官方文件（Skills）
- **URL**: https://code.claude.com/docs/en/skills
- **Key takeaway**: Claude Code 官方支援 skills 功能，SKILL.md 的 YAML frontmatter 中 `name` 和 `description` 是必填欄位。`disable-model-invocation: true` 可讓 skill 只注入文字而不觸發 model call。

### 4. aihero.dev 文章
- **URL**: https://www.aihero.dev/5-agent-skills-i-use-every-day
- **Key takeaway**: Matt Pocock 親自說明他日常使用的 5 個 skills，強調 skill 是讓 agent 參與開發流程，而非單一指令執行器。

### 5. "Grill Me" Skill 病毒式傳播
- **URL**: https://www.aihero.dev/my-grill-me-skill-has-gone-viral
- **Key takeaway**: `grill-me` 是最受關注的 skill，在社群引起廣泛討論，用來壓力測試設計決策。

### 6. "Claude Code for Real Engineers" 課程
- **URL**: https://www.aihero.dev/cohorts/claude-code-for-real-engineers-2026-04
- **Key takeaway**: Matt Pocock 在 aihero.dev 提供付費課程，這些 skills 是課程材料的一部分。課程強調「溝通、預期、規劃、分解」。

### 7. Agent Skills 生態系（awesome-agent-skills）
- **URL**: https://github.com/VoltAgent/awesome-agent-skills
- **Key takeaway**: 1000+ skills 的 curated 列表，mattpocock/skills 是其中重要的貢獻者。Skills 可跨越 Claude Code、Codex、Gemini CLI、Cursor 等工具使用。

### 8. 社群文章 — "10 Claude Code Skills I Actually Use"
- **URL**: https://www.welcomedeveloper.com/posts/the-10-claude-code-skills/
- **Key takeaway**: 多個開發者已採用這套 skills 到日常工作流程，`tdd`、`git-guardrails-claude-code`、`grill-me` 最常被提及。

### 9. "Skills" NPM 套件
- **URL**: https://www.npmjs.com/package/skills
- **Key takeaway**: `skills` 套件由 Vercel 維護，管理 SKILL.md 的安裝/更新/移除。

### 10. LinkedIn Post（Matt Pocock）
- **URL**: https://www.linkedin.com/posts/mapocock_claude-code-for-real-engineers-activity-7439340061130690560-CfBZ
- **Key takeaway**: Matt Pocock 積極在 LinkedIn 推廣這套 skills 與課程。

## 關鍵 Takeaways

1. **不是框架，是 prompt 工程製品**：每個 skill 本質上是精心設計的指令文本（prompt），透過 YAML frontmatter 讓工具自動識別。

2. **開放標準**：遵循 Vercel Labs 定義的 Agent Skills 開放標準，理論上可移植到其他 coding agent。

3. **社群影響力大**：`grill-me` 已病毒式傳播，`tdd` 和 `git-guardrails` 被廣泛採用。

4. **與課程綁定**：部分 skills（如 `scaffold-exercises`）是為 aihero.dev 的教學課程設計的，不是通用工具。

5. **Newsletter 連結**：https://www.aihero.dev/s/skills-newsletter（約 60,000 訂閱者）

## 相關資源

- [GitHub 官方 Repo](https://github.com/mattpocock/skills)
- [Vercel Labs Skills 工具](https://github.com/vercel-labs/skills)
- [Claude Code Skills 官方文件](https://code.claude.com/docs/en/skills)
- [aihero.dev 部落格](https://www.aihero.dev/posts)
- [Claude Code for Real Engineers 課程](https://www.aihero.dev/cohorts/claude-code-for-real-engineers-2026-04)
- [Awesome Agent Skills](https://github.com/VoltAgent/awesome-agent-skills)
