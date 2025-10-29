# News Summarizer TW — 新聞分類與摘要系統

**News Summarizer TW** 是一個面向研究者與工程師的開源工具包，涵蓋中文（繁體）新聞資料的清理、轉繁、統計分析、摘要生成與評估等完整流程。專案以「模組化、多階段」設計，便於在不同資料規模與模型條件下快速複用與擴充。

---

##  資料來源（Dataset）

本專案預設使用 **Kaggle: *Categorised News Dataset from Fudan University*** 作為示範語料來源。  
此資料集整理自復旦大學的中文新聞分類語料，涵蓋多個主題分類，適合做文本處理、摘要與分類等任務。

> Kaggle 連結（請至 Kaggle 下載並遵循其條款）：  
> https://www.kaggle.com/datasets  （搜尋關鍵字：*Categorised News Dataset from Fudan University*）

請將下載後的原始檔案放置於 `data/raw/` 目錄（詳見「資料夾規範」）。

---

##  核心特性

- **8 階段可組合 Pipeline**：從資料治理到模型訓練與評估，逐步清晰、易於維護。
- **繁體中文友善**：內建簡轉繁（S2T），適配臺灣/繁中新聞語料。
- **模型抽象層**：`models/llm_model.py` 統一摘要接口，可替換雲端 LLM 或本地模型。
- **可觀測性**：統計報表與評估結果輸出，利於實驗紀錄與比較。

---

## 🗂 專案結構（摘要）

```
news-summarizer-tw/
├─ models/
│  ├─ __init__.py
│  └─ llm_model.py                # 摘要/推論的統一接口（可擴充多實作）
├─ utils/
│  ├─ __init__.py
│  └─ filter_and_copy.py          # 篩選與複製檔案的小工具
├─ stage01_preprocess_folder_restructure.py
├─ stage02_text_convert_s2t.py
├─ stage03_text_cleaning.py
├─ stage04_data_statistics_analysis.py
├─ stage05_generate_summary.py
├─ stage06_summary_evaluation_bertscore.py
├─ stage07_dataset_preparation.py
├─ stage08_model_training.py
├─ requirements.txt
└─ README.md
```

> 註：檔名與既有腳本保留，方便讀者直接對照與延伸。

---

##  快速開始

### 1) 建立環境
```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

pip install -r requirements.txt
```

### 2) 準備資料
1. 從 Kaggle 下載 **Categorised News Dataset from Fudan University**。  
2. 將檔案解壓縮並放到：`data/raw/`。  
3. 若原始資料夾命名不同，請在各 stage 指令中以 `--src` 指定來源路徑。

### 3) 執行多階段 Pipeline（可分段執行）
```bash
# 01. 目錄重整 / 規範化
python stage01_preprocess_folder_restructure.py --src data/raw --dst data/01_restructured

# 02. 簡轉繁（Simplified → Traditional）
python stage02_text_convert_s2t.py --src data/01_restructured --dst data/02_trad

# 03. 文字清理（去雜訊/符號/空白/亂碼等）
python stage03_text_cleaning.py --src data/02_trad --dst data/03_clean

# 04. 數據統計與分析（字數分佈、長度統計、詞頻等）
python stage04_data_statistics_analysis.py --src data/03_clean --report out/reports/stats.json

# 05. 生成摘要（透過 models/llm_model.py 封裝的模型）
python stage05_generate_summary.py --src data/03_clean --dst data/05_summaries --model llm_default

# 06. 摘要品質評估（BERTScore）
python stage06_summary_evaluation_bertscore.py --src data/05_summaries --ref data/refs --out out/metrics/bertscore.csv

# 07. 建立訓練資料集（train/valid/test 切分）
python stage07_dataset_preparation.py --src data/03_clean --dst data/07_dataset --split 0.8 0.1 0.1

# 08. 模型訓練（可依需求替換演算法/框架）
python stage08_model_training.py --data data/07_dataset --out out/models/baseline
```

> 備註：若原始腳本尚未完全 argparse 化，可先依預設路徑放置資料；或在腳本內調整 I/O 參數。

---

##  模型層（`models/llm_model.py`）

- 提供 `summarize(text, max_len=...)` 等統一接口。
- 可新增：
  - **雲端 LLM**（OpenAI、Google 等）
  - **本地模型**（Hugging Face `transformers`、`llama.cpp` 等）
- 建議以環境變數/設定檔控制模型選擇與推論參數。

---

##  評估（Evaluation）

- 內建 **BERTScore**（Stage 06），可衡量系統摘要與參考摘要的一致性。
- 可擴充 ROUGE、BLEURT 或自定義的人評流程。

---

##  資料夾規範（建議）

```
data/
├─ raw/                  # Kaggle 下載的原始資料（唯讀）
├─ 01_restructured/      # 重整後
├─ 02_trad/              # 簡轉繁
├─ 03_clean/             # 清理後
├─ 05_summaries/         # 生成的摘要
├─ 07_dataset/           # 訓練資料（train/valid/test）
└─ refs/                 # 參考摘要（若有）

out/
├─ reports/
├─ metrics/
└─ models/
```

---

##  擴充與實務建議

- **長文摘要**：滑動視窗切片、語義分段（TextTiling）、或以檢索強化（RAG）導引摘要。
- **效能最佳化**：批次 I/O、快取、Parquet / Polars；模型端可考慮蒸餾、量化與剪枝。
- **重現性**：建議新增 `configs/`（YAML）管理實驗組態，並加入 CI/測試流程。

---

##  授權（License）

本專案採用 **MIT License**（見本庫 `LICENSE` 檔）。  
> 注意：**資料集本身的授權與使用條款**由其原始提供方（Kaggle/復旦大學語料）決定，請務必遵循其規範。

---

##  致謝（Acknowledgements）

- Kaggle: *Categorised News Dataset from Fudan University*（資料蒐集與整理）
- 開源社群在中文 NLP、生語言模型與評估指標（如 BERTScore、ROUGE）上的貢獻

---

