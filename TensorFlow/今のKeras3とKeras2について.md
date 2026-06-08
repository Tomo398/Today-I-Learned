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
import keras
# 2. 裏で何を動かすか（PyTorch / JAX / TensorFlow）は、環境変数で指定する
import os
os.environ["KERAS_BACKEND"] = "tensorflow"  # または "jax", "tensorflow"

keras.model()
```
仮にos.environ["KERAS_BACKEND"]を指定しなかったら、importで存在するライブラリをあたり存在するものを使用する