# Real-Estate Pricing Anomaly Detection Model

## 1. Problem Statement
本專案旨在建立一個不動產相對定價模型。相較於預測絕對房價，本研究聚焦於尋找「空間橫截面上的定價異常 (Pricing Anomalies)」。透過建立區域中性化 (Regional Neutralization) 的基準，過濾大盤漲跌趨勢，試圖從非結構化的開價數據中，篩選出具備潛在低估空間的物件訊號 (Candidate Screening Signal)。

## 2. Dataset
* **資料來源：內政部交易實價登錄、bigfun比房網。

## 3. Methodology
* **Data Engineering:** 針對 144 萬筆資料實作 Winsorization (首尾 1% 極端值截斷) 以降低離群值對模型的擾動；空間座標 (x, y) 缺失值採 `groupby(['dist', 'road'])` 階層式均值填補。
* **Target Formulation:** 為消除行政區本身的絕對地段價值差異，建立相對折溢價指標：
  `Target = Listing_Unit_Price / District_Quarter_Listing_Mean` 
  *(註：此 Target 定義目前僅供 Offline Research 使用。因預測當下無法取得完整的當季均價，尚不適用於 Live Inference，詳見 Roadmap)*
* **Model Selection:** 採用 LightGBM 迴歸模型。其基於 Histogram 的決策樹架構能有效切割二維地理座標 (Lat, Lon)，捕捉非線性的空間溢價特徵。

## 4. Validation Design
為盡可能模擬真實推論環境 (Live Inference) 並減少時間洩漏 (Temporal Leakage)，本專案捨棄隨機切分 (Random Split)，採用嚴格的 Chronological Split：
* **Train Set:** 2023 - 2024
* **Validation Set:** 2025 (For Early Stopping 防止過擬合)
* **Test Set:** 2026Q1 (Out-of-sample 盲測)

## 5. Results & Ablation Study (2026Q1 Test Set)
在未見的 2026Q1 測試集中，我們進行了特徵消融實驗 (Ablation Study) 與模型橫向比較。結果顯示，LightGBM 在加入正交特徵後，展現出較低的 Test MSE 與較高的 Out-of-sample $R^2$，效能優於線性迴歸與區域均值基準模型：

| Model | Feature Strategy (特徵策略) | Test MSE | Test R-squared | Notes |
|:---|:---|:---:|:---:|:---|
| Null Baseline | Baseline (District Mean) | 0.1109 | -0.0002 | 基準線 (猜平均) |
| Ridge (Linear) | Basic Features (OHE dist) | 0.1047 | 0.0559 | 線性對照組 |
| LightGBM | Model A (無坪數特徵) | 0.0602 | 0.4612 | 基於空間地理特徵 |
| LightGBM | + Efficiency_Ratio (得房率) | 0.0542 | 0.5144 | 捕捉公設比之隱藏定價 |
| LightGBM | + Size (總坪數) | 0.0524 | 0.5310 | 捕捉規模折價效應 |
| **LightGBM (Proposed)** | **+ Size + Efficiency_Ratio** | **0.0503** | **0.5500** | **雙重特徵組合 (主力模型)** |
| Random Forest | + Size + Efficiency_Ratio (OHE) | 0.0502 | 0.5505 | 樹狀模型驗證對照 |

## 6. Interpretability (SHAP Analysis)
透過 SHAP 進行特徵歸因，觀察模型是否學習到符合市場直覺的定價邏輯：
1. **車位稀釋效應:** 車位特徵對「單價 Target」產生穩定的負向 SHAP 值，符合車位坪數拉低整體均價的數學特性。
2. **非線性屋齡特徵:** 模型顯示屋齡與價值間存在 U 型關係。在特定精華地段，極高屋齡物件獲得正向 SHAP 貢獻，**此現象暗示模型可能捕捉到了與都市更新 (Urban Renewal) 直覺一致的潛在溢價**，而非單純的線性折舊。

## 7. Future Engineering Roadmap (未來優化藍圖)
本 POC 在空間橫截面上已初步驗證定價異常 (Pricing Anomalies) 的捕捉能力。為推進至具備實戰價值的量化篩選系統，後續優化將聚焦於以下四個維度：

1. **消除未來數據引用 (Look-ahead Bias Mitigation):** 目前 Target 採用「當季均價」進行中性化。實務 Live Inference 時，將改採 **「上一季區域均價 (Lagged Quarterly Mean)」** 作為分母基準，徹底消除 Look-ahead Bias；並建立 Property ID 追蹤邏輯，過濾重複上架之物件以確保樣本獨立性。
2. **流動性與去化門檻 (Liquidity Constraints):** 不動產屬低頻交易資產，缺乏流動性的低估物件無變現價值。未來將結合歷史週轉率 (Turnover Rate) 設定流動性過濾機制，萃取出兼具「價格 Alpha」與「交易可行性」的標的。
3. **另類空間數據擴展 (Alternative Spatial Data):** 透過 API 整合外部地理矩陣，將「大眾運輸樞紐距離」、「日照幾何角度」及「嫌惡設施分佈範圍」等非傳統特徵量化為新的 Alpha Factors。
4. **總體經濟與跨市場因子 (Macro & Cross-Market Factors):** 納入台股大盤波動率、資金流向比例等跨市場資金動能，並將央行信用管制 (LTV Limits) 等宏觀法規進行量化編碼，提升模型面對結構性轉折 (Structural Breaks) 時的預測韌性。

---

## 8. Reproducibility

### Repo Structure
```text
.
├── data/
│   └── sample_data.csv        # Data schema example (100 rows)
├── images/
│   └── shap_summary.png       # SHAP interpretation chart
├── estate_model.py            # 資料處理、模型訓練與評估 Pipeline
└── README.md
