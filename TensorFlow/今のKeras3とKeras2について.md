## Keras3とKeras2について

### Keras2について
Keras2のときは、TensorFlowの中にKerasが組み込まれていた<br>
よって、tensorflow.kerasというような記述をした

```python
import tensorflow.keras as tfk
tfk.model()
```

また、tensorflow.probabilityはkeras2でしか使うことができないため、import文で<br>
**import tf_keras**と書く必要がある

### Keras3について
Keras3はtensorflow、pytorch、jaxを統合的に扱えるライブラリとなった<br>
よって、
```python
# ○ 必ずインポートする前にバックエンドを固定する
import os
os.environ["KERAS_BACKEND"] = "torch"
import keras

keras.model()
```
仮にos.environ["KERAS_BACKEND"]を指定しなかったら、importで存在するライブラリをあたり存在するものを使用する