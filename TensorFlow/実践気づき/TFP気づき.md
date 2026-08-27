## TFPと点推定モデルの差異
### 基本的な差異
そもそも予測するのは各時点における確率分布(横軸:目的変数(y)が取りうる値、縦軸:確率密度)となる<br>
この際、予測する対象に合わせて確率分布を指定することが肝となる

### 実装差異
基本的に変える点は3つである<br>
1.損失関数<br>
これは、須らく**負の対数尤度**にしなければならない<br>
```python
def negative_log_likelihood(y_true, y_dist):
    return -y_dist.log_prob(y_true)
```
2.出力層<br>
出力層を(指定した)分布のパラメータにする必要がある<br>
例えば、正規分布であれば平均と分散の二つ
```python
params = layers.Dense(2)(x)
```
3.出力<br>
出力自体を確率分布で出す必要がある<br>
その際、**tfp.layers.DistributionLambda()**を使い、どのような分布でどのようなパラメータがいるのか指定する必要がある<br>
```python
outputs = tfpl.DistributionLambda(lambda params: tfd.Normal(loc=params[...,0:1],scale=1e-3 + tf.nn.softplus(params[...,1:2])))(params)
```

### より詳細な書き方
出力の書き方が無名関数をつかったり、独自の関数を使う必要があったりとやや煩雑ではある<br>
そこで、種類を纏めようと思う<br>

## 1. 実数全体の連続分布

| 分布 | 支持範囲 | 主要引数 | 制約 | 最小構文 | 主な用途・注意点 |
|---|---:|---|---|---|---|
| `tfd.Normal` | `(-∞, ∞)` | `loc`, `scale` | `scale > 0` | `tfd.Normal(loc=mu, scale=pos(raw_scale))` | 標準的な回帰。`loc`は平均、`scale`は標準偏差 |
| `tfd.StudentT` | `(-∞, ∞)` | `df`, `loc`, `scale` | `df > 0`, `scale > 0` | `tfd.StudentT(df=2.0 + pos(raw_df), loc=mu, scale=pos(raw_scale))` | Normalより外れ値に強い。平均には`df > 1`、有限分散には`df > 2`が必要 |
| `tfd.Laplace` | `(-∞, ∞)` | `loc`, `scale` | `scale > 0` | `tfd.Laplace(loc=mu, scale=pos(raw_scale))` | 中心が尖り、Normalより裾が厚い。固定scaleのNLLはMAEと対応 |
| `tfd.Logistic` | `(-∞, ∞)` | `loc`, `scale` | `scale > 0` | `tfd.Logistic(loc=loc, scale=pos(raw_scale))` | Normalより裾が厚い対称分布 |
| `tfd.Cauchy` | `(-∞, ∞)` | `loc`, `scale` | `scale > 0` | `tfd.Cauchy(loc=loc, scale=pos(raw_scale))` | 極端に裾が厚い。平均・分散が存在しないため、平均予測には不向き |
| `tfd.GeneralizedNormal` | `(-∞, ∞)` | `loc`, `scale`, `power` | `scale > 0`, `power > 0` | `tfd.GeneralizedNormal(loc=loc, scale=pos(raw_scale), power=pos(raw_power))` | `power`で尖度を調整。`power=2`でNormal型、`power=1`でLaplace型 |
| `tfd.Gumbel` | `(-∞, ∞)` | `loc`, `scale` | `scale > 0` | `tfd.Gumbel(loc=loc, scale=pos(raw_scale))` | 非対称な極値分布。最大値・最小値などに利用 |
| `tfd.TwoPieceNormal` | `(-∞, ∞)` | `loc`, `scale`, `skewness` | `scale > 0`, `skewness > 0` | `tfd.TwoPieceNormal(loc=loc, scale=pos(raw_scale), skewness=pos(raw_skew))` | 左右で幅の異なる非対称正規型。`skewness=1`でNormal |

## 2. 正の連続分布

| 分布 | 支持範囲 | 主要引数 | 制約 | 最小構文 | 主な用途・注意点 |
|---|---:|---|---|---|---|
| `tfd.Gamma` | `(0, ∞)` | `concentration`, `rate` または `log_rate` | `concentration > 0`, `rate > 0` | `tfd.Gamma(concentration=pos(raw_c), rate=pos(raw_rate))` | 右に歪んだ正値。平均は `concentration / rate`。TFPはrate表現 |
| `tfd.LogNormal` | `(0, ∞)` | `loc`, `scale` | `scale > 0` | `tfd.LogNormal(loc=log_loc, scale=pos(raw_scale))` | `loc`, `scale`は`log(Y)`のNormalパラメータであり、元のYの平均・標準偏差ではない |
| `tfd.Exponential` | `[0, ∞)` | `rate` | `rate > 0` | `tfd.Exponential(rate=pos(raw_rate))` | 1パラメータの右裾分布。平均は `1 / rate` |
| `tfd.Weibull` | `[0, ∞)` | `concentration`, `scale` | 両方 `> 0` | `tfd.Weibull(concentration=pos(raw_c), scale=pos(raw_scale))` | 寿命・故障時間・風速など。Exponentialを特殊例として含む |
| `tfd.InverseGamma` | `(0, ∞)` | `concentration`, `scale` | 両方 `> 0` | `tfd.InverseGamma(concentration=pos(raw_c), scale=pos(raw_scale))` | 右裾が重い。分散パラメータの事前分布としても使われる |
| `tfd.HalfNormal` | `[0, ∞)` | `scale` | `scale > 0` | `tfd.HalfNormal(scale=pos(raw_scale))` | 0を中心とするNormalの正側。位置パラメータはない |
| `tfd.HalfStudentT` | `[loc, ∞)` | `df`, `loc`, `scale` | `df > 0`, `scale > 0` | `tfd.HalfStudentT(df=pos(raw_df), loc=loc, scale=pos(raw_scale))` | Student-tの右半分。正値かつ裾の厚い量 |
| `tfd.Pareto` | `[scale, ∞)` | `concentration`, `scale` | 両方 `> 0` | `tfd.Pareto(concentration=pos(raw_c), scale=pos(raw_scale))` | 非常に重い右裾。極端値や上位尾部 |
| `tfd.LogLogistic` | `(0, ∞)` | `loc`, `scale` | `scale > 0` | `tfd.LogLogistic(loc=loc, scale=pos(raw_scale))` | 対数値がLogisticに従う裾の厚い正値分布 |

## 3. 0から1・有限区間の連続分布

| 分布 | 支持範囲 | 主要引数 | 制約 | 最小構文 | 主な用途・注意点 |
|---|---:|---|---|---|---|
| `tfd.Beta` | `(0, 1)` | `concentration1`, `concentration0` | 両方 `> 0` | `tfd.Beta(concentration1=pos(raw_a), concentration0=pos(raw_b))` | 割合・比率。式上では、[0,1]だが実装上は(0,1)、正確な0と1を通常のBetaにそのまま入れてはいけない |
| `tfd.Kumaraswamy` | `(0, 1)` | `concentration1`, `concentration0` | 両方 `> 0` | `tfd.Kumaraswamy(concentration1=pos(raw_a), concentration0=pos(raw_b))` | Betaに似るが、再パラメータ化・CDF・分位点が比較的扱いやすい |
| `tfd.LogitNormal` | `(0, 1)` | `loc`, `scale` | `scale > 0` | `tfd.LogitNormal(loc=loc, scale=pos(raw_scale))` | `logit(Y)`がNormal。`loc`はYそのものの平均ではない |
| `tfd.TruncatedNormal` | `[low, high]` | `loc`, `scale`, `low`, `high` | `scale > 0`, `low < high` | `tfd.TruncatedNormal(loc=loc, scale=pos(raw_scale), low=0.0, high=1.0)` | Normalを区間で切断して再正規化。切断後の平均は`loc`と一致しない |

## 4. 離散分布

| 分布 | 支持範囲 | 主要引数 | 制約 | 最小構文 | 主な用途・注意点 |
|---|---:|---|---|---|---|
| `tfd.Bernoulli` | `{0, 1}` | `probs` または `logits` | 片方だけ指定 | `tfd.Bernoulli(logits=logits)` | 二値事象。NNでは通常logitsを直接出力 |
| `tfd.Categorical` | `{0, ..., K-1}` | `probs` または `logits` | 最終軸がKクラス | `tfd.Categorical(logits=logits)` | 整数ラベルの多クラス分類 |
| `tfd.OneHotCategorical` | K次元one-hot | `probs` または `logits` | 最終軸がKクラス | `tfd.OneHotCategorical(logits=logits)` | one-hot形式の多クラス分類 |
| `tfd.Binomial` | `{0, ..., total_count}` | `total_count`, `probs` または `logits` | `total_count >= 0`; 確率表現は片方のみ | `tfd.Binomial(total_count=n, logits=logits)` | n回試行中の成功回数 |
| `tfd.Poisson` | `{0, 1, 2, ...}` | `rate` または `log_rate` | `rate > 0`; 片方だけ指定 | `tfd.Poisson(rate=pos(raw_rate))` | カウントデータ。平均と分散がともにrate |
| `tfd.NegativeBinomial` | `{0, 1, 2, ...}` | `total_count`, `probs` または `logits` | `total_count > 0`; 確率表現は片方のみ | `tfd.NegativeBinomial(total_count=pos(raw_count), logits=logits)` | 分散が平均より大きい過分散カウント |
| `tfd.Geometric` | `{0, 1, 2, ...}` | `probs` または `logits` | 片方だけ指定 | `tfd.Geometric(logits=logits)` | 最初の成功までの失敗回数 |
| `tfd.Multinomial` | 合計が`total_count`のK次元カウント | `total_count`, `probs` または `logits` | 最終軸がKカテゴリ | `tfd.Multinomial(total_count=n, logits=logits)` | 複数カテゴリの回数ベクトル |

## 5. 多変量分布

| 分布 | 支持範囲 | 主要引数 | 制約 | 最小構文 | 主な用途・注意点 |
|---|---:|---|---|---|---|
| `tfd.MultivariateNormalDiag` | `R^D` | `loc`, `scale_diag` | `scale_diag`の各要素を正にする | `tfd.MultivariateNormalDiag(loc=loc, scale_diag=pos(raw_scale))` | D次元出力。各次元の分散は異なるが共分散は0 |
| `tfd.MultivariateNormalTriL` | `R^D` | `loc`, `scale_tril` | `scale_tril`は正の対角を持つ下三角行列 | `tfd.MultivariateNormalTriL(loc=loc, scale_tril=scale_tril)` | 出力間の相関も予測。パラメータ数はD(D+1)/2 |

48時点を独立なNormalの集合ではなく1つの確率ベクトルとして扱う例：

```python
n_outputs = 48
params = layers.Dense(2 * n_outputs)(x)

dist = tfd.MultivariateNormalDiag(
    loc=params[..., :n_outputs],
    scale_diag=pos(params[..., n_outputs:])
)
```

## 6. 混合分布

| 分布 | 主要引数 | 最小構文 | 主な用途・注意点 |
|---|---|---|---|
| `tfd.MixtureSameFamily` | `mixture_distribution`, `components_distribution`, `reparameterize=False` | `tfd.MixtureSameFamily(tfd.Categorical(logits=mix_logits), tfd.Normal(loc=locs, scale=scales))` | 同じ種類の成分を混合。多峰性予測に向く |

K成分の混合正規分布では、混合logits・各成分のloc・各成分のscaleの合計`3K`個を出力する。

```python
k = 3
params = layers.Dense(3 * k)(x)

dist = tfd.MixtureSameFamily(
    mixture_distribution=tfd.Categorical(
        logits=params[..., :k]
    ),
    components_distribution=tfd.Normal(
        loc=params[..., k:2 * k],
        scale=pos(params[..., 2 * k:3 * k])
    )
)
```

## 7. `DistributionLambda`での共通実装

```python
params = layers.Dense(2)(x)

outputs = tfpl.DistributionLambda(
    make_distribution_fn=lambda p: tfd.Normal(loc=p[..., 0:1],scale=pos(p[..., 1:2])),
    convert_to_tensor_fn=tfd.Distribution.mean
)(params)
```
p[...,0:1]の意味だが、0:1は0番目の出力、つまり、平均を指定しており、...はその他の次元を纏めて維持する記法である<br>
また、いずれかのパラメータが必ず正であるとき*1e-3 + tf.nn.softplus(params[...,1:2])*と書くことで、無理やり正の値にして学習を安定させる<br>

```python
def negative_log_likelihood(y_true, y_dist):
    return -y_dist.log_prob(y_true)
```

```python
model.compile(
    optimizer="adam",
    loss=negative_log_likelihood
)
```

## 8. 分布選択の早見表

| 目的変数 | 第一候補 | 比較候補 | 注意点 |
|---|---|---|---|
| 制約のない連続値 | `Normal` | `StudentT`, `Laplace`, `GeneralizedNormal` | 外れ値が多いならStudentT |
| 正の連続値 | `Gamma` | `LogNormal`, `Weibull` | 0を含む場合は支持範囲を確認 |
| 0から1の割合 | `Beta` | `Kumaraswamy`, `LogitNormal` | 正確な0・1を含むなら通常のBetaは不可 |
| 0から1に制約された連続値 | `TruncatedNormal` | ゼロ・ワン膨張Betaなど | `loc`は切断後平均ではない |
| 0/1 | `Bernoulli` | - | logitsを推奨 |
| クラス番号 | `Categorical` | - | yは整数ラベル |
| カウント | `Poisson` | `NegativeBinomial` | 過分散ならNegativeBinomial |
| 複数連続値 | `MultivariateNormalDiag` | `MultivariateNormalTriL` | 相関を扱うかで選択 |
| 多峰性 | `MixtureSameFamily` | - | 学習が不安定になりやすい |

## 重要な落とし穴

| 落とし穴 | 内容 |
|---|---|
| `scale`を標準偏差と思い込む | Normalでは標準偏差だが、StudentT・LogNormal・Gammaなどでは解釈が異なる |
| `loc`を平均と思い込む | Normalでは平均だが、LogNormal・LogitNormal・TruncatedNormal・Cauchyなどでは異なる、または平均が存在しない |
| 正の制約を忘れる | `scale`, `rate`, `concentration`, `df`などに生のDense出力をそのまま入れない |
| `probs`と`logits`を両方指定する | どちらか一方だけを指定する |
| 支持範囲とyが合っていない | Betaに0・1、Gammaに0以下、Poissonに非整数を入れない |
| 予測範囲をclipして評価する | 分布の不適合を隠すため、まず未加工で評価する |