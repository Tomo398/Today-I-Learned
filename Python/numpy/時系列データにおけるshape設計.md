## 時系列データにおけるshape設計
時系列モデルではまず、<br>
X(samples, lookback, features) → Y(samples, forecast horizon)<br>
という1サンプルとは何か(samples=1のときについて考える)について決める。その後、使うモデルに合わせて*reshape*,*flatten*する<br>

### 4軸に分けて考える
T : 元データの全時点数<br>
F : 特徴量数<br>
L : 過去何時点を見るか（lookback）<br>
H : 未来何時点を予測するか（forecast horizon）<br>

Xは基本的に(samples, L, F), Yは(samples, H)である<br>

### 各ケースにおける考え方
dataframeをそのままnumpyに変えると、普通Xは(T, F)である<br>

#### 1時点→1時点の場合
この場合は、samples = TとなるためXは(samples, F)である。<br>
Yの方はFは存在しないため(samples,)のみでいい(この,(カンマ)はshapeがタプルで返すためついているだけ)<br>
仮に無駄に(3,1)のようになっていれば*reshape(-1)*をして次元を削減する<br>

### 24時点→1時点の場合
Xを**window化によって**このn行が1セット、と認識させる必要がある<br>
その後に*to_numpy()*すると、X(samples, n, F), Y(samples,)となる<br>
CNNならこのまま渡していいが、Denseは**時間軸を独立した軸と扱わないので**(samples, n×F)のように*reshape(-1)*をする

### 1時点→24時点の場合
この場合は、X(samples, F), Y(samples, 24)でよくて、<br>
Yを**window化**した上でFuctionalAPIの最後を*outputs=layers.Dense(24)(x)*とすればよい<br>
このとき、FunctionalAPIで層をどこかで、(samples,24,F)から(samples, 24×F)に*Flatten*する必要がある

### 24時点→24時点の場合
Xを**window化によって**このn行が1セット、と認識させた後、Yの方も**window化**した上でoutputsで指定する<br>


### window化について
下のコードは、X1=[x1,x2,x3],X2=[x2,x3,x4],...としたい場合である<br>
仮に、X1=[x1,x2,x3],X2=[x4,x5,x6],...のようにしたい場合は*step*=3とすればよい

```python
def make_window(X, Y, window_size=3):
    X_windows = []
    Y_windows = []

    for i in range(start=window_size, stop=len(X),step=1):

        # 過去window_size時点
        X_window = X[i - window_size:i]

        # その次の1時点
        Y_window = Y[i]

        X_windows.append(X_window)
        Y_windows.append(Y_window)

    return (
        np.array(X_windows),
        np.array(Y_windows)
    )
```