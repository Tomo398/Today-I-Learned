## _parameter_properties()
*_parameter_properties()*は各分布パラメータについて、「1個の分布を作るのに何次元必要か」をTFPに教えるメソッドである<br>
これにより、TFPが分布のbatch_shapeを自動計算できる
### _parameter_properties()の必要性
単にパラメータのshapeを見ればいいように感じるが、1mdで示したように、<br>
同じ*shape==(3)*でも、batch_shapeが3でevent_shapeが[]の場合と、batch_shapeが[]で、event_shapeが3の場合は同じ表示になる<br>
※これは前者が異なる正規分布が3つある場合で、後者が3次元正規分布がある場合に対応している<br>
そこで、TFPにNormalのlocは、1個の分布につきスカラー<br>
MultivariateNormalのlocは、1個の分布につきベクトルと教える必要がある<br>
その情報が*ParameterProperties.event_ndims*である

### event_ndims
注意点として、ここでの*event_ndims*はevent_shapeそのものではない<br>
**1個分のパラメータの次元数**を表す<br>
仮に*event_ndims=1*だとしたら、最後の軸の長さが1という意味でなく、右端から1本の軸を1個分のパラメータに使うという意味である<br>
例えば、parameter.shape = [32, 3]、event_ndims = 1であれば<br>
3が1個分のパラメータである<br>
基本的には、パラメータTensorのshapeは以下のように分解される<br>
$$ parameter.shape = parameter batch shape + 1個分のparameter shape$$
そして、*event_ndims*は1個分のparameterが何**次元**か(スカラー?1次元ベクトル?など)を表す


例1:**スカラーパラメータ**<br>
Normalのlocは1個の正規分布につき1個の数値である<br>
```python
tfp.util.ParameterProperties(event_ndims=0,)
```
*event_dims=0*なので、パラメータshape全体がbatch部分となる
<br>
※1次元ベクトルとスカラーは似ているものの、別物である<br>
1次元ベクトルは1本の数直線上で向きと大きさを持つが、スカラーは大きさしか持たない<br>

loc.shape = [3]、parameter batch shape = [3]<br>

例2:**複数のスカラーパラメータ**
```python
concentration = tf.constant([1.0,2.0,3.0,])
```
これもスカラーが結局3つあるだけなので、*event_ndims=0*<br>
よって、shape全体の[3]がbatch部分となる<br>

例3:**ベクトルパラメータ**<br>
3変量*(3次元多変量)正規分布を1個作るには、locとして3個の数値が必要<br>
```python
dist = tfd.MultivariateNormalDiag(
    loc=[0.0, 1.0, 2.0],
    scale_diag=[1.0, 1.0, 1.0],
)
```
ここでは、*loc = [0.0, 1.0, 2.0]*というベクトル全体が1個分の*loc*パラメータである<br>
この場合ベクトルは1次元なので、*event_ndims=1*と指定する<br>

例4:複数のベクトルパラメータ
```python
loc = tf.constant([
    [0.0, 1.0, 2.0],
    [10.0, 11.0, 12.0],
])
loc.shape# [2, 3]
```
これはあくまで1次元ベクトルが2つあるだけなので*event_ndims=1*である<br>
よって、右端1本の軸を1個分のパラメータに使う<br>
従って、1個分のparameter shapeは3である

纏めとしていうと、分布の*event_shape*は分布から1回標本を生成した時のshapeであるが、<br>パラメータの*event_shape*は1個分の分布パラメータを表すために必要な右端の軸数(次元数(個数ではない))