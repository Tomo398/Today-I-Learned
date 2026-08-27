## Broadcasting, batching, and shapes(の3種類)
TFPにおいて重要なのは、Tensorのshapeを次の3種類にの「意味」に分けることでだ<br>
| 種類             | 意味                   |
| -------------- | -------------------- |
| `event_shape`  | 1回の確率事象の形            |
| `batch_shape`  | パラメータの異なる分布を何個並べているか |
| `sample_shape` | 各分布から何回ずつ無作為抽出するか    |
標本を生成したときのshapeは原則として、
sample_shape + batch_shape + event_shapeの順番となる<br>
ここでの+は足し算ではなく、shapeを左から連結するという意味<br>
例:
sample_shape = [5],batch_shape  = [2, 3],event_shape  = [4]<br>
なら、標本のshapeは[5, 2, 3, 4]

### event_shape
event_shapeはその分布における**1回の結果**がどのような形かを表す<br>
```python
dist = tfd.Normal(loc=[0.0, 10.0, 20.0],
scale=[1.0, 1.0, 1.0],
)
dist.sample()#値は複数(3つ)あるが、それぞれの正規分布はスカラー分布なのでevent_shapeは1


dist.event_shape# TensorShape([])
```
仮に下記のような多変量分布の場合、1回の標本から3次元ベクトルが得られる<br>
したがって、*event_shape*は3になり、3個の値をまとめたベクトル全体が1回の確率事象である

```python
vector_dist = tfd.MultivariateNormalDiag(
    loc=[0.0, 0.0, 0.0],
    scale_diag=[1.0, 1.0, 1.0],
)
```

### batch_shape
*batch_shape*はパラメータの異なる分布を何個まとめて保持しているかを表す<br>
上のコードだと、パラメータの異なる分布を3つ持っているので、batch_shapeは3

まとめて、比較すると下の表になる
| 分布            | `batch_shape` | `event_shape` | `sample()` |
| ------------- | ------------: | ------------: | ---------: |
| スカラーNormalを3個 |         `[3]` |          `[]` |      `[3]` |
| 3次元Normalを1個  |          `[]` |         `[3]` |      `[3]` |

shape単体で見ると同じだが、中身が異なる

### log_prob()の場合
スカラー分布を3個並べた場合は、
```python
scalar_batch.log_prob([0.1, 0.2, 0.3])
# shape: (3,)
[log p1(0.1),log p2(0.2),log p3(0.3),]
```
3個それぞれの対数確率密度が返る<br>
一方、3次元正規分布では、<br>
```python
vector_dist.log_prob([0.1, 0.2, 0.3])

# shape: ()
log p([0.1, 0.2, 0.3])
```
上記のように、3次元ベクトル全体に対する1個の対数確率密度が返る<br>
よって、valueのshapeはsample_shape + batch_shape + event_shapeだが、<br>
log_probのshapeは、sample_shape + batch_shapeのように<br>
event_shapeが消える(event全体に対して1個の対数確率を計算するから)<br>

### sample_shape
*sample_shape*は、それぞれの分布から**何回ずつ標本を生成**するかを表す<br>
これは分布オブジェクト自体の固定属性ではなく、sample()を呼び出すときに指定する<br>
```python
dist = tfd.Normal(loc=[0.0, 10.0, 20.0],scale=1.0,)
dist.sample_shape,dist.batch_shape,dist.event_shape #[],[3],[1]
sample = dist.sample(5)
dist.sample_shape,dist.batch_shape,dist.event_shape #[5],[3],[]
```
このように、行方向が標本抽出回数、列方向が異なる分布の数となる<br>

### 注意(バッチサイズとsample_sizeについて)
sample_sizeはkerasのbatch_sizeは別物である<br>
確かに*batch_size=32*の入力からnnetが32個の分布を予測した場合、TFP側ではこの32が**batch_shape(の一部)に現れる**<br>
一方、sample(100)の100は予測された各分布から100回ずつ乱数生成するという意味なので*sample_shape*という意味になる<br>

```python
#nnet48時点出力
params.shape# (batch_size, 48, 3)

zero_logits = params[..., 0:1]
concentration = (1e-5+ tf.nn.softplus(params[..., 1:2]))
rate = (1e-5+ tf.nn.softplus(params[..., 2:3]))
```
この各shapeは*(batch_size,48,1)*となる<br>
各位置にスカラーのゼロ過剰ガンマ分布を割り当てる場合<br>
batch_shape = [batch_size, 48, 1]、event_shape = []となる<br>

### _event_shape()と_event_shape_tensor()の違い
自作分布では、公式が*_event_shape()*と*_event_shape_tensor()*を要求する<br>
どちらも*event shape*を返すが、形式が異なる<br>

#### _event_shape()
静的なPython側の**shape情報**を返す<br>
```python
def _event_shape(self):
    return tf.TensorShape([])
```

#### _event_shape_tensor()
**TensorFlowの演算として使える**Tensorを返す<br>
```python
def _event_shape_tensor(self):
    return tf.constant([],dtype=tf.int32,)
```
こちらの返り値は整数Tensorとなる

### Broadcasting
そもそも、*Broadcasting*とは、shapeの異なるTensor同士を自動的に同じshapeとして計算する仕組みである<br>

TFPでは、分布パラメータをBroadcastingした結果が、基本的に分布のbatch_shapeとなる<br>

例は以下のコードとなる
```python
dist = tfd.Normal(loc=[0.0, 10.0, 20.0],scale=1.0,)

loc.shape# (3,)

scale.shape# ()
```
このようにscaleはスカラーだが、Broadcasting(配列自動補完)により、*scale = [1.0, 1.0, 1.0]*と拡張される

### Broadcastingの基本ルール
2つのshapeを右端に揃えて(寄せて)、各次元を比較する<br>
各次元について、次のどれかを満たすならBroadcasting可能となる<br>
1.次元の大きさが同じ<br>
2.どちらかが1<br>
3.片方にその次元が存在しない<br>
以下に例を挙げる<br>
```python
loc = tf.constant([0.0, 10.0, 20.0,])# shape: [3]

scale = tf.constant([[1.0],[5.0],])# shape: [2, 1]

dist = tfd.Normal(loc=loc,scale=scale,)
```
この場合、shapeを右端に揃える(寄せる)と
```python
loc:   [1, 3]
scale: [2, 1]
```
になる<br>
各次元について、1と2→2、3と1→3なので、
```python
dist.batch_shape
# TensorShape([2, 3])
```
これは6個の正規分布を表す<br>

### 複数の値で全分布を評価する場合
次の3個の正規分布を考える
```python
dist = tfd.Normal(loc=[0.0, 10.0, 20.0],scale=1.0,)
values = tf.constant([2.0,-1.0,0.0,1.0,2.0,])# shape: [5]
```
上記の5個のスカラー値で3個全ての分布を評価したいとする<br>
この場合*dist.log_prob(values)*は上手くいかない<br>
なぜなら、分布と観測値のshapeは3と5で合わないから<br>

そこで、各観測地で全ての分布で評価したい場合は、
```python
values = values[..., tf.newaxis]
values.shape# (5, 1)
```
として次元を追加する<br>
そうすると、
```python
log_probs = dist.log_prob(values)

log_probs.shape# (5, 3)
```

よって、params[..., 0:1]、params[..., 1:2]、params[..., 2:3]<br>
のように直接インデックスを指定するのではなくスライス記法で書くことで最後の軸がそのまま維持されて*broadcasting*しやすくなる<br>

#### インデックスとスライスの違い
*params[..., 0:1]*だと軸が残り、*params[..., 0]*だと軸が消える理由を考える<br>
例として、2次元のTensorを考える<br>
```python
x = tf.constant([[10, 11, 12],
                [20, 21, 22],])

print(x[1, 2])
# 22
```
この場合、Tensorから1個の値を指定するには2つの番号が必要となる<br>
*x[行番号, 列番号]*である<br>
言い換えると、2次元であるとは、値を特定するために2つのインデックスが必要<br>
よって、*x[:, 0]*を実効すると0番目の列のみ使われるので、このshapeは*(2,)*となる<br>

一方、スライスは軸を固定していない<br>
*x[:, 0:1]*は、全ての行について0列目から1列目の手前までの**列の集合**を取り出すという意味である<br>
よって、結果は*[[10],[20]]*になる<br>
列数は1個であるものの、列という方向自体は残りshapeは*(2,1)*になる<br>

