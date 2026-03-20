# ディリクレ多項分布で読み解くブランド選択の構造

## はじめに

前回記事にて、NBD モデルで「カテゴリをどれだけ頻繁に買うか」を記述した。次の問いは「ではどのブランドを選ぶか、そして同じブランドを繰り返し選ぶのか」である。この「ブランド選択」の構造を定量化するのがディリクレ多項分布モデルである。

## データ

前記事と同じく Dunnhumby "The Complete Journey" の SOFT DRINKS カテゴリ（52週間、2,272世帯、6ブランドグループ）を使用する。

## 手法：ディリクレ多項分布モデル

### モデル

消費者 $i$ が $n_i$ 回の購買機会を持ち、各機会でブランドを独立に選ぶとする。各消費者はブランド選好ベクトル $\boldsymbol{\theta}_i = (\theta_1, \ldots, \theta_J)$（$\sum_j \theta_j = 1$）を持ち、その消費者間異質性をディリクレ分布で表現する。

**ディリクレ分布**の確率密度関数は次のとおりである。

$$
p(\boldsymbol{\theta} \mid \boldsymbol{\alpha}) = \frac{1}{B(\boldsymbol{\alpha})} \prod_{j=1}^{J} \theta_j^{\alpha_j - 1}, \qquad \boldsymbol{\theta} \in \Delta^{J-1}
$$

ここで $B(\boldsymbol{\alpha})$ は多変量ベータ関数である。

各消費者の購入回数ベクトル $\mathbf{x}_i = (x_{i1}, \ldots, x_{iJ})$ は、選好 $\boldsymbol{\theta}_i$ が与えられたもとで多項分布に従う。

$$
\boldsymbol{\theta}_i \sim \mathrm{Dirichlet}(\boldsymbol{\alpha}), \qquad \mathbf{x}_i \mid \boldsymbol{\theta}_i, n_i \sim \mathrm{Multinomial}(n_i, \boldsymbol{\theta}_i)
$$

$\boldsymbol{\theta}_i$ を積分消去すると、$\mathbf{x}_i$ の周辺分布としてディリクレ多項分布が得られる。

$$
P(\mathbf{x}_i \mid n_i, \boldsymbol{\alpha})
= \binom{n_i}{\mathbf{x}_i} \frac{B(\mathbf{x}_i + \boldsymbol{\alpha})}{B(\boldsymbol{\alpha})}
$$

ベータ関数をガンマ関数で展開すると：

$$
P(\mathbf{x}_i \mid n_i, \boldsymbol{\alpha})
= \binom{n_i}{\mathbf{x}_i} \frac{\Gamma(S)}{\Gamma(n_i + S)} \prod_{j=1}^{J} \frac{\Gamma(x_{ij} + \alpha_j)}{\Gamma(\alpha_j)}
$$

### パラメータの意味

| パラメータ            | 意味                                                                |
| --------------------- | ------------------------------------------------------------------- |
| $\alpha_j$            | ブランド $j$ への選好の強さ                                         |
| $S = \sum_j \alpha_j$ | **集中度パラメータ**：消費者間の選好の均質性を表す1つの数値（小さいほど個人ロイヤルティが高い） |
| $\alpha_j / S$        | ブランド $j$ の期待市場シェア                                       |

$S$ が小さいほど消費者間の選好の異質性が大きく（各消費者が特定ブランドに偏る）、個人レベルのブランドロイヤルティが高い。$S$ が大きいほど消費者間が均質になり、繰り返し購買確率はシェアに近づく。

## 結果

MLE でパラメータを推定した結果を以下に示す。対数尤度は $\ell = -12{,}847$、AIC $= 25{,}706$（パラメータ数 6）。

$$S = 4.13$$

モデルシェア $\alpha_j / S$ は「ある消費者がランダムに1回購買するときにブランド $j$ を選ぶ確率」の集団平均であり、理論的には延べ購買数ベースのシェアと一致する（$E[\text{市場シェア}_j] = \alpha_j / S$）。実測シェアは全消費者の延べ購買数を集計した値であり、両者は同じ量を異なる方法で求めたものである。

| ブランド | $\alpha_j$ | シェア（モデル） | シェア（延べ購買実測） |
| -------- | ---------- | ---------------- | ---------------------- |
| Mfr_103  | 1.210      | 29.3%            | 31.5%                  |
| Mfr_1208 | 1.160      | 28.1%            | 26.7%                  |
| Mfr_2224 | 0.807      | 19.5%            | 16.7%                  |
| Mfr_69   | 0.524      | 12.7%            | 17.8%                  |
| Others   | 0.328      | 7.9%             | 5.9%                   |
| Mfr_2    | 0.106      | 2.6%             | 1.4%                   |

### S パラメータの解釈：ロイヤルティのリフト

#### 繰り返し購買確率の導出

ディリクレ多項分布は**交換可能列**（exchangeable sequence）を構成するため、購買の順序に関わらず同時確率が変わらない（de Finetti の定理）。この性質を利用して繰り返し購買確率を導出する。

2回の購買がともにブランド $j$ である同時確率は、消費者の選好 $\theta_j$ で条件付けて期待値を取ると：

$$
P(\text{1回目} = j,\ \text{2回目} = j)
= \mathbb{E}_{\theta_j}\bigl[P(\text{1回目}=j \mid \theta_j)\cdot P(\text{2回目}=j \mid \theta_j)\bigr]
= \mathbb{E}[\theta_j \cdot \theta_j]
= \mathbb{E}[\theta_j^2]
$$

ここで2つ目の等号は、同じ $\theta_j$ を持つ消費者の各購買が条件付き独立であること（交換可能性の帰結）を使った。

ディリクレ分布の平均と分散は以下のとおりである。

$$
\mathbb{E}[\theta_j] = \frac{\alpha_j}{S}, \qquad
\mathrm{Var}(\theta_j) = \frac{\alpha_j(S - \alpha_j)}{S^2(S + 1)}
$$

$\mathbb{E}[\theta_j^2] = \mathrm{Var}(\theta_j) + \mathbb{E}[\theta_j]^2$ に代入すると：

$$
\mathbb{E}[\theta_j^2]
= \frac{\alpha_j(S - \alpha_j)}{S^2(S+1)} + \frac{\alpha_j^2}{S^2}
= \frac{\alpha_j(S - \alpha_j) + \alpha_j^2(S+1)}{S^2(S+1)}
= \frac{\alpha_j S - \alpha_j^2 + \alpha_j^2 S + \alpha_j^2}{S^2(S+1)}
= \frac{\alpha_j S(1 + \alpha_j)}{S^2(S+1)}
= \frac{\alpha_j(\alpha_j + 1)}{S(S + 1)}
$$

したがって：

$$
P(\text{1回目} = j,\ \text{2回目} = j) = \frac{\alpha_j(\alpha_j + 1)}{S(S + 1)}
$$

1回目がブランド $j$ である確率は $P(\text{1回目} = j) = \mathbb{E}[\theta_j] = \alpha_j / S$ なので、条件付き確率の定義から：

$$
P(\text{次の購買} = j \mid \text{前の購買} = j)
= \frac{\mathbb{E}[\theta_j^2]}{\mathbb{E}[\theta_j]}
= \frac{\alpha_j(\alpha_j + 1)}{S(S + 1)} \cdot \frac{S}{\alpha_j}
= \frac{\alpha_j + 1}{S + 1}
$$

**直感的な理解（Pólya の壺）**: 壺の中にブランド $j$ の玉が $\alpha_j$ 個、全体で $S$ 個入っている。1個取り出したら**同じ色の玉を2個戻す**。前回 $j$ を選んだ直後は $j$ の玉が $\alpha_j + 1$ 個、全体が $S + 1$ 個になるため、次に $j$ を引く確率が $(\alpha_j + 1)/(S + 1)$ となる。

これとランダムに選んだ場合のシェア $\alpha_j / S$ の比を「ロイヤルティのリフト」と呼ぶ。

| ブランド | シェア | 繰り返し購買確率 | リフト |
| -------- | ------ | ---------------- | ------ |
| Mfr_103  | 29.3%  | 43.0%            | 1.47×  |
| Mfr_1208 | 28.1%  | 42.1%            | 1.50×  |
| Mfr_2224 | 19.5%  | 35.2%            | 1.80×  |
| Mfr_69   | 12.7%  | 29.7%            | 2.34×  |
| Others   | 7.9%   | 25.9%            | 3.26×  |
| Mfr_2    | 2.6%   | 21.5%            | 8.41×  |

式から自明だが、シェアが低いブランドほどリフトが大きくなっている。例えば Mfr_2 は市場シェアは 2.6% にすぎないが、繰り返し購買確率は 21.5%——シェアの 8.4 倍——である。これは $(\alpha_j+1)/(S+1)$ と $\alpha_j/S$ の差が $\alpha_j$ が小さいほど相対的に大きくなる式の性質による。

これに対し **Mfr_103・Mfr_1208** はシェアが大きいためリフトは 1.5 倍程度にとどまり、繰り返し購買確率がシェアに近い——他ブランドからの流入も多く、購買のたびに固定した好みではなくシェアに比例した確率でブランドが選ばれる状態に近い。

### スイッチング行列

スイッチング行列の要素 $(j, k)$ は「前回 $j$ を買った消費者が次回 $k$ を選ぶ確率」を表す（行 = 前回購買ブランド、列 = 次回購買ブランド）。

|          | Mfr_103   | Mfr_1208  | Mfr_2224  | Mfr_69    | Others    | Mfr_2     |
| -------- | --------- | --------- | --------- | --------- | --------- | --------- |
| Mfr_103  | **0.430** | 0.226     | 0.157     | 0.102     | 0.064     | 0.021     |
| Mfr_1208 | 0.236     | **0.421** | 0.157     | 0.102     | 0.064     | 0.021     |
| Mfr_2224 | 0.236     | 0.226     | **0.352** | 0.102     | 0.064     | 0.021     |
| Mfr_69   | 0.236     | 0.226     | 0.157     | **0.297** | 0.064     | 0.021     |
| Others   | 0.236     | 0.226     | 0.157     | 0.102     | **0.259** | 0.021     |
| Mfr_2    | 0.236     | 0.226     | 0.157     | 0.102     | 0.064     | **0.215** |

対角成分（太字）が繰り返し購買確率、非対角成分がスイッチング先の確率である。

注目すべきは非対角成分のパターンである。どのブランドからのスイッチングであっても、流入先は市場シェアに比例して Mfr_103・Mfr_1208 に集中する。これはディリクレ分布の特性（exchangeability）であり、ブランド間にペアワイズな特別な親和性がないことを前提としている。「Mfr_A から Mfr_B への移行」と「Mfr_C から Mfr_B への移行」は同じ確率で起きる。

## 考察

### なぜ単純なリピート率ではなくモデルを使うのか

ブランドごとのリピート率は、購買データから直接計算することもできる。

しかし単純集計には問題がある。購買回数が2回の消費者は「2回とも同じブランド → 100%」か「それぞれ別のブランド → 0%」の二択しかなく、個人レベルの推定値は極めて不安定である。また、消費者ごとに独立に計算しても、「このカテゴリはロイヤルティが高い市場か」という問いには答えにくい。

ディリクレ多項分布モデルはこの問題を以下の方法で解決する。個人ごとにリピート率を計算するのではなく、全消費者のブランド選択行列 $X$（世帯 × ブランドの購買回数）を一括して $\boldsymbol{\alpha}$ に集約する。


購買回数が2〜3回しかない消費者も、どのブランドを選んだかという情報を通じて $\boldsymbol{\alpha}$ の推定に貢献する。1人の少ない観測値が全体の推定値を大きく動かすことはなく、数千世帯のデータで安定した推定が得られる。

また、このモデルからは $S$ という1つの数値で消費者間の選好の均質性を要約でき（$S$ が小さいほど個人ロイヤルティが高い）、パラメータが推定できれば「次の購買がどうなるか」のシミュレーションも可能になる。


## ソースコード

```python
import numpy as np
from scipy.special import gammaln
from scipy.optimize import minimize


def dm_neg_loglik(log_alpha, X, n):
    """Negative log-likelihood of Dirichlet-Multinomial distribution."""
    alpha = np.exp(log_alpha)
    S = alpha.sum()
    N = len(n)
    ll = N * gammaln(S) - gammaln(n + S).sum()
    ll += (gammaln(X + alpha) - gammaln(alpha)).sum()
    return -ll


# X: shape (N, J)  世帯 × ブランドの購買回数行列
# n: shape (N,)    各世帯の合計購買回数
J = X.shape[1]
share_init = X.sum(axis=0) / X.sum()
alpha_init = share_init * J

result_dm = minimize(
    dm_neg_loglik,
    x0=np.log(alpha_init),
    args=(X, n),
    method="Nelder-Mead",
    options={"maxiter": 50000, "xatol": 1e-8, "fatol": 1e-8},
)

alpha_hat = np.exp(result_dm.x)
S_hat = alpha_hat.sum()
```

## 参考文献

- Dunnhumby. "The Complete Journey." Kaggle. https://www.kaggle.com/datasets/frtgnn/dunnhumby-the-complete-journey
