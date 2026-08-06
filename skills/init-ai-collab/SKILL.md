---
name: init-ai-collab
description: Use this skill when the user asks to initialize, scaffold, or set up AI-collaboration files for a new (or existing but not-yet-set-up) project repo — creating CLAUDE.md, README.md, .claude/rules/, and .claude/skills/. Only use on explicit request (e.g. "initialize this repo", "set up AI collab files here") — do not trigger automatically just because a repo lacks these files.
---

# 專案 AI 協作檔案初始化

在一個新的（或尚未設定過的）專案 repo 裡，建立 AI 協作所需的標準檔案與資料夾骨架。

## 執行步驟

1. 確認目前工作目錄是目標專案的 repo 根目錄。
2. 檢查以下路徑是否已存在，**已存在的檔案/資料夾不覆蓋**，跳過並告知使用者：
   - `./CLAUDE.md`
   - `./README.md`
   - `./.claude/rules/`
   - `./.claude/skills/`
3. 對不存在的項目，建立空白檔案／資料夾，不加入任何標題或內容，完全留給使用者自己撰寫：
   - `CLAUDE.md`：空檔案。
   - `README.md`：空檔案。
   - `.claude/rules/`、`.claude/skills/`：建立空資料夾（可加 `.gitkeep` 以便 git 追蹤空資料夾）。
4. 完成後列出實際建立了哪些檔案／資料夾，以及跳過了哪些（因為已存在）。

## 範圍限制

- 只建立空白骨架，不推測或自動填入任何內容（例如標題、技術棧、專案簡介）。
- 不修改、不覆蓋任何已存在的檔案。
- 不處理 `.claude/agents/`、`.claude/commands/`、`.claude/hooks` 等其他類型，除非使用者另外要求。
