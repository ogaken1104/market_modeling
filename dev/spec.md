# 市場モデリング分析 - 仕様書

## 概要

Dunnhumby "The Complete Journey" の SOFT DRINKS カテゴリを対象に、3つの統計モデル（負の二項分布、ディリクレ多項分布、NBD-Dirichlet モデル）で市場をモデリングする。分析結果は `article/output.md` に記事として出力する。

## 成果物

- `src/analysis.ipynb` - 分析ノートブック（単一ファイル、上から順に再現可能）
- `article/output.md` - 記事（常体）

## 技術スタック

- Python 3.11 / uv
- scipy（分布フィッティング・最適化）
- kaggle（データ取得）
- pandas, numpy, matplotlib, seaborn

---

## ステップ

### Step 0: 環境準備
- [x] scipy, kaggle を pyproject.toml に追加し `uv sync`

### Step 1: データ取得・前処理
- [x] Kaggle API で Dunnhumby "The Complete Journey" データを取得
  - `kaggle datasets download frtgnn/dunnhumby-the-complete-journey` → `data/dunnhumby/` に配置
  - `~/.kaggle/kaggle.json` が設定済みであること
- [x] product テーブルから SOFT DRINKS の product_id を抽出
- [x] トランザクションをフィルタリング（SOFT DRINKS のみ）
- [x] 期間を最初の52週間（1年間）に絞る
- [x] 世帯ごとの購入回数を集計
- [x] ブランド集約（MANUFACTURER 上位5 + Others の6カテゴリ）
  - 注: BRAND カラムは National/Private の2値のみのため MANUFACTURER コードで代替

### Step 2: 負の二項分布（NBD）モデル
- [x] モデルの数式を Markdown セルで解説
  - ガンマ分布 × ポアソン分布 → 負の二項分布の導出
  - パラメータ r, p の意味（購買の異質性）
- [x] 世帯ごとの購入回数分布にフィッティング（scipy.stats.nbinom + MLE）
- [x] 実測値 vs フィッティング結果の比較（棒グラフ、ポアソンとの比較付き）
- [x] パラメータの解釈（平均購入回数、分散、異質性の度合い）

### Step 3: ブランド別 NBD フィッティング（方針変更）
- [x] ブランド別の購入回数データ作成（0購入世帯を含む）
- [x] 各ブランドに NBD をフィッティング（MLE）
- [x] 浸透率（Penetration）と購入頻度（Buying Rate）の算出・比較
- [x] Penetration vs Buying Rate の散布図
- [x] ブランド別フィッティング結果のヒストグラム（6パネル）
- [x] サマリテーブル（浸透率順ソート）

※ ディリクレ多項分布モデルは `src/dirichlet_multinomial.ipynb` に退避

### Step 5: 記事作成
- [x] `article/output.md` に分析結果を記事としてまとめる
  - 導入：なぜ統計モデルで市場をモデリングするのか
  - 手法：NBD モデルの数式と解説
  - 結果：カテゴリ全体 + ブランド別のフィッティング結果
  - 考察：浸透率 vs 購入頻度の比較、マーケティング施策への示唆
  - 参考文献

---

## データ仕様

| テーブル | 主要カラム | 用途 |
|---------|-----------|------|
| transaction_data | household_key, BASKET_ID, DAY, PRODUCT_ID, QUANTITY, SALES_VALUE | 購買履歴 |
| product | PRODUCT_ID, COMMODITY_DESC, BRAND, MANUFACTURER | 商品マスタ |

## 注意事項

- ブランド集約: 上位5ブランドを個別に扱い、残りは「Others」にまとめる（ディリクレのパラメータ推定収束のため）
- 期間: 最初の52週間で定常性を確保
- ノートブックの Markdown セルは日本語で記述
