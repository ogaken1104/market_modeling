# NBD とディリクレ多項分布で市場を解釈する

## はじめに

以前の記事で、 SOFT DRINKS カテゴリ単体に対して [NBD モデル](https://qiita.com/ground__around/items/7fda63ac989c5060a20b)と[ディリクレ多項分布モデル](https://qiita.com/ground__around/items/cfd1d4f67ded3678fe44)を当てはめた。単体カテゴリの分析で得られるのは「そのカテゴリの $K$・$S$ の値」だが、複数の製品カテゴリ間で $K$ と $S$ を比較することで、カテゴリごとの市場構造の違いを定量的に把握したい。

本稿では 10 カテゴリに両モデルを適用し、カテゴリごとの $K$（購入頻度の均一性）と $S$（消費者間の選好の均質性）を散布図上に配置し解釈をおこなう。

## データ

Dunnhumby "The Complete Journey" のトランザクションデータを使用する。期間は最初の 52 週間（DAY 1〜364）に絞り、定常性を確保する。

### カテゴリ選定条件

以下の基準をすべて満たすカテゴリを分析対象とする。

- 購買世帯数 ≥ 200
- 主要メーカー（MANUFACTURER）≥ 3
- 52週間の購買件数 ≥ 1,000

選定された 10 カテゴリと基本統計を以下に示す。

| カテゴリ               | 購買世帯数 | 購買件数 | メーカー数 |
| ---------------------- | ---------- | -------- | ---------- |
| SOFT DRINKS            | 2,272      | 56,090   | 33         |
| FLUID MILK PRODUCTS    | 2,288      | 39,146   | 29         |
| CHEESE                 | 2,167      | 33,693   | 33         |
| BAG SNACKS             | 2,139      | 30,676   | 42         |
| BAKED BREAD/BUNS/ROLLS | 2,269      | 37,788   | 41         |
| COLD CEREAL            | 1,839      | 17,152   | 8          |
| YOGURT                 | 1,423      | 20,857   | 8          |
| FROZEN PIZZA           | 1,742      | 20,772   | 35         |
| SOUP                   | 1,878      | 20,968   | 36         |
| FRZN MEAT/MEAT DINNERS | 1,756      | 25,334   | 38         |

## 手法

### NBD モデル（K パラメータの推定）

世帯 $i$ の年間購買回数 $r_i$ が負の二項分布に従うとモデル化する。

$$r_i \sim \mathrm{NegBin}(M, K), \qquad \mathrm{Var}(r) = M + \frac{M^2}{K}$$

- $M$：平均購買回数
- $K$：形状パラメータ。大きいほど消費者間の購買回数が均一（$K \to \infty$ でポアソン分布に収束）、小さいほどヘビーバイヤーへの偏在が強い

全世帯（購買ゼロを含む）の購買回数ベクトルに対して MLE で $(M, K)$ を推定する。

### ディリクレ多項分布モデル（S パラメータの推定）

各カテゴリの購買世帯について、購買回数上位 5 メーカー + Others の 6 ブランドグループに集約する。世帯 $i$ のブランド購買回数ベクトル $\mathbf{x}_i$ がディリクレ多項分布に従うとモデル化する。

$$\boldsymbol{\theta}_i \sim \mathrm{Dirichlet}(\boldsymbol{\alpha}), \qquad \mathbf{x}_i \mid \boldsymbol{\theta}_i \sim \mathrm{Multinomial}(n_i, \boldsymbol{\theta}_i)$$

$\boldsymbol{\theta}_i$ を積分消去すると周辺分布としてディリクレ多項分布が得られる。MLE で $\boldsymbol{\alpha}$ を推定し、集中度パラメータ $S = \sum_j \alpha_j$ を算出する。

$S$ が小さいほど消費者間の異質性が大きく、個人ロイヤルティが高い。$S$ が大きいほど消費者間が均質で、繰り返し購買確率はシェアに近づく。

## 結果
![K × S カテゴリマッピング](fig_ks_map.png)

### NBD フィッティング結果（K パラメータ）の解釈

Milk・Bread/Rolls は $K$ が高く、消費者間の購買回数のばらつきが小さい——多くの家庭が似た頻度で購入する日用品的な性質を反映している。一方 Yogurt は $K = 0.236$ と最も低く、少数のヘビーバイヤーが購買の大部分を占める構造になっている。

### ディリクレ多項分布フィッティング結果（S パラメータ）の解釈

Cheese・Bag Snacks は $S$ が高く、消費者間の選好が均質で、繰り返し購買確率がシェアに近い。一方 Yogurt・Frozen Meat・Soup は $S$ が低く、各消費者が特定のブランドに固執する傾向が強い。


## 考察

### K × S によるマーケティング施策の方向性

K と S の組み合わせは、カテゴリに対してどんなマーケティング施策が有効かの方向性を示す。

「$K$ が低いカテゴリ（ヘビーバイヤー偏在）」では、購買の大部分を少数世帯が担っている。このようなカテゴリでは「非購買世帯を新規獲得する施策」よりも「既存ヘビーバイヤーの維持」を優先する合理性がある。Yogurt（$K=0.236$）はその典型で、購買世帯数自体は少なくないが、購買頻度の分散が非常に大きい。

「$S$ が高いカテゴリ（消費者間の選好が均質）」では、消費者間のブランド選好のばらつきが小さく、繰り返し購買確率がシェアに近づく。Cheese（$S=5.74$）では $(\alpha_j+1)/(S+1) \approx \alpha_j/S$ となり、前回選んだブランドへの固執が弱い。

### Yogurt について

Yogurt は $K = 0.236$（最小）かつ $S = 0.83$（最小）である。これは、ヨーグルト自体の購買回数・ブランド別の購買確率の両方が消費者間で大きくばらついていることを意味する。つまり、ヨーグルトは「少数のヘビーバイヤーが特定ブランドに固執して繰り返し購入する」という構造になっている。

### 分析の前提と限界

本分析にはいくつかの留意点がある。

- **ブランド集約の粗さ**: 上位5メーカー + Others という集約は、メーカー数が多いカテゴリ（BAG SNACKS: 42メーカー）では「Others」にシェアが集中し、個別ブランドの解像度が下がる
- **IIA 制約**: ディリクレ多項分布は「どのブランドからでも流入確率はシェアに比例する」という IIA 制約を持つ。ブランド間の構造的な近接性（例：同一フレーバー間のスイッチング）は捉えられない

## ソースコード

### 環境構築

Python 3.11 以上、[uv](https://docs.astral.sh/uv/) を使用。

```toml
# pyproject.toml
[project]
name = "jupyter-uv"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "pandas>=2.2.3",
    "numpy>=2.2.3",
    "scikit-learn>=1.6.1",
    "matplotlib>=3.10.0",
    "seaborn>=0.13.2",
    "jupyter>=1.1.1",
    "nbconvert>=7.17.0",
    "ipykernel>=7.0.1",
    "kaggle>=2.0.0",
]
```

```bash
uv sync
# Kaggle API キーを ~/.kaggle/kaggle.json に配置してからデータ取得
kaggle datasets download frtgnn/dunnhumby-the-complete-journey -p data/dunnhumby --unzip
```

### 分析コード

```python
import os
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.special import gammaln
from scipy.optimize import minimize
from scipy.stats import nbinom

sns.set_theme(style="whitegrid")
plt.rcParams["font.family"] = "Hiragino Sans"

# --- データ読み込み ---
DATA_DIR = "data/dunnhumby"
df_trans = pd.read_csv(os.path.join(DATA_DIR, "transaction_data.csv"))
df_prod = pd.read_csv(os.path.join(DATA_DIR, "product.csv"))

df_trans = df_trans[df_trans["DAY"] <= 364].merge(
    df_prod[["PRODUCT_ID", "COMMODITY_DESC", "MANUFACTURER"]], on="PRODUCT_ID"
)

# --- カテゴリ選定 ---
TARGET_CATEGORIES = [
    "SOFT DRINKS", "FLUID MILK PRODUCTS", "CHEESE", "BAG SNACKS",
    "COLD CEREAL", "YOGURT", "FROZEN PIZZA", "SOUP",
    "FRZN MEAT/MEAT DINNERS", "BAKED BREAD/BUNS/ROLLS",
]

# --- NBD フィッティング ---
def fit_nbd(counts):
    """NBD MLE: returns (K, M) where K=shape, M=mean."""
    mean_x = counts.mean()
    var_x = counts.var()
    p_init = mean_x / var_x if var_x > mean_x else 0.5
    r_init = max(mean_x * p_init / (1 - p_init + 1e-9), 0.1)

    def neg_loglik(params):
        r, p = params
        if r <= 0 or p <= 0 or p >= 1:
            return 1e10
        return -nbinom.logpmf(counts, n=r, p=p).sum()

    result = minimize(neg_loglik, x0=[r_init, p_init], method="Nelder-Mead",
                      options={"maxiter": 10000})
    r_hat, p_hat = result.x
    return r_hat, r_hat * (1 - p_hat) / p_hat, result.success  # K, M, converged


all_households = df_trans["household_key"].unique()
nbd_records = []
for cat in TARGET_CATEGORIES:
    df_cat = df_trans[df_trans["COMMODITY_DESC"] == cat]
    counts = df_cat.groupby("household_key")["BASKET_ID"].count() \
                   .reindex(all_households, fill_value=0).values
    K, M, converged = fit_nbd(counts)
    nbd_records.append({"category": cat, "M": M, "K": K})

df_nbd = pd.DataFrame(nbd_records)

# --- ディリクレ多項分布フィッティング ---
def dm_neg_loglik(log_alpha, X, n):
    """Negative log-likelihood of Dirichlet-Multinomial distribution."""
    alpha = np.exp(log_alpha)
    S = alpha.sum()
    ll = len(n) * gammaln(S) - gammaln(n + S).sum()
    ll += (gammaln(X + alpha) - gammaln(alpha)).sum()
    return -ll


def fit_dm(df_cat):
    """Fit DM model for a category. Returns S_hat."""
    top5 = df_cat["MANUFACTURER"].value_counts().head(5).index.tolist()
    df_cat = df_cat.copy()
    df_cat["brand_group"] = df_cat["MANUFACTURER"].where(
        df_cat["MANUFACTURER"].isin(top5), other="Others"
    ).astype(str)

    hh_brand = (
        df_cat.groupby(["household_key", "brand_group"])["BASKET_ID"]
        .count().unstack(fill_value=0)
    )
    hh_brand = hh_brand[hh_brand.sum(axis=1) > 0]
    X = hh_brand.values.astype(float)
    n = X.sum(axis=1)
    J = X.shape[1]
    alpha_init = (X.sum(axis=0) / X.sum()) * J

    result = minimize(dm_neg_loglik, x0=np.log(alpha_init), args=(X, n),
                      method="Nelder-Mead",
                      options={"maxiter": 50000, "xatol": 1e-8, "fatol": 1e-8})
    return np.exp(result.x).sum()  # S_hat


dm_records = []
for cat in TARGET_CATEGORIES:
    S_hat = fit_dm(df_trans[df_trans["COMMODITY_DESC"] == cat])
    dm_records.append({"category": cat, "S": S_hat})

df_dm = pd.DataFrame(dm_records)

# --- K × S マッピング ---
df_map = df_nbd[["category", "K"]].merge(df_dm[["category", "S"]], on="category")
short_labels = {
    "SOFT DRINKS": "Soft Drinks", "FLUID MILK PRODUCTS": "Milk",
    "CHEESE": "Cheese", "BAG SNACKS": "Bag Snacks",
    "COLD CEREAL": "Cold Cereal", "YOGURT": "Yogurt",
    "FROZEN PIZZA": "Frozen Pizza", "SOUP": "Soup",
    "FRZN MEAT/MEAT DINNERS": "Frozen Meat", "BAKED BREAD/BUNS/ROLLS": "Bread/Rolls",
}
df_map["label"] = df_map["category"].map(short_labels)

k_med = df_map["K"].median()
s_med = df_map["S"].median()

fig, ax = plt.subplots(figsize=(9, 7))
ax.scatter(df_map["K"], df_map["S"], s=80, color="steelblue", zorder=3)
for _, row in df_map.iterrows():
    ax.annotate(row["label"], xy=(row["K"], row["S"]),
                xytext=(6, 4), textcoords="offset points", fontsize=10)

ax.axvline(k_med, color="gray", linestyle="--", linewidth=0.8, alpha=0.7)
ax.axhline(s_med, color="gray", linestyle="--", linewidth=0.8, alpha=0.7)

ax.text(df_map["K"].max() * 0.98, df_map["S"].max() * 0.98,
        "選好均質\n購入頻度均一", ha="right", va="top", fontsize=9, color="gray")
ax.text(df_map["K"].min() * 1.02, df_map["S"].max() * 0.98,
        "選好均質\nヘビー偏在", ha="left", va="top", fontsize=9, color="gray")
ax.text(df_map["K"].max() * 0.98, df_map["S"].min() * 1.02,
        "個人ロイヤルティ高\n購入頻度均一", ha="right", va="bottom", fontsize=9, color="gray")
ax.text(df_map["K"].min() * 1.02, df_map["S"].min() * 1.02,
        "個人ロイヤルティ高\nヘビー偏在", ha="left", va="bottom", fontsize=9, color="gray")

ax.set_xlabel("$K$（購入頻度の均一性）", fontsize=12)
ax.set_ylabel("$S$（消費者間の選好の均質性）", fontsize=12)
ax.set_title("K × S カテゴリマッピング", fontsize=14)
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig("article/fig_ks_map.png", dpi=150, bbox_inches="tight")
plt.show()
```

## 参考文献

- Dunnhumby. "The Complete Journey." Kaggle. https://www.kaggle.com/datasets/frtgnn/dunnhumby-the-complete-journey
- Ehrenberg, A. S. C. (1988). *Repeat Buying: Facts, Theory and Applications*. Griffin.
- Goodhardt, G. J., Ehrenberg, A. S. C., & Chatfield, C. (1984). The Dirichlet: A comprehensive model of buying behaviour. *Journal of the Royal Statistical Society. Series A*, 147(5), 621–655.
