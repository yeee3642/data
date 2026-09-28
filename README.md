# 區塊鏈駭客案例庫與流動池風險監控

先把一條鏈上曾經被駭的案例整理成**案例庫**並分類，再用案例庫歸納出過濾規則，最後**收集流動池資料**，過濾掉危險或沒有價值的池子、留下值得關注的。目前涵蓋 Solana、Ethereum、BNB Smart Chain（BSC）。

```
案例庫（*_hacks/）──歸納規則──► 資料收集與過濾（*_monitor/）──► 保留 / 丟棄＋理由（SQLite）
        ▲
原始收集資料（research/）
```

## 內容

| 目錄 | 說明 | 狀態 |
|---|---|---|
| [`solana_hacks/`](solana_hacks/README.md) | Solana 資安事件案例庫：SQLite、CSV、中文摘要、根因、漏洞模式與防禦建議 | ✅ 111 起 |
| `ethereum_hacks/` | 以太坊案例庫 | ⏳ 匯入 DeFiHackLabs 345 起中 |
| `bsc_hacks/` | BSC 案例庫 | ⏳ 匯入 DeFiHackLabs 371 起中 |
| `crypto_hacks.db` | 三條鏈合併的跨鏈資料庫，可依鏈、年份、攻擊類別比較 | ✅ 自動產生 |
| [`hack_db/`](hack_db/README.md) | 共用工具：資料結構、驗證、建置、合併研究結果、統計報告、DeFiHackLabs 匯入、研究 workflow 腳本 | ✅ |
| [`solana_monitor/`](solana_monitor/README.md) | Solana 新流動池收集與過濾（Raydium、Pump.fun、PumpSwap、Meteora、Orca） | ✅ |
| [`evm_monitor/`](evm_monitor/README.md) | Ethereum / BSC 新流動池收集與過濾（Uniswap、SushiSwap、PancakeSwap V2 / V3），合約安全檢查使用 GoPlus | ✅ |
| `monitor_common/` | 兩個監控工具共用的元件：DexScreener、AVE、規則框架、儲存 | ✅ |
| [`research/`](research/README.md) | 原始收集資料：每輪調查的原始輸出、DeFiHackLabs 匯入資料 | ✅ |

## 快速開始

只需要 Python 3.10 以上；核心功能都只用標準函式庫。

```bash
python -m hack_db                      # 建置所有案例庫與 crypto_hacks.db
python -m hack_db.report solana_hacks  # 產生統計報告
pytest tests/

# 離線示範：用範例資料跑一次過濾
python -m solana_monitor scan --pairs-file solana_monitor/examples/sample_pairs.json --no-rpc
python -m evm_monitor --chain bsc scan --pairs-file evm_monitor/examples/sample_pairs_bsc.json --no-security
```

選用套件：`websockets`（WebSocket 監聽）、`pycryptodome`（測試中的地址 checksum 驗證）。

## 注意

- 案例資料僅供資安研究與防禦用途，金額以事件發生時的美元估算，詳細來源與差異見每筆紀錄的 `sources` 與 `notes`。
- 過濾規則只能降低風險，不保證安全。
