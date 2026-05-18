## TensorFlow公式　Functional APIのガイド

### Functional APIのガイド

### 基本的な書き方
まず、入力ノードを書く(nameという引数でlayerの名前を付けることができる)
```
inputs = keras.Input(shape=(784,),name="First input layer")
```
次に、次のレイヤーを作成し、入力ノード次のレイヤーを渡す、
前から繋げるとき、**前のlayer変数を後ろに持ってくる**
```
dense = layers.Dense(64, activation = "relu")
x = dense**(inputs)**

x = layers.Dense(64, activation = "relu")**(x)**
outputs = layers.Dense(10)**(x)**

keras.Model(inputs=inputs,outputs=outputs,name="mnits_model")

keras.utils.plot_model(model, "my_first_model.png",show_shapes=True)
```

※最初はレイヤーの生成とレイヤーの実行を分けて書いているが、
それ以降のように、一緒に書いてもいい

トレーニング、評価、推論はsequantial modelと同じ
*model.compile()*や*history = model.fit()*、*test_score = model.evaluate()*は同じである

保存とシリアル化も同じである、*model.save()*や*keras.models.load_model()*を使う

### 複数の入出力の作り方
深くは紹介しないが、モデルそのものを新たに組み込むことができる

#### layers.Conv2DTranspose()
転置畳み込み、フィルターを用いて、特徴マップの次元を上げる
引数は通常の*layers.Conv2D*と同じ

#### layers.UpSampling2D()
単に、画像を引き延ばす行為。間は周囲のピクセル平均や隣にあるピクセルをコピーする、主な引数は*size=(H,W)*である。HとWで縦と横を何倍するか指定できる

### layers.Embedding(input_dim,output_dim)
適当に(辞書的に)単語に数字を割り当てる行為、学習を繰り返す中で正しいベクトル空間になっていく

引数は*input_dim*(辞書の語彙数)、*output_dim*(1単語を何次元のベクトルで表現するか)

### layers.LSTM()
文章のような「順番(時系列)に意味のあるデータ」を最初から最後まで順番に読み込んで要約するレイヤー、単語ごとにそれぞれの次元を作るのではなく、全体でベクトル空間を作れる

引数は*units*、*return_sequences*、*activation*、*recurrent_activation*である。
主に指定するのは*units*(最終的に何次元のベクトルに要約するか)と、*return_sequences*(最終的なベクトルだけ出すか(False)、単語を読み込む度にベクトルを出すか(True))である。*return_sequence=True*にすると、次元は削減されず、次のLSTM層に渡せる。結果的により深い文脈を捉えられる可能性がある。

### layers.concatenate([])
複数入力を結果的に一つの出力としたいとき等に使う。イメージはlayersにおけるpd.merge

### 複数の出力の際の損失関数
複数の出力をする際には、用途に応じて別々の損失関数を設定する必要がある。モデル全体としては、両方の損失関数がバランスよく最小になるように目指すが、*loss_weights*でどちらを重視するかを設定できる。
```
model.compile(
    optimizer=keras.optimizers.RMSprop(1e-3),
    loss=[
        keras.losses.BinaryCrossentropy(from_logits=True),
        keras.losses.CategoricalCrossentropy(from_logits=True),
    ],
    loss_weights=[1.0, 0.2],
)
```

### 複数入力、出力の際のfit
基本的に、*.fit(x,y,epochs)*だし、複数入力出力の際も同じである

しかし、複数あるため、辞書的に書く必要がある
```
model.fit(
    {"title":title_data,"body":body_data,"tags":tags_data},
    {"priority":priority_targets, "department":dept_targets},
    epochs=2,
    batch_size=32,
)
```

### 既存モデルの(中間層の)利用
tfkは静的なモデルであるため、既存モデル(重みが最適化されたモデル)のレイヤーの出力を利用できる

具体的には下記のようなコードとなる(前提としてvgg19というモデルがあるとする)
```
#vgg19のn層目の出力を取る
vgg_n_output = vgg19.get_layer[n].output

#vgg19の入り口から、そのn層目の出口までを「一つの巨大レイヤー」としてパッケージする
vgg_sub_model = keras.Model(inputs=vgg19.input, outputs=vgg_n_output)

#VGGの重みが自分の学習で壊されないようにロックする
vgg_sub_model.trainable = False
```
上記のように書いたら、後はFunctional API風にがっちゃんこすればいい

### より柔軟なモデル設計
基本的にtfkには様々な組み込みレイヤーが用意されているが、
足りない場合は、レイヤークラスを継承して、自分で作成するしかない
基本的にレイヤーを継承して作成することはないが、動的な
アーキテクチャ(再帰的なネットワークやTree RNNなど)を使う必要がある場合Functional APIでは実装できない

しかし、クラスの継承まで行うとほとんどPytorchと書き方が変わらなくなってしまうという現状がある