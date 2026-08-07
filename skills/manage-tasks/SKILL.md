---
name: manage-tasks
description: Use this skill for any work involving the user's file-based task tracker — creating a task, updating its status or progress, asking how a task is going or what is still missing, asking what to work on next, or listing/filtering tasks. Triggers on phrases like "建立一個 task", "這個 task 進度如何", "還缺什麼", "我完成了 X 接下來做什麼", "有哪些 task". Do not use for ephemeral in-conversation todo lists or for the TodoWrite tool.
---

# 任務追蹤系統

以檔案為儲存方式的任務追蹤系統。每個任務是一個 Markdown 檔案，結構化欄位放在 YAML frontmatter，敘述性內容放在正文。

frontmatter 的欄位值必須嚴格遵守本文件定義的詞彙，因為查詢是靠 `grep` 掃描這些欄位完成的。欄位值一旦自由發揮，查詢就會漏掉任務。

## 儲存位置

**使用者未指定位置時，一律使用專案根目錄下的 `tasks/`。** 這是預設值，不需要為此詢問使用者。

專案根目錄指 git 儲存庫的根目錄，以 `git rev-parse --show-toplevel` 取得。這一步不能省略：在子目錄工作時若直接使用相對路徑 `tasks/`，會在錯誤的位置建立第二個任務目錄，導致後續查詢漏掉任務。不在 git 儲存庫內時，改用目前工作目錄。

只有在使用者明確表示某任務不屬於當前專案（例如稱其為個人任務、跨專案任務）時，才改存到 `~/.claude/tasks/`。

目錄不存在時直接建立，不需要詢問。首次在某個專案建立任務目錄時，一併告知使用者路徑，並提醒這個目錄會被 git 追蹤——若不希望任務納入版控，需自行加入 `.gitignore`。

檔名格式為 `<id>-<英文-slug>.md`，例如 `T-003-user-login.md`。slug 只是方便瀏覽，所有程式性的引用一律使用 `id`。

## 任務結構

任務檔案的骨架定義在與本文件同一目錄下的 `template.md`。建立新任務時以該檔案為起點複製，不要憑記憶重新產生結構。

以下是一份填寫完成的任務，示範各欄位與成功條件實際該寫成什麼樣子：

```markdown
---
id: T-003
title: 實作使用者登入
status: todo
priority: high
scope: M
tags: [auth, backend]
due: 2026-08-20
depends_on: [T-001, T-002]
created: 2026-08-07
updated: 2026-08-07
---

## 任務內容

提供 email 加密碼的註冊與登入流程，登入成功後簽發 JWT 作為後續 API 的憑證。

採用 JWT 而非 session 是為了讓 API 能無狀態地驗證，未來拆分服務時不需共用 session 儲存。密碼雜湊使用 bcrypt。

## 成功條件

- [ ] 使用者能以 email 與密碼註冊，密碼經 bcrypt 雜湊後才寫入資料庫
- [ ] 已註冊的帳號能登入並取得 JWT
- [ ] 持過期的 JWT 呼叫受保護的 API 會得到 401
- [ ] 同一帳號密碼連續錯誤五次後鎖定十分鐘

## 進度紀錄

- 2026-08-07 建立任務
```

注意成功條件的寫法：每一條都指向一個可以實際操作並看到結果的行為，因此任何時候都能明確回答「這條做到了沒有」。若寫成「登入功能完成」「安全性足夠」，之後就無法判斷進度。

### 欄位定義

| 欄位 | 必填 | 允許值 | 說明 |
|------|------|--------|------|
| `id` | 是 | `T-` + 三位數字 | 建立後永不變更。其他任務靠它建立依賴關係。 |
| `title` | 是 | 自由文字 | 一句話說明要完成什麼。 |
| `status` | 是 | `todo` / `in_progress` / `blocked` / `done` / `cancelled` | 只能是這五個值之一。 |
| `priority` | 是 | `high` / `medium` / `low` | 未指定時預設 `medium`。 |
| `scope` | 是 | `S` / `M` / `L` / `XL` | 工作量規模。S 為半天內，M 為數天，L 為一到兩週，XL 為需要拆解。 |
| `tags` | 是 | tag 陣列，例如 `[auth, backend]`；未分類時填 `[]` | 分類標籤，可同時屬於多個。指派前須遵守下方的詞彙一致性規則。 |
| `due` | 否 | `YYYY-MM-DD` 或留空 | 目標達成日期。沒有期限就留空，不要填 `無` 或 `TBD`。 |
| `depends_on` | 是 | id 陣列，例如 `[T-001]`；無依賴時填 `[]` | 列出必須先完成的任務。 |
| `created` | 是 | `YYYY-MM-DD` | 建立日期。 |
| `updated` | 是 | `YYYY-MM-DD` | 每次修改此檔案時同步更新。 |

**成功條件必須寫成 checkbox 清單，且每一條都要能被客觀驗證。** 這一點決定了整個系統能不能回答「還缺什麼」——未勾選的項目就是答案。若成功條件寫成「登入功能做好」這種無法驗證的敘述，之後只能靠猜測回答進度。

### tags 詞彙一致性

`tags` 的內容不預先限定，但**指派任何 tag 前，必須先取得目前已經使用過的 tag**：

```bash
grep -h "^tags:" "$(git rev-parse --show-toplevel)"/tasks/*.md \
  | sed 's/tags: *//' | tr -d '[]' | tr ',' '\n' \
  | sed 's/^ *//;s/ *$//' | sort -u
```

若清單中已有語意相同或相近的 tag，一律沿用既有的，不要新增拼法不同的變體。`auth` 與 `authentication`、`db` 與 `database` 這類同義變體一旦並存，依 tag 篩選就會漏掉任務，而且不會有任何錯誤訊息提示你漏了。

確實屬於新分類時才新增 tag，並在回覆中明確告知使用者「新增了 tag `X`」，讓使用者有機會否決或改名。

tag 的書寫格式：小寫英文、多字以連字號連接（`data-migration`）、使用單數（`api` 而非 `apis`）。

一個任務的 tag 建議控制在三個以內。需要更多才描述得清楚時，通常代表這個任務範圍過大，應考慮拆解。

## 操作

### 建立任務

1. 掃描任務目錄取得現有的最大 id，新 id 為其加一。目錄為空時從 `T-001` 開始。
2. 複製 `template.md` 作為新任務檔案，再逐一替換 `{{ }}` 標記的內容。
3. 依「tags 詞彙一致性」取得現有 tag 清單，從中挑選適用的填入 `tags`。使用者未指定分類時，可依任務內容自行判斷並沿用既有 tag；判斷不出來就留 `[]`，不要勉強新增 tag。
4. 若使用者未提供 `priority`、`scope`、`due` 或 `depends_on`，先以預設值（`medium`、`M`、留空、`[]`）填入並在回覆中說明，不要為了補齊欄位而中斷提問。
5. 使用者若只給了模糊的成功條件，將它改寫成可驗證的 checkbox；改寫後在回覆中列出，讓使用者有機會修正。
6. 寫入後檢查檔案中不再含有 `{{`，確認所有 placeholder 都已替換。
7. 回報 id、檔案路徑、使用了哪些預設值，以及是否新增了任何 tag。

### 更新任務

1. 用 `grep` 依 id 或標題關鍵字定位檔案。
2. 修改對應欄位，同時更新 `updated`。
3. 在「進度紀錄」附加一行 `- YYYY-MM-DD <這次改了什麼>`。

狀態轉換時的額外處理：

- 改為 `done` 前，先確認成功條件是否全部勾選。若仍有未勾選項目，指出是哪幾項，詢問是要一併勾選還是這些條件已不適用。
- 改為 `blocked` 時，必須在進度紀錄寫明被什麼卡住。
- 某任務改為 `done` 後，主動檢查是否有任務的 `depends_on` 包含它，並在回覆中列出因此解除封鎖的任務。

### 查詢單一任務進度

回答「XXX 進度如何、還缺什麼」時，依序提供：

1. 目前 `status`、`priority`、`due`（若已逾期或即將到期要指出）。
2. 成功條件中已勾選與未勾選的項目，未勾選的就是「還缺什麼」。
3. `depends_on` 中尚未 `done` 的任務，這些是外部阻礙。
4. 進度紀錄中最近幾筆。

### 查詢下一步

回答「接下來要做什麼」時，套用以下規則挑選候選任務：

1. 篩選出 `status` 為 `todo`，且 `depends_on` 中所有任務都已 `done` 的任務。這些是目前真正可以開始的。
2. 排序依據：先看 `due`（已逾期或最接近的優先），再看 `priority`，最後看 `scope`（同等條件下小的優先，較快產生進展）。
3. 同時列出 `status` 為 `in_progress` 的任務——手上已經開始的東西通常應該先收尾。
4. 若有任務因依賴而卡住，簡短說明卡在哪個任務上。

建議通常給二到三個，並說明為什麼是這幾個，不要只丟出一份完整清單。

### 列出任務

用 `grep` 對 frontmatter 篩選，例如：

```bash
R="$(git rev-parse --show-toplevel)"
grep -l "^status: in_progress" "$R"/tasks/*.md   # 依狀態
grep -l "^tags:.*\bauth\b" "$R"/tasks/*.md       # 依 tag
```

依 tag 篩選前，先確認該 tag 確實存在於 tag 清單中。若使用者提到的分類名稱與既有 tag 不完全相同（例如問「認證相關的任務」但實際 tag 是 `auth`），先對應到既有 tag 再查詢，並在回覆中說明用了哪個 tag，避免使用者以為沒有結果就是沒有任務。

輸出以表格呈現，欄位為 id、title、status、priority、due。任務數量超過約十五筆時，只顯示與使用者問題相關的部分，並說明總筆數。

## 範圍限制

- 不自行變更任務狀態。狀態只在使用者說明進展時才更新。
- 不自動建立使用者沒要求的任務。從對話中察覺到可能的任務時，提出建議並等待確認。
- 不刪除任務檔案。不再需要的任務改為 `cancelled`，並在進度紀錄說明原因。
