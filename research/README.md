# 原始收集資料

這裡保存建立案例庫時收集到的原始資料，方便追溯每一筆紀錄的來源，或重新整理、重新訓練。整理後的正式資料在各鏈的 `<鏈>_hacks/data/incidents.json`。

## `solana/`：Solana 案例的調查原始輸出

每個檔案都是一輪網路調查的原始結果（JSON 陣列，欄位與 `incidents.json` 相同），包含後來被刪除或修正的紀錄：

| 檔案 | 內容 |
|---|---|
| `round1_2020-2022.json` | 第一輪：2020–2022 年事件 |
| `round1_2023-2024.json` | 第一輪：2023–2024 年事件 |
| `round1_2025.json` | 第一輪：2025 年事件 |
| `round1_2026.json` | 第一輪：2026 年事件 |
| `round1_crosscut.json` | 第一輪：L1 漏洞、釣魚 / 供應鏈、交易所熱錢包（跨年份） |
| `round2_expansion.json` | 第二輪擴充：停擺事件、重大漏洞揭露、惡意套件等 44 筆 |

合併時的整理規則（去重、刪除下游影響事件、重新分類）記錄在 git 歷史與 `solana_hacks/README.md`。

## `defihacklabs/`：DeFiHackLabs 匯入資料

由 `python -m hack_db.defihacklabs` 從 [DeFiHackLabs](https://github.com/SunWeb3Sec/DeFiHackLabs) 產生：

| 檔案 | 內容 |
|---|---|
| `entries.json` | 全部 871 起事件：日期、名稱、根因標籤、損失描述、PoC 路徑與網址、依 PoC 判斷的鏈別、分析連結、區塊鏈瀏覽器連結 |
| `ethereum_inputs.json` | 345 起以太坊事件，已解析美元損失與候選來源，作為紀錄撰寫 workflow 的輸入 |
| `bsc_inputs.json` | 371 起 BSC 事件，同上 |

PoC 原始碼沒有複製進來，需要時可以用 `poc_url` 查看，或用 `python -m hack_db.defihacklabs <目錄>` 重新下載。

## `evm_workflow_results/`：Ethereum / BSC 紀錄撰寫結果

`dhl_case_records` workflow 的原始輸出（每筆紀錄附審查者的判定與修正），用 `python -m hack_db.merge` 合併進 `ethereum_hacks` / `bsc_hacks`。
