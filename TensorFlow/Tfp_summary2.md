## TensorFlow Probability

### 概要
### 分布
*tfp(.distributions).xxx*を引数として分布を持ってくる

#### 単純な分布

#### 正規分布
```python
normal = tfd.Normal(loc=0., scale=1.)
print(normal)
```
*loc*が分布を左右に動かす位置パラメータ(今回だと平均)、*scale*が分布の広がりを変えるスケールパラメータ(今回だと標準偏差)にあたる

### 分布と形状
TensorFLowのTensorsには形状がある。普通のTensorのshapeなら、n行x列のデータと解釈できる。<br>
しかし、Tensorflow Probabilityでは同じ(3,2)でも<br>
3個の分布があるのか？<br>
1個の分布から2次元ベクトルが出るのか？<br>
3×2個の分布があるのか？<br>
を明確に区別する。<br>
このような形状セマンティクスをTFPでは持っている。<br>
TFPがもつ形状セマンティクスには二つある。それは**バッチ形状**と**イベント形状**だ。<br>
**バッチ形状**は異なるパラメータを持つ分布が**いくつ並んでいるか**を表している<br>
例えば、下記のようなコードがあったとする
```python
dist = tfd.Normal(
    loc=[0.0, 1.0, 2.0],
    scale=[1.0, 1.0, 1.0]
)
```
これは一つの正規分布ではなく、
```python
Normal(0, 1)
Normal(1, 1)
Normal(2, 1)
```
という3個の正規分布の集まりである。<br>
よって、*dist.batch_shape*は*(3,)*となる<br>

**イベント形状**は一つの分布から1回サンプルしたときの値の形である。<br>
例えば、2変量(次元)正規分布を考える(※混合正規分布ではない)<br>
```python
dist = tfd.MultivariateNormalDiag(
    loc=[0.0, 0.0],
    scale_diag=[1.0, 1.0]
)
```
1回サンプルすると、*[0.3,-1.2]*のように2次元ベクトルが出てくる<br>
※1次元正規分布の場所はxをサンプリングするだけでいいが、2次元平面上の1点をサンプリングするためにはx1とx2が必要となる<br>
※欲しいのは「確率そのもの」ではなく、
その確率分布に従ってでる確率変数X、つまりサンプル値(実現値)xである。

また、TFPではshapeの表示として左にバッチ形状、右にイベント形状がでてくるようになっている<br>
例えば、*分布オブジェクトの*shapeが(5, 3, 2)だったとき、<br>
batch_shape = (5, 3)<br>
event_shape = (2,)<br>
つまり、<br>
5(*行*)×3(*列*)個の分布があり、<br>
それぞれの分布は2次元ベクトルを出す<br>
と解釈できる<br>
※batch_shapeであえて第一、第二引数と分ける理由は、分布をだたの一列ではなく、**元のデータ構造を保ったまま並べたい**から<br>
例えば、時系列モデルにおいて*(batch=サンプル系列の数(目的変数が何時点存在するか), time=各系列の時点数(一時点の目的変数を予測するのに使う時数))*という構造が良く出てくる<br>
```python
dist = tfd.Normal(
    loc=tf.zeros((32, 100, 1)),
    scale=tf.ones((32, 100, 1))
)

logp = dist.log_prob(y_true)
```
これは*32本の系列*があり、各系列に*100時点ぶんの分布*がある

### .sample()
*data = tdf.distributions*をしてから、分布から何回値をサンプル(とってくる)するかを明示することができる<br>
```python
normals = tfd.Normal([-2.5, 0., 2.5], 1.)
#ここで平均を複数にすると、分散が同一の3つの分布を作れる
samples = normals.sample(1000)
print("Shape of samples:", samples.shape)
```
このときのshapeは*sample_shape+batch_shape+event_shape*となり、(1000,3)と出てくるが、<br>
この1000はsample_shapeで、3はbatch_shapeである

数としては同じだが、意味があるならあえて
```python
samples = normals.sample([10, 10, 10])
```
としてもよい

### dist.log_prob(x)
xという値が、その分布のもとでどれくらい起こりやすいかという対数確率密度(関数)を返す
```python
normals.log_prob([-2.5, 0., 2.5])
```

### sns.histplot()
基本的にヒストグラムを描くためのメゾット。
1変量、2変量のヒストグラムに対応している

引数1:*data*<br>
単にデータを指定する<br>
引数2:*bin*<br>
階級数(棒の数)を指定する<br>
引数3:*binwidth*<br>
棒1本あたりの幅を指定する、連続値の場合はこちらを指定した方がいい<br>
引数4:*binrange(a,b)*<br>
ヒストグラムを描く範囲を指定する<br>
引数5:*stat*<br>
棒の高さを何と表示するか指定する、具体的には<br>
*"count(件数)*"frequency(頻度)"*,*"probability(割合)"*,*"density"(密度、面積が1になるように正規化)*

参考コード
```python
sns.histplot(samples[:, i], stat="density")
plt.title("Samples from a standard Normal")
plt.show()
```

### ブロードキャスト(データ次元拡張)
ブロードキャストするときは最後の次元を合わせる必要がある
```python
xs = np.linspace(-6, 6, 200)#shape(200,)
normals = tfd.Normal(
    loc=[-2., 0., 2.],
    scale=[1., 1., 1.]
)#shape(3,)
```
このままだと最後の次元が200と3で合わないが、
```python
xs = tf.linspace(-6., 6., 200)[:, tf.newaxis]#shape(200,1)
normals = tfd.Normal(
    loc=[-2., 0., 2.],
    scale=[1., 1., 1.]
)#shape(3,)
```
*[:,tf.newaxis]*を付け加えることで、(200,1)と(3,)になる<br>
ブロードキャストの際、(3,)は前に1が補われていると考えるため、(200,1)と(1,3)となり次元が合う

```python
lps = normals.log_prob(xs)#shape(200,3)
```
shape(200,3)となる

### 多変量分布
多変量分布のオブジェクト(インスタンス)を生成する際には、*tdf.MultivariateNormalDiag*を使う<br>
これをplotする際は、*sns.jointplot(x=samples[:, 0], y=samples[:, 1], kind='scatter(kde,hist)')*を使う
```python
samples =
[
  [sample_1_from_dist0, sample_1_from_dist1, sample_1_from_dist2],
  [sample_2_from_dist0, sample_2_from_dist1, sample_2_from_dist2],
  [sample_3_from_dist0, sample_3_from_dist1, sample_3_from_dist2],
  ...
  [sample_1000_from_dist0, sample_1000_from_dist1, sample_1000_from_dist2]
]
```
のように格納されているため、*sample[:,1]*のように行全部、列は指定となる