# NBD-Dirichlet 多カテゴリ分析 - 仕様書

## 概要

Dunnhumby "The Complete Journey" の複数カテゴリに対して NBD モデルと
ディリクレ多項分布モデルを当てはめ、カテゴリごとの $K$（購入頻度の異質性）と
$S$（ブランドロイヤルティ）を散布図上にマッピングする。

## 成果物

- `src/dmnbd.ipynb` - 分析ノートブック（上から順に再現可能）
- `article/output_dmnbd.md` - 記事（常体）

## 技術スタック

- Python（uv 管理）: pandas, numpy, scipy, matplotlib, seaborn
- データ: Dunnhumby "The Complete Journey"（Kaggle 経由）

---

## ステップ

### Step 0: 環境準備

- [ ] `src/dmnbd.ipynb` を新規作成
- [ ] import セル（os, pandas, numpy, matplotlib, seaborn, scipy.special.gammaln, scipy.optimize.minimize）

### Step 1: データ取得・前処理

- [ ] Kaggle API で CSV ダウンロード（`transaction_data.csv`, `product.csv`）
- [ ] 対象カテゴリの選定（下記「カテゴリ選定方針」参照）
- [ ] カテゴリごとに household × 52週間フィルタ適用

### Step 2: カテゴリ選定

分析対象は以下の条件をすべて満たすカテゴリを選ぶ。

- 購買世帯数 ≥ 200
- 主要ブランド（MANUFACTURER）が 3 以上存在する
- 52週間の購買件数 ≥ 1,000

- [ ] `commodity_desc` ごとに上記集計を行い、対象カテゴリ一覧を確定する
- [ ] 候補として SOFT DRINKS, CHEESE, FLUID MILK PRODUCTS, COLD CEREAL 等を検討

### Step 3: カテゴリごとの NBD フィッティング

各カテゴリについて：

- [ ] 世帯ごとの購買回数 $r_i$ を集計
- [ ] NBD の対数尤度を最大化して $(M, K)$ を推定
  - $M$: 平均購買回数、$K$: shape（小さいほどヘビーバイヤー偏在）
- [ ] フィッティング結果を DataFrame `df_nbd` に格納（列: `category`, `M`, `K`）

### Step 4: カテゴリごとのディリクレ多項分布フィッティング

各カテゴリについて：

- [ ] 購買回数上位 5 MANUFACTURER + Others の 6 ブランドに集約
- [ ] 世帯 × ブランドのワイド形式 DataFrame を作成
- [ ] ディリクレ多項分布の MLE で $\boldsymbol{\alpha}$ を推定、$S = \sum_j \alpha_j$ を算出
- [ ] フィッティング結果を DataFrame `df_dm` に格納（列: `category`, `S`）

### Step 5: $K$ × $S$ マッピング

- [ ] `df_nbd` と `df_dm` を `category` キーで結合
- [ ] 散布図を作成（x 軸: $K$、y 軸: $S$、各点にカテゴリ名を付記）
- [ ] 4象限の解釈をアノテーションで追加

### Step 6: 記事作成

- [ ] `article/output_dmnbd.md` を `/analyze-article` コマンドで執筆

---

## データ仕様

### NBD パラメータ

$$r_i \sim \mathrm{NegBin}(M, K), \quad \mathrm{Var}(r) = M + M^2/K$$

- $K$ が大きい → 消費者間の購買回数が均一（全員が似たペースで買う）
- $K$ が小さい → ヘビーバイヤーが偏在（少数が大量購入）

### ディリクレ集中度パラメータ

- $S$ が大きい → ブランドロイヤルティが高い（繰り返し購買が多い）
- $S$ が小さい → ブランドスイッチングが頻繁

### $K$ × $S$ 4象限の解釈

|  | $S$ 大（ロイヤルティ高） | $S$ 小（ロイヤルティ低） |
|---|---|---|
| **$K$ 大**（購入頻度が均一） | 成熟した日用品市場（例：歯磨き粉） | 差別化が弱い市場（例：ミネラルウォーター） |
| **$K$ 小**（ヘビーバイヤー偏在） | ニッチ嗜好品市場（例：クラフトビール） | プロモーション主導の市場（例：スナック菓子） |

---

## 注意事項

- NBD フィッティングは `negative_binomial.ipynb` の実装を再利用する
- ディリクレ MLE は `dirichlet_multinomial.ipynb` の `dm_neg_loglik` を再利用する
- 収束しないカテゴリは対象から除外してよい
