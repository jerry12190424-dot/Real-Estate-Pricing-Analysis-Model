# 🏘️ Real Estate Alpha: Spatial Mispricing Prediction Model

## 📖 Project Vision (專案願景)
本專案為一個「不動產自動估價與超額價值尋找系統」的 Proof of Concept (POC)。
在量化研究的框架下，本模型不單純預測絕對價格，而是透過**區域中性化 (Cross-sectional Neutralization)** 處理，尋找在橫截面上開價異常偏低、具備潛在套利空間的「錯殺/被低估物件 (Underpriced Assets)」。

## 🧠 Core Methodology (核心量化邏輯)
* **資料工程與極端值處理:** 處理 2023-2026Q1 超過 144 萬筆開價數據。實作 Winsorization (首尾 1% 極端值截斷) 與防呆過濾機制，並處理空間座標 (x, y) 缺失值。
* **Alpha Target 建構:** 放棄直接預測絕對房價，改以 `實際開價 / 區域季均價` 作為預測目標 (Target)，強制模型學習物件在該行政區內的相對折溢價關係。
* **演算法選型 (LightGBM):** 採用基於決策樹的梯度提升框架。其原生的高效能節點分裂，能完美在二維地理座標 (Longitude, Latitude) 上進行正交切割，精準捕捉複雜的「地段空間溢價」等非線性特徵。
* **嚴謹的時間序列驗證 (Out-of-sample Split):** 摒棄傳統的隨機切分 (Random Split) 以避免 Data Leakage。採用嚴格的時間外推測試 (Train: 2023-2025, Test: 2026Q1)，最真實地模擬量化回測環境。

## 📊 Model Interpretability (模型解釋性)
專案內建 SHAP (SHapley Additive exPlanations) 事後解釋模組。從 SHAP Summary Plot 中可驗證模型的決策邏輯完全符合真實市場直覺（例如：車位屬性對「單價」產生精準的負向 SHAP 貢獻，捕捉了車位坪數稀釋單價的現象）。

---

## 🚀 Engineering Reflections & Future Roadmap (工程檢討與未來優化)
目前的 MVP 證明了此框架能有效尋找空間錯價。為符合真實量化交易的生產環境 (Production Environment) 標準，下一步優化將著重於以下三個維度：

### 1. 消除未來數據引用 (Look-ahead Bias Resolution)
* **現狀:** 目前 MVP 的 Target 採用「當季橫截面均價」進行中性化。
* **優化:** 實務上在 Live Inference 時，當季均價尚未發生。未來將改採 **Lagged Quarterly Mean (滯後一季均價)** 或 **Rolling 90-day Mean (滾動 90 日均價)** 作為分母，徹底消除 Look-ahead Bias，確保時間序列的絕對嚴謹。

### 2. 另類數據 (Alternative Data) 因子擴展
放棄單純依賴傳統房市開價特徵，計畫透過 API 整合豐富的地理空間資訊 (Spatial Data)，將「嫌惡設施/凶宅分佈距離」、「房屋採光與日照角度」量化為新的 **Alpha Factors**，捕捉傳統定價網站忽略的隱含價值。

### 3. 流動性風險控管 (Liquidity Risk Management)
在真實市場中，毫無流動性的資產即使被低估也無法變現。預測流程的最後一環將結合「熱銷物件過濾機制」，分析歷史週轉率並設定嚴格的流動性門檻，確保模型篩選出的潛在投資標的具備真實交易可行性。
