# 傾向スコアによる因果推論（401kデータ） - 分析仕様書

## データ概要

- データセット: `wooldridge.data('401ksubs')`（9,275行 × 11列）
- 出典: 1991 Survey of Income and Program Participation (SIPP)、Abadie (2003) が使用
- 処置変数: `e401k`（=1 なら 401(k) 適格）
  - 雇用主が提供するかどうかで決まるため、個人の貯蓄意欲とは比較的独立
- アウトカム: `nettfa`（純金融資産、単位: $1,000）
- 共変量: `inc`（年収）, `marr`（既婚）, `male`（男性）, `age`（年齢）, `fsize`（家族人数）
  - `incsq`, `agesq` は `inc`, `age` の二乗項（必要に応じて使用）

## ノートブック

- ファイル: `src/analysis_401k.ipynb`（新規作成）

## 分析ステップ

### Step 0: 環境構築・データ読み込み
- [ ] 必要パッケージのインポート (pandas, numpy, sklearn, matplotlib, wooldridge)
- [ ] データ読み込みと確認
  - `wooldridge.data('401ksubs')` でデータ取得
  - 基本的な形状・欠損値の確認

### Step 1: 探索的データ分析 (EDA)
- [ ] 処置群 (`e401k=1`) ・対照群 (`e401k=0`) の基本統計量の比較
- [ ] 共変量の標準化平均差 (SMD) を算出（調整前）
  - 目安: |SMD| < 0.1 でバランス良好

### Step 2: 傾向スコアの推定
- [ ] ロジスティック回帰モデルで傾向スコアを推定
  - 説明変数: inc, marr, male, age, fsize, incsq, agesq
- [ ] 傾向スコアの分布を処置群・対照群で可視化（ヒストグラム）
- [ ] 共通サポート仮定の確認

### Step 3: バランシングの確認
- [ ] 傾向スコア調整後の標準化平均差 (SMD) を算出
- [ ] 調整前後の SMD を Love plot で比較
  - |SMD| < 0.1 のラインを表示

### Step 4: 効果の推定
- [ ] マッチング (ATT)
  - 最近傍マッチング（1対1、復元なし）
  - 介入群に対してマッチングし ATT を算出
- [ ] 層別解析 (ATE)
  - 傾向スコアを層に分割
  - 各層の処置効果を算出し、加重平均で全体効果を推定
- [ ] IPW (ATE)
  - 重み: 処置群 = 1/e(x), 対照群 = 1/(1-e(x))
  - 重みつき平均の差で ATE を算出

### Step 5: 結果のまとめ
- [ ] 結果を表形式でまとめる

| method         | estimand | effect |
| -------------- | -------- | ------ |
| Naive (unadj.) | -        | xx     |
| Matching       | ATT      | xx     |
| Stratification | ATE      | xx     |
| IPW            | ATE      | xx     |

## 使用ライブラリ

- pandas, numpy: データ操作
- sklearn (LogisticRegression): 傾向スコア推定
- matplotlib: 可視化
- wooldridge: データセット取得
