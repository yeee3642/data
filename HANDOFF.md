# 交接文件（截至 2026-09-28）

## 一、目標與進度

| 步驟 | 內容 | 狀態 |
|---|---|---|
| 1 | 選定主要鏈：Solana，並擴充 Ethereum、BSC | ✅ |
| 2 | 建立案例庫：收集被駭案例、分類、歸納邏輯漏洞模式 | Solana ✅；Ethereum / BSC 🟡 初稿 |
| 3 | 收集流動池資料並過濾（DexScreener、AVE、GMGN、自行監聽鏈上事件） | ✅ 程式完成；需開放網路才能跑即時資料 |

## 二、案例庫現況

| 案例庫 | 筆數 | 損失合計 | 狀態 |
|---|---:|---:|---|
| `solana_hacks/` | 111 | 約 18.1 億美元（只影響 Solana 的 93 起約 10.3 億） | ✅ 已經過「事實」與「範圍 / 分類」兩個角度交叉驗證 |
| `ethereum_hacks/` | 236 | 約 33 億美元（最大：Bybit 15 億、Ronin 6.24 億） | 🟡 初稿：由 DeFiHackLabs PoC 撰寫，**尚未獨立審查**，每筆 `notes` 都有標註 |
| `bsc_hacks/` | 266 | 約 1.88 億美元（最大：Uranium 5,000 萬） | 🟡 初稿，同上 |
| `crypto_hacks.db` | 613 | — | ✅ 三條鏈合併，主鍵 `(chain, id)` |

Solana 的邏輯漏洞分析結論：帳戶驗證缺失（account validation）占邏輯漏洞損失約 97%，是過濾規則的首要依據。

## 三、未完成事項（依優先順序）

1. **審查 Ethereum / BSC 初稿**（502 筆）。用 `hack_db/workflows/dhl_case_records.js` 的審查階段逐筆重讀 PoC 嘗試推翻，再用 `python -m hack_db.merge`（**不加** `--draft`）合併，取代初稿。
2. **補寫剩下的 DeFiHackLabs 事件**：以太坊 105 起、BSC 101 起（例如 Euler），清單在 `research/evm_workflow_results/*_remaining_inputs.json`。
3. **補非 DeFiHackLabs 事件**：交易所熱錢包、釣魚、BNB Chain 跨鏈橋等。需要大量網路搜尋，見第五節。
4. **監控工具接即時資料**：需要開放網路，見第五節。

## 四、接續步驟

```bash
git clone https://github.com/yeee3642/data && cd data
python -m hack_db            # 建置所有資料庫與 CSV
pytest tests/                # 目前 64 項全部通過

# 1. 重新下載 DeFiHackLabs PoC（remaining_inputs 只有 poc_url，workflow 需要本機 poc_path）
python -m hack_db.defihacklabs /tmp/dhl
#    產生 /tmp/dhl/ethereum_inputs.json、bsc_inputs.json（含 poc_path）

# 2. 在 Claude Code 中請 Claude 用 hack_db/workflows/dhl_case_records.js 撰寫＋審查，參數範例：
#    {"dataset": "ethereum_hacks", "chainName": "Ethereum", "singleScope": "ethereum_only",
#     "inputPath": "/tmp/dhl/ethereum_inputs.json", "count": 345, "part": "eth"}
#    只補剩下的事件時，把 inputPath 指向篩選過的清單（對照 *_remaining_inputs.json 的日期與名稱）。

# 3. 合併結果
python -m hack_db.merge ethereum_hacks result.json --dry-run   # 先檢查
python -m hack_db.merge ethereum_hacks result.json             # 經審查的結果
python -m hack_db.merge ethereum_hacks drafts.json --draft     # 未審查的初稿（會在 notes 標註）
python -m hack_db
python -m hack_db.report ethereum_hacks > /tmp/stats.md        # 更新 README 統計
```

注意：Workflow 每次最多同時跑「CPU 數 − 2」個 agent（4 核心機器只有 2 個），大量工作請拆成多個 workflow 並行。

## 五、環境設定（雲端環境設定頁面）

- **網路搜尋額度**：Claude Code 預設每個 session 只有 200 次搜尋，這次已用完。請加環境變數 `CLAUDE_CODE_MAX_WEB_SEARCHES_PER_SESSION=5000`，再開新 session。
- **網路白名單**（監控工具與研究需要）：
  - `api.dexscreener.com`
  - `api.mainnet-beta.solana.com`（或你的 Solana RPC，例如 Helius）
  - `data.ave-api.xyz`（AVE，需要 `X-API-KEY`）
  - `api.gopluslabs.io`
  - BSC / Ethereum RPC（例如 `bsc-dataseed.bnbchain.org`、你的 Infura / Alchemy）
  - `raw.githubusercontent.com`（DeFiHackLabs）
- **GMGN**：官方 OpenAPI 要到 gmgn.ai/ai 註冊 Ed25519 公鑰，只能透過 `gmgn-cli` 使用，沒有公開 REST 文件，所以目前沒有接。

## 六、監控工具快速使用

```bash
# Solana：Raydium、Pump.fun、PumpSwap、Meteora、Orca
python -m solana_monitor rules                 # 列出所有過濾規則與理由
python -m solana_monitor scan                  # DexScreener 新池子 → 過濾 → 存 SQLite
python -m solana_monitor listen                # WebSocket 監聽建池交易

# BSC / Ethereum：PancakeSwap V2 / V3、Uniswap V2 / V3、SushiSwap
python -m evm_monitor --chain bsc rules
python -m evm_monitor --chain bsc listen       # 監聽 PairCreated / PoolCreated 事件
python -m evm_monitor --chain bsc scan
python -m evm_monitor --chain bsc report
```

過濾分四層：上線條件 → 市場數據 → 合約安全（Solana 看 mint / freeze authority 與 Token-2022 擴充；EVM 看 GoPlus 的蜜罐、稅率、owner 權限）→ 案例庫比對（協議近期是否被駭）。每個池子都存下「保留 / 丟棄」與命中的規則理由。

## 七、其他

- 這個 repo 是**私人** repo，commit 作者都是 yeee3642（GitHub noreply 信箱）。
- `yeee3642/OpenMythos` 上的 `claude/focused-goldberg-lncke2` 分支自動刪除失敗，請到 https://github.com/yeee3642/OpenMythos/branches 手動刪除。
- 資料僅供資安研究與防禦用途；過濾規則只能降低風險，不保證安全。
