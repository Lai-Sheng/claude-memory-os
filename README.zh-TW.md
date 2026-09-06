# claude-memory-os

**給 [Claude Code](https://claude.com/claude-code) 和
[Codex](https://developers.openai.com/codex) 用的檔案式記憶／專案作業系統。**

Claude Code 給了你 `CLAUDE.md` 和一個記憶資料夾，但沒告訴你該怎麼組織它們——
於是大多數人的結局都一樣：一個專案一個檔、只增不刪，直到每次對話都要載入兩萬個
token 的「三月已經修好的 bug」。

這個 repo 就是修正它的結構，抽取自一套連續運行半年、同時管十幾個專案的實戰設定。

[English → README.md](README.md)

---

## 只有一個核心想法

> **把「每次都必須載入的」和「按需才載入的」分開，並且用一個例行程序強制執行這條界線。**

其他所有設計都是從這句話長出來的。

```
┌──────────────────────────────────────────────────────────────┐
│ L1  CLAUDE.md            ~6 KB    永遠載入                    │
│     人設 · 路由選單 · 模式閘門 · 鐵則                          │
├──────────────────────────────────────────────────────────────┤
│ L2  MEMORY.md            <16 KB   永遠載入                    │
│     一檔一行索引 · 近期日誌 · 專案清單                         │
├──────────────────────────────────────────────────────────────┤
│ L3  projects/{p}/core.md          選定專案時載入               │
│     為何存在 · 不可違背的鐵則 · 設計憲法                       │
│     projects/{p}/state.md         按需載入（覆寫式）           │
│     進行中的工作 · 阻塞 · 下次從哪繼續                         │
├──────────────────────────────────────────────────────────────┤
│ L4  logs/YYYY-MM-DD.md            只有指名才載入               │
│     projects/{p}/archived/*.md    時序紀錄 ＋ 冷凍狀態         │
└──────────────────────────────────────────────────────────────┘
```

每一層都有**位元組預算**和**載入觸發條件**。超出預算就拆檔；被載入的頻率高過
它該有的觸發條件，就降級。

---

## 你實際會拿到什麼

### 1. 路由層，不是說明書

`CLAUDE.md` 只回答三個問題：我在跟誰講話、我們可以做哪些事、哪些規則不可違背。
其他一律不放。

最多人第一個抄走的是**路由選單**：每次新對話一開始，助理主動列出你的活躍專案，
你只要「指」就好。這是把「回想」換成「辨認」——也代表第 12 個專案跟第 1 個專案
一樣好取用。

### 2. core / state / archived 三層拆檔

| | `core.md` | `state.md` | `archived/` |
|---|---|---|---|
| **放什麼** | 為何存在、鐵則、設計憲法 | 當前工作、阻塞、續接點 | 歷史狀態快照 |
| **寫入方式** | 很少動，戰略轉向才改 | **每階段覆寫** | 新增帶日期的檔 |
| **何時載入** | 選定專案時 | 按需 | 只有指名 |
| **判斷標準** | 「一年後還成立嗎？」 | 「30 天後還重要嗎？」 | 「還能解釋什麼嗎？」 |

真正撐起整套系統的是**寫入方式**。`state.md` 是**覆寫、絕不追加**；舊狀態要嘛
移進 `archived/`，要嘛直接刪掉——日誌已經記過了，存兩份才是 bug，不是保險。

### 3. `/save-progress`——強制執行迴圈

靠自律維持的架構一定會腐化。這套用一個「關終端機前跑一次」的指令來強制執行：

- **Step 0**：寫入前先量目標檔。超標 → 提案拆檔，讓你自己決定現在拆還是延後
- **Step 0.5**：**每次都量索引**。超過 16 KB → 跨月歸檔、每條砍成一行
- **Step 1–5**：覆寫 `state.md`、寫當日日誌、更新索引，最後告訴你下次從哪繼續

Step 0 之所以存在，是因為**這個指令的第一版正是它現在在防的那場膨脹的元凶**。
只追加的存檔程序每次多 300 bytes：兩個月內完全看不出來，然後它就吃掉你一半的
上下文視窗。

### 4. Feedback 檔案——讓糾正活得比對話久

當你糾正的是助理「怎麼做事」而不是「做出什麼」，那條糾正就該變成一個小小的、
永遠載入的檔案：規則 ＋ 背後的事件 ＋ 怎麼套用。

那個 **Why** 不是裝飾。有記錄理由的規則，日後情況改變時可以重新評估；沒有理由的
規則會變成 cargo cult，最後被忽略。

### 5. 分區（Zoning）——專案大到一個檔裝不下時

當一個「專案」其實是好幾份工作共用一個名字，就把它拆成各自單一職責的分區，
並遵守三條規則：**事實下沉**到共用目錄、**狀態歸屬看誰在推進**、
**跨區只放連結永不複製**。

詳見 [docs/srp-zoning.md](docs/srp-zoning.md)。

---

## 安裝

```bash
git clone https://github.com/Lai-Sheng/claude-memory-os.git
cd claude-memory-os
```

### 第 1 步——複製檔案

記憶**放哪都可以**——路徑是在路由檔裡宣告的，沒有什麼神祕目錄要你去找。
以下指令用 `~/AgentMemory`。

**macOS / Linux — Claude Code**

```bash
mkdir -p ~/AgentMemory ~/.claude/commands
cp template/CLAUDE.md     ~/.claude/CLAUDE.md
cp template/commands/*.md ~/.claude/commands/
cp -r template/memory/.   ~/AgentMemory/
```

**macOS / Linux — Codex**

```bash
mkdir -p ~/AgentMemory ~/.codex/commands
cp template/AGENTS.md     ~/.codex/AGENTS.md
cp template/commands/*.md ~/.codex/commands/
cp -r template/memory/.   ~/AgentMemory/
```

**Windows PowerShell — Claude Code**

```powershell
New-Item -ItemType Directory -Force ~\AgentMemory, ~\.claude\commands | Out-Null
Copy-Item template\CLAUDE.md ~\.claude\CLAUDE.md
Copy-Item template\commands\*.md ~\.claude\commands\
Copy-Item -Recurse -Force template\memory\* ~\AgentMemory\
```

**Windows PowerShell — Codex**

```powershell
New-Item -ItemType Directory -Force ~\AgentMemory, ~\.codex\commands | Out-Null
Copy-Item template\AGENTS.md ~\.codex\AGENTS.md
Copy-Item template\commands\*.md ~\.codex\commands\
Copy-Item -Recurse -Force template\memory\* ~\AgentMemory\
```

⚠️ 已經有 `CLAUDE.md`／`AGENTS.md` 了？請**合併**，不要直接覆蓋。

### 第 2 步——告訴路由層記憶在哪 🚨

打開 `~/.claude/CLAUDE.md`（或 `~/.codex/AGENTS.md`），把 **§0** 裡的
`{{MEMORY_ROOT}}` 換成你剛才用的絕對路徑。`~/AgentMemory/README.md` 也要改。

**§0 是記憶層會不會被載入的關鍵，不要刪掉它。** 少了它，agent 會只憑路由選單
回答、安靜地跳過記憶——看起來一切正常，直到它推翻你上週記錄過的決定。

### 第 3 步——填完其餘欄位，然後自檢

填好人設、路由選單、`user_profile.md`，然後確認沒有漏填的：

```bash
grep -rn "{{" ~/.claude/CLAUDE.md ~/AgentMemory/
```

```powershell
Select-String -Pattern "{{" -Path ~\.claude\CLAUDE.md, ~\AgentMemory\* -Recurse
```

**印出來的每一行都是你還沒填的佔位符。** 印不出東西以後，從
`template/memory/projects/_project_template/` 複製出第一個專案，下次工作結束前
跑一次 `/save-progress`。

完整步驟：**[docs/getting-started.md](docs/getting-started.md)**、
Codex 專屬：**[docs/codex.md](docs/codex.md)**

> **想用 Claude Code 原生的記憶目錄？** 也可以把 `{{MEMORY_ROOT}}` 指向
> `~/.claude/projects/<workspace>/memory/`，用 `ls -d ~/.claude/projects/*/`
> 找出你的那個。這不是必要的——自己選的路徑比較好備份，也比較好搬到另一台機器。

---

## 文件

| 文件 | 內容 |
|---|---|
| [getting-started.md](docs/getting-started.md) | 15 分鐘上手、健檢指令、常見錯誤 |
| [architecture.md](docs/architecture.md) | 四層架構，以及每條界線為什麼存在 |
| [srp-zoning.md](docs/srp-zoning.md) | 專案長大到 core/state 裝不下時怎麼拆 |
| [codex.md](docs/codex.md) | Codex 安裝、記憶載入的關鍵差異、雙 agent 並存 |

---

## 設計原則

1. **載入是成本**：每一個永遠載入的位元組，你每則訊息都要付一次。
2. **要預算，不要決心**：沒有人在量的限制只是願望。
3. **覆寫優於追加**：只追加的結構沒有天然的大小上限。
4. **歷史和狀態是不同的檔案**：日誌回答「何時」，專案檔回答「現在」，不要兩邊都寫。
5. **辨認優於回想**：主動遞選單，永遠別要求使用者自己記得有哪些選項。
6. **規則要帶著理由走**：沒記錄 why 的規則，不是被盲從就是被忽略。
7. **冷凍優於刪除**：做完的專案該停止被載入，不是停止存在。

---

## 這適合你嗎？

**大概適合**：你同時跑好幾個長期專案，而且已經注意到助理的回答會飄向上個月的話題。

**大概不適合**：你一次只開一個 repo、每次都從零開始——原生的 `CLAUDE.md` 就夠了，
這套只會變成負擔。

**Claude Code 和 Codex 都能跑**，而且共用同一套記憶樹——見
[docs/codex.md](docs/codex.md)。這套分層適用於任何「把檔案載入上下文視窗」的
agent，只有路由檔和指令目錄是平台專屬的。

### ⚠️ Codex 的關鍵差異

**Claude Code 會自動載入記憶索引，Codex 不會。** Codex 只自動吃 `AGENTS.md`，
所以 L2 索引層必須靠指令去抓，這帶來兩個額外要求：

1. **`AGENTS.md` §0 記憶啟動段**：宣告記憶根目錄，並命令 agent 在回答任何專案
   問題前先讀 `MEMORY.md`
2. **`memory/README.md`**：宣告該目錄是唯一真相源，讓歸屬權存在於 agent 上下文之外

兩份都在 `template/` 裡。**別刪掉 §0**——少了它，agent 會只憑路由選單回答、
安靜地跳過整個記憶層。這種失效看起來一切正常，直到它推翻你上週記錄過的決定。

> repo 名稱維持原樣是為了網址穩定，它不是 Claude 專用的。

---

## 授權

MIT，見 [LICENSE](LICENSE)。拿去用、fork、改名都可以。只要它幫你省下一個上下文
視窗，這個 repo 就算值了。
