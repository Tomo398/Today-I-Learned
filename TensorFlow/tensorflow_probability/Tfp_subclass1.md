## TensorFlow Probabilityのサブクラス作成
### そもそもサブクラスとは
**既存の親クラス**を継承して作るクラスである

### 一般的な(generic)ベースとなる分布クラス
```python
tfp.distributions.Distribution(
    dtype,
    reparameterization_type,
    validate_args,
    allow_nan_stats,
    parameters=None,
    graph_parents=None,
    name=None
)
```
基本的に*dtype*は*tf.float32*であり*reparameterization_type*はサンプリングを通じてパラメータへ微分できるかであり、<br>
*validate_args*はパラメータが有効か検査するかであり*allow_nan_stats*は未定義の平均/
・分散などをNanとして返すかであり、<br>
*parameters*は分布を再構築するためのコンストラクタ引数であり、<br>
*graph_parents*は古いTensorFlowグラフ用の依存Tensor(通常指定しない)であり、*name*はTensorFlow内部での分布名を指す<br>

※コンストラクタとは、クラスからオブジェクトを作るときに呼ばれる処理を**コンストラクタ**という
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

person = Person(name="Tom",age=25,)
#このときの"Tom"と25がコンストラクタ引数である
```
よって、ここの*parameters*の意味としてはこの分布をどのコンストラクタ引数から作成したかを記録した辞書であり、<br>
copy()などで分布を再構築するときに使われる<br>

### 基本パラメータの詳細説明
1. *reparameterization_type*はsample()で生成した乱数を経由して、分布パラメータまで勾配を流せるかについて<br>
sample()とは、確率分布そのものを表すオブジェクト(dist)からランダムに1個生成する
```python
dist = tfd.Normal(loc=0.0,scale=1.0,)
dist.sample()
```
ニューラルネットワークによっては、予測した分布からsample()で値を作り、その値を使って損失を計算する場合がある
```marmaid
flowchart TD
    P["分布パラメータ θ"] --> D["確率分布 pθ"]
    D --> S["sample()で乱数 X を生成"]
    S --> L["Xから損失を計算"]
    L -. "逆伝播" .-> P
```
基本的には負の対数尤度を損失とするので考慮する必要なないが、分布から生成される将来の結果に対する<br>
**平均的な損失**を最小化したいときである<br>
例:CRPSを損失として使うとき<br>
このCRPSは『観測値$x$に対する累積分布関数$A(x)$』と『予測された累積分布$F(x)$』を比較評価する指標である
$$CRPS = \int_{-∞}^{∞} [F_i(x) - A_i(x)]^2 dx$$

2. *validate_args*は分布に渡されたパラメータや、*log_prob()*に渡された値が<br>
その**分布の数学的な条件を満たしているか**検査する設定<br>
例:指数分布の$\lambda$は0より大きい必要があるが、<br>
それを満たしていないと*inf*といったエラーが出る可能性がある<br>
よって、*True*にした方がいい<br>
※Trueにした場合、自分で検査条件そのものも実装する必要がある

3. *allow_nan_stats*は分布は数学的に有効だが、求めとようとした<br>
平均・分散・最頻値など定義できない場合NaNを返すことを許すかを決める設定<br>
statsはそもそも分布の統計量であり、以下のメソッドが該当する<br>
```python
dist.mean()       # 平均
dist.variance()   # 分散
dist.stddev()     # 標準偏差
dist.mode()       # 最頻値
dist.covariance() # 共分散
dist.entropy()    # エントロピー
```
例えば、コーシー分布は平均が定義できないが、エラーではなくNaNで返すように設定できる<br>
こちらも、Trueにする場合は、自分でどうするか実装する必要がある

### Subclassing
サブクラスでは、同じ名前の関数にアンダースコアをつけたものにすることが望まれている<br>
基本的に引数の構成は同一にすべきだ(*name=*を除く)<br>
例えば、*log_prob(value,name="log_prob")*を有効化するには、<br>
サブクラスで*_log_prob(value)*という風に満たす必要がある<br>

| 公開側                | 自作クラス側                         |
| ------------------ | ------------------------------ |
| `dist.log_prob(y)` | `def _log_prob(self, y)`       |
| `dist.sample(5)`   | `def _sample_n(self, n, seed)` |
| `dist.mean()`      | `def _mean(self)`              |
| `dist.variance()`  | `def _variance(self)`          |
| `dist.cdf(y)`      | `def _cdf(self, y)`            |


```python
@distribution_util.AppendDocstring('Some other details.')
def _log_prob(self, value):
  ...
```
この"some other details."という一文を埋め込むべきだ<br>
TFPメソッドは一般的に、分布サブクラスには少なくとも下記のメソッドを与えることを想定している<br>
```python
_sample_n.
_log_prob or _prob.
_event_shape and _event_shape_tensor.
_parameter_properties OR _batch_shape and _batch_shape_tensor.
```
Batch shapeメソッドは多くの場合自動的に*parameter_properties*から得られる<br>
よって、それを直接使うことは大抵の場合必要ではない<br>
例外は、非Tensorパラメータを受け入れる場合や非標準のバッチセマンティクスを持つ分布などがある(*BatchReshape*など)<br>

いくつかの機能性は追加のメソッドを使うことに依存しているだろう<br>
下記のものが、分布サブクラスを実行するための共通なものだ<br>
```python
Relevant statistics, such as _mean, _mode, _variance and/or _stddev.
At least one of _log_cdf, _cdf, _survival_function, or _log_survival_function.
_quantile.
_entropy.
_default_event_space_bijector.
_parameter_properties (to support automatic batch shape derivation, batch slicing and other features).
_sample_and_log_prob,
_maximum_likelihood_parameters
```
注意として、存在する分布を*__init__*で再定義するようなサブクラスは、<br>
その親クラス*_parameter_properties*のコメントは自動的には継承しないことが挙げられる<br>
よって、サブクラスでは、その特徴を確認するために明示的にその*_parameter_properties*メソッドを与える必要がある<br>
