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

Calendar 特徵（weekday、is\_weekend、month）每組只保留一份。

## 檔案結構

```
air_pollution/
├── preprocess.ipynb          # ETL：API 抓取 → 日平均 → 補值 → daily_imp2.csv
├── BSTB.ipynb                # CausalImpact 分析（含執行結果與圖）
└── api.py                    # EPA API key（不納入版本控制）
```

## 執行方式

```bash
# 環境
conda activate air

# 前處理（重新抓資料）
jupyter nbconvert --to notebook --execute preprocess.ipynb

# CausalImpact 分析
CUDA_VISIBLE_DEVICES="" jupyter nbconvert --to notebook --execute BSTB.ipynb --output BSTB.ipynb
```
