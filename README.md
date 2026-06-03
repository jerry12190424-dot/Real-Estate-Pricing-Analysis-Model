# 🏢 Real-Estate-Pricing-Analysis-Model

## 📌 Project Vision (專案願景)
本專案為一個「不動產自動估價與超額價值尋找系統」的 Proof of Concept (POC)。
在量化研究的框架下，本模型不單純預測絕對價格，而是透過**區域中性化 (Cross-sectional Neutralization)** 處理，尋找在橫截面上開價異常偏低、具備潛在套利空間的「錯殺/被低估物件 (Underpriced Assets)」。

## ⚙️ Core Methodology (核心量化邏輯)
* **資料工程與極端值處理:** 處理 2023-2026Q1 超過 144 萬筆開價數據。實作 Winsorization (首尾 1% 極端值截斷) 與防呆過濾機制，並處理空間座標 (x, y) 缺失值。
* **Alpha Target 建構:** 放棄直接預測絕對房價，改以 `實際開價 / 區域季均價` 作為預測目標 (Target)，強制模型學習物件在該行政區內的相對折溢價關係。
* **演算法選型 (LightGBM):** 採用基於決策樹的梯度提升框架。其原生的高效能節點分裂，能完美在二維地理座標 (Longitude, Latitude) 上進行正交切割，精準捕捉複雜的「地段空間溢價」等非線性特徵。
* **嚴謹的時間序列驗證 (Out-of-sample Split):** 摒棄傳統的隨機切分 (Random Split) 以避免 Data Leakage。採用嚴格的時間外推測試 (Train: 2023-2025, Test: 2026Q1)，最真實地模擬量化回測環境。

## 📈 Model Interpretability (模型解釋性)
專案內建 SHAP (SHapley Additive exPlanations) 事後解釋模組。從 SHAP Summary Plot 中可驗證模型的決策邏輯完全符合真實市場直覺（例如：車位屬性對「單價」產生精準的負向 SHAP 貢獻，捕捉了車位坪數稀釋單價的現象）。

---

## 🧭 Engineering Reflections & Future Roadmap (工程檢討與未來優化)
目前的 MVP 證明了此機器學習框架能有效在空間橫截面上尋找錯價。為符合真實量化交易與精準估價的生產環境 (Production Environment) 標準，下一步優化將著重於以下四個維度：

### 1. 消除未來數據引用與樣本偏差 (Data Leakage & Bias Mitigation)
* **相對定價基準遞延 (Lagged Neutralization):** 目前的 Target 採用「當季均價」進行中性化。實務上在 Live Inference 時，當季均價尚未發生。未來將改採 **上一季的區域均價 (Lagged Quarterly Mean)** 作為比較基準，徹底消除 Look-ahead Bias。
* **重複物件過濾 (Listing Deduplication):** 同一實體物件可能在不同時間點重複上架、降價求售。未來將建立物件追蹤碼 (Property ID tracking) 邏輯，確保時間序列樣本的獨立性，避免模型對特定滯銷物件產生過度擬合 (Overfitting)。

### 2. 另類地理與環境數據擴展 (Alternative Spatial Data)
放棄單純依賴傳統房市開價與格局特徵，計畫透過 API 或爬蟲整合更細緻的外部地理與環境資訊。例如：將「大眾運輸樞紐距離 (交通)」、「採光與日照角度幾何模型」，甚至「嫌惡設施與凶宅分佈」量化為新的 **Alpha Factors**，捕捉傳統定價網站忽略的隱含價值與流動性折價。

### 3. 跨市場資金動能與熱度指標 (Cross-Market Capital Flows)
房地產價格深受總體經濟與游資溢出效應 (Spillover Effect) 的影響。未來模型將納入跨市場資金動能因子（例如：台股大盤指數波動、資金流向股市與房地產的相對比例、M2 貨幣供給量等），從宏觀維度掌握市場的真實熱度與買盤動能。

### 4. 總體政策與法規因子 (Macro Policy & Regulatory Factors)
不動產是高度受政策調控的資產類別。未來計畫將央行信用管制 (如貸款成數上限 LTV Limits)、稅制改革 (如房地合一稅、囤房稅) 及利率決策等宏觀政策變數進行量化編碼。這能大幅提升模型在面對市場結構性轉折 (Structural Breaks) 時的預測韌性與準確度。
