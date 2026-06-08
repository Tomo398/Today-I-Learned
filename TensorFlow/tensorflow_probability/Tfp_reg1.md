## TensorFlow Probability
### 確率的回帰

### tfp.layers.DistributionLambda(lambda t: tfd.Normal(loc=t, scale=1)),
このlambdaで出力を何にするかを指定できる<br>
この場合、*lambda t:tfd.Normal(loc=t,scale=1)*としているため、分散は固定だが、出力として平均を出させる<br>
また、tfpライブラリはkeras3に対応していないため、keras2(tf_keras)使う必要がある<br>

### import文
```python
import numpy as np
import tensorflow as tf
import tensorflow_probability as tfp
import tf_keras#keras2を使う必要があるが、asでtfkと書けば解決するような
```

### 負の対数尤度(確率密度)
確率的回帰においては、損失関数を負の対数確率密度関数に設定する必要がある<br>

```python
negloglik = lambda y, rv_y: -rv_y.log_prob(y)#lambda 引数:式 ※(式の結果を返り値とする)
```
rv_y：モデルが予測した分布<br>
y:実測値

### コード例
```python
model = tf_keras.Sequential([
  tf_keras.layers.Dense(1),
  tfp.layers.DistributionLambda(lambda t: tfd.Normal(loc=t, scale=1)),
])

model.compile(optimizer=tf_keras.optimizers.Adam(learning_rate=0.01), loss=negloglik)
model.fit(x, y, epochs=1000, verbose=False)
```
tf_keras.layers.Dense(1)は出力が平均だけだったから1だが、分散も出力するなら*(1+1)*となる

```python
yhat = model(x_tst)
assert isinstance(yhat, tfd.Distribution)
```
実際のデータに当てはめている<br>
*assert isinstance(yhat, tfd.Distribution)*でyhat が TensorFlow Probability の分布オブジェクトであることを確認する<br>
つまり、モデルの出力が確率分布になっているかを確認している

### 予測分布のplot

```python
mean = yhat.mean().numpy().squeeze()#.squeeze()で念のため1次元配列のみにしている
std = yhat.stddev().numpy().squeeze()
x_plot = x_tst.squeeze()

plt.figure(figsize=(8, 5))

plt.plot(x, y, 'b.', label='observed')
plt.plot(x_plot, mean, 'r', label='mean', linewidth=3)

plt.fill_between(
    x_plot,
    mean - std,
    mean + std,
    alpha=0.3,
    label='mean ± 1 std'
)

plt.fill_between(
    x_plot,
    mean - 2 * std,
    mean + 2 * std,
    alpha=0.15,
    label='mean ± 2 std'
)

plt.legend()
plt.show()
```

### 分散も出力に使う場合
```python
model = tf_keras.Sequential([
  tf_keras.layers.Dense(1 + 1),
  tfp.layers.DistributionLambda(
      lambda t: tfd.Normal(loc=t[..., :1],
                           scale=1e-3 + tf.math.softplus(0.05 * t[...,1:]))),#Dense 層が出した2つの値のうち、2つ目を標準偏差に使うという意味
                           #分散が負にならないようにsoftplus(ソフトマックス関数)をかませている
                           #0.05は任意だが、t[...,1:]が大きく変わってもいいようにしている
])

```