# rStack

給 Claude Code／Codex 用的 agent skill 合集。

| Skill | 用途 |
|---|---|
| [session-handoff](#session-handoff) | 把目前 session 整理成可驗證、可續作的交接文件 |

## 安裝

用 [skills.sh](https://skills.sh)：

```bash
bunx skills add godgiraffe/rStack            # 選擇要裝哪幾支
bunx skills add godgiraffe/rStack -s session-handoff   # 只裝一支
```

或手動複製 `skills/<name>/` 到 `~/.claude/skills/`（Claude Code）或 `~/.agents/skills/`（Codex）。

---

## session-handoff

把目前 session 整理成一份「續作索引」，讓下一個 session（可能是別的模型或 runtime）只讀這份檔案就能在五分鐘內安全接手。

- 先判斷需不需要 handoff，不需要就不產生檔案
- 寫之前實際查 git 狀態與驗證結果，沒跑的標 `NOT RUN`，不從對話敘述推定
- 新知識先分流（repo 可查的不記、長期知識進知識庫、進度才進 handoff）
- 不寫 secret、token、個資
- 支援 read／resume：用目前 repo 狀態驗證舊 handoff 是否還成立
- `restart`：寫完 handoff 直接開新 session 接手

### 使用

| 說法 | 行為 |
|---|---|
| `/session-handoff`、「收工」、「寫 handoff」 | 建立或更新 handoff |
| 「需要 handoff 嗎？」 | 只評估，不需要就不寫檔 |
| `read handoff <路徑>`、「接手」 | 讀 handoff 並用 repo 現況驗證 |
| `/session-handoff restart` | 寫完後開新 session 接手 |

handoff 預設寫到專案根的 `.handoffs/handoff-<YYYYMMDD>-<slug>.md`。

### 建議設定

**把 `.handoffs/` 加進全域 gitignore**，避免它出現在每個 repo 的 `git status`：

```bash
echo ".handoffs/" >> "$(git config --global core.excludesFile || echo ~/.gitignore_global)"
```

（還沒設過 `core.excludesFile` 的話，先 `git config --global core.excludesFile ~/.gitignore_global`。）

### 專案層覆寫

Claude Code 裡，同名的專案 skill 會被全域 skill 遮蔽而不載入。專案要加自己的交接規則，放在 `docs/context/session-handoff.md`，skill 會自動讀取並套用。

### 相容性

- `restart` 的「自動開新 session」只在 Claude 桌面 app 有效；終端機 CLI 與 Codex 會改印一行接手指令讓你貼。
- 內文以繁體中文撰寫，handoff 會依使用者與專案的語言產出。

---

## License

MIT
