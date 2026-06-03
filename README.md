# 🏢 Real-Estate-Pricing-Analysis-Model

> A Quantitative Approach to Spatial Mispricing and Alpha Generation in Real Estate Markets.

## 1. 📌 Project Vision (專案願景)
本專案為一個「不動產自動估價與超額價值尋找系統」的 Proof of Concept (POC)。
在量化研究的框架下，本系統不旨在預測絕對價格，而是透過**區域中性化 (Cross-sectional Neutralization)** 處理，尋找在空間橫截面上開價異常偏低、具備潛在套利空間的「錯殺/被低估物件 (Underpriced Assets)」。

---

## 2. ⚙️ Core Methodology (核心量化邏輯)

### 2.1 資料工程與防呆過濾 (Data Engineering)
* **Dataset:** 處理 2023-2026Q1 超過 144 萬筆開價數據。
* **Winsorization:** 實作首尾 1% 極端值截斷，排除離群值雜訊。
* **Imputation:** 針對空間座標 $(x, y)$ 缺失值，建立 `groupby(['dist', 'road'])` 的階層式均值填補機制。

### 2.2 Alpha Target 建構 (Target Formulation)
放棄直接預測絕對房價，改以建立相對折溢價指標 (Relative Premium/Discount)。
強制模型學習物件在該行政區內的相對價值關係，數學抽象化為：
`Target = Actual_Unit_Price / E[Unit_Price | District, Quarter]`

### 2.3 演算法架構 (Algorithm Selection)
採用基於決策樹的梯度提升框架 (**LightGBM**)。其原生的高效能節點分裂 (Node Splitting)，能有效處理二維地理座標 (Longitude, Latitude) 上的正交切割，顯著捕捉地段空間溢價等高度非線性 (Non-linear) 特徵。

### 2.4 時間序列外推驗證 (Out-of-sample Validation)
摒棄傳統機器學習的隨機切分 (Random Split) 以絕對避免 Data Leakage。採用嚴格的**時間外推切割 (Temporal Split)**：
* `Train/Val Set`: 2023-2025 (Out-of-bag validation for Early Stopping)
* `Test Set`: 2026Q1 (真實模擬量化回測與未來推論情境)

---

## 3. 📈 Model Interpretability (模型解釋性)
專案內建 **SHAP (SHapley Additive exPlanations)** 事後解釋模組。從 SHAP Summary Plot 的特徵歸因分析中，特徵歸因結果與實務市場定價邏輯具備高度一致性：

1. **車位坪數稀釋效應 (Dilution Effect):** 車位屬性對「單價 Target」產生穩定的負向 SHAP 貢獻，模型成功學習到車位坪數會拉低整體平均單價的數學關係。
2. **老屋都更潛力溢價 (Urban Renewal Premium):** 模型發現在「屋齡 (Age)」與「價值」之間存在非線性 U 型關係。在特定高價值地段，極高屋齡的老屋反而獲得了正向的 SHAP 貢獻，證明模型自主挖掘出了潛在的改建與都市更新 (都更) 價值，而非單純的線性折舊。

---

## 4. 🧭 Future Engineering Roadmap (未來優化藍圖)
目前的 MVP 證明了此機器學習框架能有效在空間橫截面上尋找錯價 (Mispricing)。為符合真實量化交易與精準估價的生產環境 (Production Environment) 標準，下一階段優化將著重於以下五個工程維度：

### 4.1 流動性風險與去化門檻 (Liquidity & Turnover Constraints)
在真實市場中，毫無流動性的資產即使被嚴重低估也無法變現。預測流程的最後一環將結合「熱銷物件過濾機制」，分析歷史週轉率 (Historical Turnover Rate) 並設定嚴格的**流動性門檻 (Liquidity Filter)**，萃取出兼具「價格 Alpha」與「高流動性」的真實可投資標的 (Actionable Investment Universe)。

### 4.2 消除未來數據引用 (Look-ahead Bias Mitigation)
* **Lagged Neutralization:** 目前 MVP 採用「當季均價」進行中性化。實務 Live Inference 時，將改採 **上一季的區域均價 (Lagged Quarterly Mean)** 作為分母基準，徹底消除 Look-ahead Bias。
* **Listing Deduplication:** 建立物件追蹤碼 (Property ID tracking) 邏輯，過濾重複上架/降價的同一物件，確保時間序列樣本的獨立性。

### 4.3 另類地理與環境數據 (Alternative Spatial Data)
透過 API 整合更細緻的外部地理空間矩陣 (Spatial Matrices)。例如：量化「大眾運輸樞紐距離」、「採光與日照幾何角度」，以及「嫌惡設施/凶宅分佈範圍」作為新的 **Alpha Factors**，捕捉傳統定價網站忽略的隱含折溢價。

### 4.4 跨市場資金動能 (Cross-Market Capital Flows)
導入跨市場資金動能因子（如：台股大盤指數波動率、M2 貨幣供給、股市與房地產資金流向比例等），從總體經濟 (Macro) 維度捕捉市場的真實熱度與買盤溢出效應 (Spillover Effect)。

### 4.5 總體政策與法規因子 (Macro Policy Factors)
將央行信用管制 (如 LTV Limits 貸款成數上限)、稅制變動 (如房地合一稅) 及利率決策等宏觀變數進行量化編碼 (Quantitative Encoding)，提升模型在面對市場結構性轉折 (Structural Breaks) 時的預測韌性。
