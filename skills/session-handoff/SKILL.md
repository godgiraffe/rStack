---
name: session-handoff
description: "Create, assess, or resume a repository-aware session handoff. Decide whether a checkpoint is needed, discover the project's handoff and knowledge conventions, verify repository and validation state, route durable knowledge, redact sensitive data, and write a concise continuation file with exact next steps. Use when the user says /session-handoff, $session-handoff, 收工, 寫 handoff, 建立接手文件, 總結這個 session, restart／換新對話（寫完直接開新 session 接手）, context 快滿, read/resume handoff, or asks whether a handoff is needed."
---

# Session Handoff

下個 session 可能是較弱的模型、也可能是別的 runtime，只靠 handoff 加專案索引接手。**寫壞＝重查一輪、寫漏＝重踩一雷、寫多＝索引淹沒。**

把 handoff 當「續作索引」，不是對話流水帳。依使用者與專案的主要語言撰寫。目標：接手者只讀這一份與其引用的穩定資料，就能在五分鐘內安全動工。

## 0. 先判斷:建立、評估、還是接手

- **明確要求寫 handoff／checkpoint** → 建立或更新。
- **要求 restart**（`/session-handoff restart`、「換新對話」）→ 建立或更新，交付後走 §8 開新 session 接手。
- **只問「需要嗎」** → 先驗證現況。工作已完成、狀態可由既有 artifact 重建、工作樹無殘留、下一步也不依賴本次推理 → **直接說不需要，不製造檔案**。
- **未完成／context 將滿／等外部輸入／有未提交變更／下一步依賴本次推理** → 建立 checkpoint。
- **要求 read／resume** → 找到真正相關的 checkpoint，再以目前 repo 狀態驗證它。**repo 與外部系統的現況永遠優先於 handoff 敘述。**
  - 有多份 handoff、使用者又沒點名主題 → 只全讀最近更新的那份；其餘只讀 `Status` 與「一句話」兩段，列給使用者挑。別份等點名才全讀。
  - 平台 memory 只開標 ⏳／⏭ 的條目；⏸ 是在等人或外部事件，主題不相關就別開。
  - 回報給 operator 時，每個票號、代號、內部詞（`#225`、`A2`、`c75`、arm、偏差自動併入）第一次出現就先寫一句白話：它是什麼、為什麼現在要做、現在能不能用，代號放括號。handoff 與 memory 是 agent 的速記，照抄給 operator 等於叫他去查 issue。

### 選完分支後的執行契約

**所有模式共用**：先走 §1（慣例）、§4（驗證）；分支工作完成後再走 §6（清理）、§7（自檢）與最後回報行。**每個節點要嘛完成、要嘛明列 `N/A｜NOT RUN｜BLOCKED`——不得因為判定「不需要 handoff」就提前結束。** §6 涉及刪檔時，清理條件成立**且**本次授權允許修改才執行，否則列出候選交 operator 決定。

**分支工作**：建立／更新 ＝ §2 → §3 → §5；restart ＝ 建立／更新，§7 自檢與最後回報行之後再走 §8；評估 ＝ 不建立檔案（路徑與追蹤／忽略狀態回報 `N/A`）；read／resume ＝ 驗證 checkpoint 是否仍成立。

## 1. 發現專案慣例(別假設)

1. 版本控制根目錄為專案根；非版控專案用目前工作目錄。
2. 使用者指定的路徑／是否跨機分享／是否進版控，優先於以下任何預設。
3. 讀專案 router 與 repo-local 覆寫：`AGENTS.md`、`CLAUDE.md`、`.codex/`、覆寫層 `docs/context/session-handoff.md`（若存在）。**只載入與交接、知識路由、驗證、敏感資料直接相關的規則。**
4. 找出既有 `.handoffs/`／`handoffs/` 與最近的同類檔。兩個目錄並存時，不要只看目錄名或 mtime；用明文規則、`git check-ignore`、`git ls-files`、內容引用與目前 branch/HEAD 判定用途。
5. 輸出位置優先序：使用者明示 → repo 明文規定 → 同類既有慣例 → `.handoffs/handoff-<YYYYMMDD>-<slug>.md` → （僅當專案根不可寫）OS 暫存目錄並警告它會被清掉。
6. 預設建**本機** checkpoint：不 stage、不 commit、不改 `.gitignore`。只有使用者明示或 repo workflow 明定，才當可追蹤的長期 artifact。寫檔後若 `git check-ignore` 顯示 handoff 沒被忽略，回報時提醒 operator 可把 `.handoffs/` 加進全域 gitignore（`git config --global core.excludesFile`）。

⚠ **同名的 repo-local skill 會被全域版遮蔽、不會載入**；專案自己的交接規則放覆寫層 `docs/context/session-handoff.md`，依它標的取代／補充套用——但仍保留本檔的驗真、敏感資料與不擴張授權原則。

## 2. 知識歸位(每條新知先分流,別全塞進 handoff)

| 這條知識 | 去哪 |
|---|---|
| repo 現況可查的（code／測試／commit 內容） | **不記**。git log 就是紀錄，只留 locator 與必要的「為什麼」 |
| 長期領域事實、API schema、新踩的雷 | 專案指定的知識庫（如 `docs/context/`、ADR、runbook、issue）。同主題速記累積 3–5 條就整理搬過去，索引只留一行連結 |
| 進度／狀態／停點／阻塞／下一步／defer | handoff 本身 |
| 對方對「你該怎麼工作」的糾正 | 獨立的長期記錄，**含 Why 與 How to apply**（只有原文沒有理由，下次還是會犯） |

- 本次授權若不含修改知識庫 → 寫進 handoff 的「待沉澱知識」並**指名建議目的地**。
- **已在穩定 artifact 的內容不要雙寫**；易消失的暫存檔、本機報告不得成為唯一依據。

## 3. 平台 memory / 索引

- **先寫 `.handoffs/`**。memory 是否可讀寫，以本次 system policy 與使用者明確授權為準；無權限時由 operator 或有權限的後續 session 搬運，**禁止繞過**。
- 有權更新時：找同主題既有條目**原地改**，不新增重複；狀態標記統一（✅結案／⏭下一步／⏸等人或外部事件／⚠陷阱／⏳待驗）。卡在別人身上的事（等 operator 決定、等出差回來、needs-triage）一律標 ⏸、不標 ⏭，因為接手流程會打開每個 ⏭ 條目；索引行只寫 git 查不到的（決策、在等誰、觸發條件）：push、merge、worktree 狀態接手時用 git 或專案狀態腳本查，不寫進 memory；commit hash、部署版本這類 fixed point 在 topic 檔記一次；**相對日期一律轉絕對**（「今天」→ `2026-07-26`）；細節裝不下就開 handoff、條目尾掛連結。
- **一條＝一行、150 字以內**；索引要能 30 秒掃完，已結案且無 ⏭ 的條目考慮縮短或併入相鄰條目。

## 4. 驗證 current truth(不從敘述推定)

撰寫前直接查可查證的狀態（專案根手冊列了一鍵狀態腳本就先跑它，例如 `bun scripts/state.ts`、`make status`，再補它沒涵蓋的項目）：

- branch、短或完整 HEAD、base/upstream 差異（**實際驗過才寫**）、工作樹狀態、相關未提交檔案逐檔列。
- 把使用者原有／無關變更與本次工作**分開**；無法證明歸屬就標「未知」，不要猜。
- 列本次相關的 commit、issue、ADR、plan、report 等 fixed point，引路徑或 URL，不重抄全文。
- 每項驗證寫明 **exact command**、`PASS｜FAIL｜NOT RUN｜BLOCKED`、對應 revision、必要警告。
  ⚠ **未跑就是 `NOT RUN`。不要為了填滿表格宣稱成功。**
- 跑過的品質關卡要記結果與輪次，不只記「過了」：測試總數與 skip 數、mutation 是否**真的轉紅**、獨立審查跑了幾輪與 journal 驗真結果（0 finding 可能是 agent 掛掉）、真機驗收跑到哪一階。**跑過但沒記，比沒跑更危險**——接手者會當它已驗。
- 不要只為寫 handoff 就重跑昂貴或有外部副作用的驗證；repo workflow 明定必跑者除外。
- 非 Git 專案改記可驗證的版本、產物 revision、檔案狀態或外部工作項目 ID。

## 5. 骨架

沿用 repo 既有格式；無慣例時用這個。省略空的選配段。

```markdown
# Session handoff — <YYYY-MM-DD> — <topic>

Status: IN_PROGRESS | WAITING | BLOCKED | COMPLETE
Captured at: <絕對時間 + 時區>
Checkpoint: <branch + HEAD ／ 版本 ／ artifact revision>

## 一句話
<完成到哪、停在哪、真正剩下什麼。白話、不放 hash 與 identifier,operator 不看其他段也能懂現況;locator 放下面各段>

## 背景與現況
- Objective:
- Working state: <branch/HEAD、領先 main N、工作樹>
- 未提交／未發布變更: <逐檔>
- 既有無關變更(要保留的):
- 需要的使用者權限或外部輸入:

## 本次做了什麼
- <改了什麼 + 為什麼;引 commit/file/issue,不重述全文>

## 驗證狀態
| Check | Result | Evidence / revision |
|---|---|---|
| `<exact command>` | PASS / FAIL / NOT RUN / BLOCKED | <結果、警告、revision> |

## ⏭ 接手第一個動作
- [SAFE | REQUIRES USER] <具體到命令／腳本／要開哪張卡>

## 剩餘步驟
1. ...

## defer 清單
- <item> — Trigger: <什麼條件才重新考慮>

## ⚠ 接手要知道的雷
- <本次新發現、還沒進知識庫的>

## 建議 skills
- `<確認存在且下一步用得到的>` — <用途>

## 參照
- <確認存在且穩定的 path / URL>
```

### 硬性限制

- `Status: COMPLETE` 時**不要把另一件新工作偽裝成剩餘步驟**；沒殘留就省略該段或寫 `None`。
- 下一步涉及金錢、真人憑證、GUI、外部發布、訊息傳送或任何新增權限 → 標 `REQUIRES USER`；**第一個動作優先寫安全的唯讀準備**。
- defer 每條附觸發條件，不寫裸的「以後再說」。
- 建議 skills 前**先確認已安裝**；不要推薦不存在的命令或 skill。
- **不寫 secret、token、password、signature、個資、真實帳戶餘額／持倉數字**或其他敏感營運資料。保留變數名稱與遮罩後形態即可。**這條沒有例外，包括「只是本機暫存檔」。**
- 更新同主題 handoff 時**只保留一個 current truth**：移除過時說法，或明確放進 `Superseded` 段，避免上面寫完成、下面還寫未完成。

## 6. 清理(常態只留最新一棒)

- 寫**結案**（merged／complete）handoff 時，同主題被取代的中繼檔（wip／code-done／review 環節）**一併刪**。
- 其他已結案舊檔：確認獨有知識已歸位（知識庫／ADR／memory）後即刪。
- **被外部索引連結不是保留理由**——handoff 不是長期知識的歸宿。有權讀該索引時：找出引用、把連結改成落地備註，然後刪。
- 無法確認外部索引（如另一個 runtime 的 memory）是否連到某舊檔時，**只清同主題鏈**，全域清理留給讀得到該索引的 session。

## 7. 交付前自檢

- 另一位接手者只讀本檔＋參照，能否五分鐘內動工？不能 → 缺的就是該補的。
- branch、HEAD、dirty files、驗證結果、路徑，是否**全部來自本次查證**？
- 第一個動作是否具體、安全、沒有暗中擴張授權？
- 本次變更／既有無關變更／未知歸屬，是否清楚分開？
- 敏感值、重複內容、過時矛盾、取不到的唯一引用，是否都清掉？
- 寫檔後**再查一次版控狀態**，確認沒有意外 stage、commit 或動到無關檔案。
- 回報給 operator 的每個代號（票號、片名、branch、選項字母）第一次出現時，都有 §0 的白話與「現在能不能用」嗎？

最後回報：是否需要 handoff、實際路徑、追蹤／忽略狀態、接手第一個動作。判定不需要時，直接說明可由哪些已驗證 artifact 重建狀態。

## 8. restart（只在「要求 restart」分支）

handoff 已寫檔且過 §7 自檢才開；寫檔失敗就停在這裡回報，不開空的新 session。

- **有開 session 的工具**（Claude 桌面 app：`mcp__ccd_session__start_session`）：`initiation: "user_asked"`、`context: "fresh"`（fork 會把舊對話整段帶過去，失去換 session 的意義）、`use_worktree: false`、model／effort 不帶（沿用）。
  - `title`：`接手：<topic>`
  - `prompt`：`read handoff <handoff 絕對路徑>`
  - `background`：上一個 session 已寫好這份 handoff；照 session-handoff 的 read／resume 分支接手——讀它、開 memory 的 ⏳／⏭ 條目、用 git 驗現況，再回報可開工與可並行的事項。
  - 開好立刻 `mcp__ccd_session_mgmt__detach_session`（`session_id`＝新 session 的 id），讓它成為同專案的頂層 session：不掛在舊 session 底下、不會跟著舊 session 封存、舊 session 也不再收到它的回報。被拒時（例如舊 session 在 worktree 裡、新 session 共用同一個 worktree）照實回報，它就留在子 session。
  - 再放進舊 session 的側欄群組：`mcp__ccd_session_mgmt__get_session`（`"self"`）讀 `group`；有值就 `mcp__ccd_sidebar__move_sessions`（`session_ids`＝[新 session 的 id]、`group_id`＝該 `group.id`），`null` 就跳過。不做這步新 session 會落在未分組的「最近項目」，因為 operator 的專案分類是手動建的自訂群組、start_session 不會繼承。預設權限模式下這步會跳確認卡。
  - 回報新 session 的連結；舊 session 要不要封存問 operator 一句，答應才封存。
- **沒有這種工具**（終端機 CLI、Codex）：最後回報加一行可直接複製的接手指令——Claude：先 `/clear`，再貼 `read handoff <絕對路徑>`；Codex：開新對話貼同一句。
