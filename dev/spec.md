# 傾向スコアによる因果推論 - 分析仕様書

## データ概要

- ファイル: `data/ec675_nsw.tab`（タブ区切り、19,204行）
- 分析対象: **NSW処置群 (sample=1, treated=1)** 297名 + **CPS比較群 (sample=2)** 15,992名
  - CPS群は非実験の対照群として treated=0 を割り当てる
- ベンチマーク: NSW実験データ (sample=1) の処置群 vs 対照群の差分をRCTによる真の効果として使用
- アウトカム: re78（1978年の実質所得）
- 共変量: age, educ, black, hispan, married, nodegree, re74, re75

## 分析ステップ

### Step 0: 環境構築・データ読み込み
- [x] 必要パッケージの追加 (sklearn, matplotlib, scipy)
- [x] データ読み込みと前処理
  - タブ区切りで読み込み
  - 分析用サブセットの作成（NSW処置群 + CPS比較群）
  - treated列の整備（CPS群に0を割り当て）
- [x] ベンチマーク効果の算出（NSW RCTデータから真のATTを計算）

### Step 1: 探索的データ分析 (EDA)
- [x] 処置群・対照群の基本統計量の比較
- [x] 共変量の標準化平均差 (SMD) を算出（調整前）
  - 目安: |SMD| < 0.1 でバランス良好

### Step 2: 傾向スコアの推定
- [x] ロジスティック回帰モデルで傾向スコアを推定
  - 説明変数: age, educ, black, hispan, married, nodegree, re74, re75
- [x] 傾向スコアの分布を処置群・対照群で可視化（ヒストグラム、bin=0.05）
- [x] 共通サポート仮定の確認
  - ほとんどのbin で両群のサンプルが存在するか

### Step 3: バランシングの確認
- [x] 傾向スコア調整後の標準化平均差 (SMD) を算出
- [x] 調整前後の SMD を Love plot で比較
  - |SMD| < 0.1 のラインを表示

### Step 4: 効果の推定
- [x] マッチング (ATT)
  - 最近傍マッチング（1対1、復元なし）
  - 介入群に対してマッチングし ATT を算出
- [x] 層別解析
  - 傾向スコアを 0.25 刻みで層に分割
  - 各層の処置効果を算出し、加重平均で全体効果を推定
- [x] IPW (ATE)
  - 重み: 処置群 = 1/e(x), 対照群 = 1/(1-e(x))
  - 重みつき平均の差で ATE を算出

### Step 5: 結果のまとめ
- [x] 結果を表形式でまとめる

| method | estimand | effect |
|--------|----------|--------|
| RCT (benchmark) | ATT | xx |
| Matching | ATT | xx |
| Stratification | ATE | xx |
| IPW | ATE | xx |

## 使用ライブラリ

- pandas, numpy: データ操作
- sklearn (LogisticRegression): 傾向スコア推定
- matplotlib: 可視化
- scipy: 統計処理（必要に応じて）
