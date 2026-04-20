# GMDB Simulation & Valuation (Investment-Linked Insurance)

## 📌 Project Overview

### 本專案建立一個投資型壽險（Investment-Linked Insurance）之模擬模型，分析不同 GMDB（Guaranteed Minimum Death Benefit）設計對保單風險、成本與利潤的影響。透過 Monte Carlo 模擬與 Risk-Neutral 定價，本專案從Real-world simulation及Risk-neutral valuation兩個角度分析產品。
---

## 🎯 Objectives

- 比較不同 GMDB 設計（Basic / Ratchet / Roll-up）
- 分析關鍵指標：
  - Account Value (AV)
  - Death Benefit (DB)
  - Guarantee Cost (GC)
  - Tail Risk
  - Profit Distribution
- 評估產品設計對風險與經濟成本的影響
- 使用 risk-neutral 方法衡量保證的市場一致價值

---

## 🧩 Model Framework

### Simulation Flow

```mermaid
graph TD
    %% 主流程
    Input[定義保單設計與精算因子] --> Mechanism{帳戶初始設定}
    Mechanism --> Simulation[蒙地卡羅模擬流程]
    Simulation --> Risk-neutral[改用Risk-neutral評價Guarantee cost]
    Risk-neutral --> Result[Profit and Risk Metrics]

    %% 蒙地卡羅詳細步驟 (子圖)
    subgraph Simulation [蒙地卡羅模擬流程]
        S1[隨機抽樣死亡判定] -->|U < qx| Dead[死亡: 觸發理賠]
        S1 -->|U >= qx| Alive[生存]
        Dead --> DB[計算給付額(DB)、GC: Basic/Ratchet/Roll-up]
        Alive --> |AV不足 lapse| Lapse[失效: surrender value]
        Alive --> |存續|S2[模擬標的資產路徑: GBM、套用保單結構更新AV]
        S2 --> |進入下個月模擬| S1
    end

    style Dead fill:#ffe9ef,stroke:#ffdee7
    style Alive fill:#ffe9ef,stroke:#ffdee7
    style Simulation fill:#c0d9d9,stroke:#01579b,stroke-width:2px
```

    %% 蒙地卡羅詳細步驟 (子圖)
    subgraph Simulation [蒙地卡羅個體模擬流程]
        S1[模擬標的資產路徑: GBM] --> S2[隨機抽樣死亡判定]
        S2 -->|U < qx| Dead[死亡: 觸發理賠]
        S2 -->|U >= qx| Alive[生存]
        Alive --> |AV不足 lapse| Lapse[失效: surrender value]
        Alive --> S4[存續: 套用保單結構更新AV]
        Dead --> S3[計算給付額: Basic/Ratchet/Roll-up]
        
        S4 -->|進入下個月模擬| S1
    
Fund Return Simulation
↓
Account Value (AV)
↓
Mortality Event
↓
Death Benefit (DB)
↓
Guarantee Cost (GC)
↓
Profit Calculation


---

## 🏗 GMDB Design

### 1. Basic
- 固定保證（例如：160 萬）
- 結構最簡單，成本最低

### 2. Ratchet
- 鎖定過去最高 AV
- 提供額外保障，但成本增加有限

### 3. Roll-up
- 保證隨時間成長
- 顯著提高 guarantee 成本與風險

---

## 📊 Key Results

### 1. Account Value (AV)

- 三種設計 AV 路徑相同 (因目前保證結構設計不影響投資績效、Fee、COI)
    
![圖片描述](Images/av.png)
---

### 2. Death Benefit (DB)
- 高 AV 情境：三者趨於一致 (p95 接近、 p99 完全一樣，顯示極端情境由高帳戶價值(報酬表現非常好的情況) 主導，而非 guarantee 結構。
- 低 AV 情境：差異顯著（Roll-up 提高死亡給付的平均值與中位數)

| Metric                           | Basic          | Ratchet        | Roll-up        |
|:---------------------------------|:---------------|:---------------|:---------------|
| Conditional Mean Death Benefit   | 1,825,812.1478 | 1,866,651.6769 | 2,412,352.2847 |
| Conditional Median Death Benefit | 1,600,000.0000 | 1,600,000.0000 | 2,456,184.9075 |
| Conditional P95 Death Benefit    | 3,004,062.7230 | 3,017,255.3892 | 3,011,555.8759 |
| Conditional P99 Death Benefit    | 3,651,355.1438 | 3,651,355.1438 | 3,651,355.1438 |


![圖片描述](Images/db.png)
![圖片描述](Images/dbkde.png)
---

### 3. Guarantee Cost (GC)

- Basic < Ratchet < Roll-up
- Roll-up 顯著增加 tail risk

| Metric                                     | Basic        | Ratchet      | Roll-up        |
|:-------------------------------------------|:-------------|:-------------|:---------------|
| Conditional Mean Guarantee Cost            | 320,104.0512 | 360,943.5802 | 906,644.1881   |
| Conditional Trigger Rate                   | 0.6829       | 0.8171       | 0.9268         |
| Conditional P95 Guarantee Cost             | 916,126.4385 | 916,126.4385 | 1,662,081.2362 |
| Conditional P99 Guarantee Cost             | 977,597.9856 | 977,597.9856 | 2,125,561.9181 |
| Worst 5% Mean Guarantee Cost (Conditional) | 954,618.2510 | 954,618.2510 | 1,955,866.0768 |

![圖片描述](Images/gc.png)
![圖片描述](Images/gckde.png)
---

### 4. Tail Risk

- 保單具有明顯 tail risk
- 少數情境產生極端虧損
- Roll-up 放大下檔風險

---

## 💰 Profit Analysis

| Design | Mean Profit |
|--------|------------|
| Basic | Positive |
| Ratchet | Slightly lower |
| Roll-up | Negative |

<br>

 | Metric    | Basic           | Ratchet         | Roll-up         |
|:----------|----------------:|----------------:|----------------:|
| mean      | 16,925.4031     | 14,559.1833     | -18,722.2021    |
| median    | 131,454.6340    | 131,454.6340    | 131,454.6340    |
| p5        | -1,145,815.2663 | -1,178,607.3289 | -1,673,094.2621 |
| p95       | 152,407.9392    | 152,407.9392    | 152,407.9392    |
| p99       | 164,025.9892    | 164,025.9892    | 164,025.9892    |
| loss_prob | 0.0820          | 0.0820          | 0.0820          |


![採log scale](Images/profit.png)


### Insights

- 大部分情境下保單穩定獲利（median 高）
- 少數情境產生巨大虧損（tail risk）
- Roll-up 出現 underpricing

👉 結構類似：
Small Gain (frequent) + Large Loss (rare)


---

## 📈 Risk-Neutral Valuation

| Design | Guarantee Value |
|--------|---------------|
| Basic | Low |
| Ratchet | Slightly higher |
| Roll-up | Significantly higher |

<br>

| Guarantee Type            |   GMDB Price |
|:--------------------------|-------------:|
| Basic (Return of Premium) |  25,280.0550 |
| Ratchet                   |  26,208.6969 |
| Roll-up                   |  68,236.2533 |


### Interpretation

- GMDB ≈ Embedded Put Option
- Roll-up 顯著提高 option value
- 成本主要來自 downside risk

---

## 🔍 Real-world vs Risk-neutral

| Perspective | Meaning |
|------------|--------|
| Real-world | 實際會發生什麼 |
| Risk-neutral | 市場覺得值多少 |

👉 兩者互補，用於：

- Profit analysis
- Pricing


---

## 💡 Key Insights

- GMDB 設計影響 tail risk，而非平均報酬
- Ratchet 是低成本保障升級
- Roll-up 顯著增加風險與成本
- 保單結構類似 short put payoff

---

## ⚠️ Practical Implications

- Roll-up 需更高費率或 hedge
- Ratchet 可作為平衡風險與吸引力的設計
- Risk-neutral valuation 可用於：
  - 定價
  - 避險成本評估

---

## 📌 Conclusion

1. GMDB 商品的核心風險來自於下檔保障機制不同， GMDB 設計在一般情境下的損益分布相近，但其差異主要來自少數極端虧損情境，因此平均結果的差異主要由 tail risk 所驅動，而GMDB 設計對尾端風險影響更為明顯。
2. Ratchet 屬於相對低成本的保障升級；Roll-up 雖提供較高保障，但需搭配更高費率或風險管理機制。


---
