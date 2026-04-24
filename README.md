# Air Pollution — COVID-19 Level 3 Alert CausalImpact Analysis

研究 2021 年 5 月 19 日台灣 COVID-19 三級警戒對交通站 NO2 空氣品質的因果效應，使用貝葉斯結構時間序列（BSTS）方法。

## 研究問題

三級警戒實施後，都市交通要地的 NO2 level(交通排放指標) 是否顯著下降？下降幅度有多大？

## 資料

| 項目 | 內容 |
|---|---|
| 來源 | 台灣 EPA API（AQX_P_488） |
| 時間範圍 | 2019-05-19 ～ 2021-06-29 |
| 站點分組 | treatment（都市交通要地）、control1(非都市區域)、control2(都市一般地區) |
| 分析變數 | NO2、PM2.5、PM10、windspeed、wind_x、wind_y、weekday、is_weekend、month |

- `wind_x` / `wind_y`：風向（度）轉換為 cos/sin 笛卡爾分量，避免循環平均誤差
- `_imp_l` 後綴：經線性內插補值（最多補 2 天缺漏），取站點平均後取 log

## 實驗分組
- Treatment(都市交通要地):
    - 交通空氣品質監測站:
        - 永和(新北)
        - 中壢(桃園)
        - 鳳山、復興(高雄)

- Control1 (非都市區域):
    - 背景空氣品質監測站:三義、橋頭、富貴

- Control2 (都市一般地區):
    - 一般空氣品質監測站:
        - 板橋、菜寮、士林(新北)
        - 平鎮(桃園)
        - 左營、前金、小港(高雄)

## 分析設計

- **Pre-period**：2019-05-19 ～ 2021-05-18
- **Post-period**：2021-05-19 ～ 2021-06-29
- **Outcome**：`treatment_no2_imp_l`
- **方法**：[causalimpact](https://github.com/WillianFuks/tfcausalimpact)（TensorFlow Probability BSTS）

### 協變量組合（4 組）

| 組別 | 協變量來源 |
|---|---|
| 1\_treat\_only | treatment 的 PM2.5、PM10、windspeed、wind\_x、wind\_y + calendar |
| 2\_treat\_c1 | 組 1 + control1 的 **NO2**、PM2.5、PM10、windspeed、wind\_x、wind\_y |
| 3\_treat\_c2 | 組 1 + control2 的 **NO2**、PM2.5、PM10、windspeed、wind\_x、wind\_y |
| 4\_treat\_c1\_c2 | 組 1 + control1 + control2 的 **NO2**、PM2.5、PM10、windspeed、wind\_x、wind\_y |

> treatment 本身的 NO2 不納入協變量（即為 outcome），control 站的 NO2 則完整保留。

## 結果圖

CausalImpact 分析結果包含三個主要部分：

1. **Observed vs. Counterfactual**
   - 實際觀測到的 treatment NO2
   - 若未實施三級警戒時模型預測的反事實 NO2

2. **Pointwise Effect**
   - 每一天實際值與反事實預測值的差異
   - 用來觀察政策實施後每日 NO2 的下降幅度

3. **Cumulative Effect**
   - 政策期間累積效果
   - 用來衡量三級警戒整體造成的 NO2 減少量

### Model 1：Treatment only

![Model 1 CausalImpact Result](https://github.com/user-attachments/assets/7b7c8893-0e4c-4dca-943b-16fc77389476)

### Model 2：Treatment + Control1

![Model 2 CausalImpact Result](https://github.com/user-attachments/assets/2c9fac3d-efbf-4cff-bd63-7792d388d537)

### Model 3：Treatment + Control2

![Model 3 CausalImpact Result](https://github.com/user-attachments/assets/e4b983f3-a918-44e1-a232-3a1d61a235ce)

### Model 4：Treatment + Control1 + Control2

![Model 4 CausalImpact Result](https://github.com/user-attachments/assets/570f5bad-bc49-4258-96db-ca70b7f898f9)



Calendar 特徵（weekday、is\_weekend、month）每組只保留一份。

## 結論

四組模型一致顯示，三級警戒期間（2021-05-19 ～ 2021-06-29）都市交通要地的 NO2 出現顯著下降。

### 各模型估計效果

| 模型 | 協變量 | 相對效果 | 95% CI | p-value |
|---|---|---|---|---|
| 1\_treat\_only | 無 control NO2 | **-17.9%** | -31.8% \~ +5.2% | 0.050 |
| 2\_treat\_c1 | +非都市背景 NO2 | **-15.1%** | -20.6% \~ -8.3% | 0.001 |
| 3\_treat\_c2 | +都市一般 NO2 | **-11.3%** | -16.7% \~ -5.5% | 0.001 |
| 4\_treat\_c1\_c2 | +C1 & C2 NO2 | **-9.8%** | -15.7% \~ -3.8% | 0.002 |

### 主要發現

1. **因果效應顯著**：納入 control 站 NO2 後（Model 2–4），p 值皆 ≤ 0.002，效應在統計上穩健。
2. **效果隨控制組納入而縮減**：Model 1 的 -17.9% 中，部分源於同期大環境因素（氣象、全區污染下降）；Model 4 排除共同趨勢後，最保守估計為 **約 -9.8%**。
3. **最保守規格（Model 4）解讀**：三級警戒使交通要地 NO2 日均值相較反事實降低約 **9.8%**（絕對值 -1.25 µg/m3/day，累積 -52.3 µg/m3），且效應顯著異於零（p = 0.002）。
4. **Model 1 邊際顯著**：無 control NO2 時模型不確定性高（SD = 1.50 vs Model 4 的 0.42），效應區間跨越零，需謹慎解讀。

> **結論**：2021 年台灣三級警戒後，都市交通站 NO2 顯著低於反事實預測值；剔除背景污染共同變動後，政策本身造成的 NO2 降幅約為 **10%**。

## 檔案結構

```
air_pollution/
├── preprocess.ipynb          # ETL：API 抓取 → 日平均 → 補值 → daily_imp2.csv
├── BSTB.ipynb                # CausalImpact 分析（含執行結果與圖）
└── api.py                    # EPA API key（不納入版本控制）
```

