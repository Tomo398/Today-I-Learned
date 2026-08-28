## with構文について
### with文の仕組み
with文は最初に何か(前処理)して、最後に必ず何か(後処理)する<br>
(try/finally文)の処理を纏めたものである<br>
より詳細に言うと、指定したクラスに記載されている__enter__メソッドと__exit__メソッドを自動実行するものである<br>

```python
try:#そもそもtryはエラーが発生する可能性があるコードを記述するためのもの
    # 前処理
    file = open("sample.txt", "r")

    # ファイルの読み込み
    print(file.read())

finally: # 処理結果に関係なく必ず実行される
    # 後処理
    file.close()
```
上記のコードのようにcloseメソッドを忘れずに記述しないと、<br>
ファイルが閉じられず、いずれtoo many open filesでエラーとなる<br>
そこで、with構文を用いて以下のようなコードを書くと、closeメソッドを(勝手に実行してくれるため)書かずに済む<br>
```python
with open("sample.txt", "r") as file:
    print(file.read())
```


※pd.read_csvでは特にcloseしていないじゃないかという反論があるが、<br>
そもそもpd.read_csvは内部で下記のようなコードを書くことで、<b r>
勝手にopenしてcloseしていると思われる<br>
```python
# pandas.read_csv の内部イメージ
def read_csv(filepath):
    # 内部でwith構文が使われている
    with open(filepath, 'r') as f:
        data = f.read()
        # 〜データをデータフレームに変換する複雑な処理〜
    
    return dataframe # 処理が終わった段階で自動的にファイルは閉じられている
```
※仮に**エラーが起きても必ず閉じてくれるため**安全性が高い<br>

### コンテキストマネージャとは
そもそもコンテキスト(**context**)とは、**特定のコードが実行される際の状態**を意味する<br>
よりプログラミングに寄せた言い方をすると、**(変数の)中身によらず同じ処理を行う状態**である<br>
注意として、*state*は(変数)の中身によって異なる処理を行う場合、である<br>
そして、コンテキストマネージャとは、*__enter__()*と、*__exit__()*メソッドを実行するもの(クラス)である<br>
__enter__、__exit__という特殊メソッドは、with構文が実行された瞬間、__enter__、__exit__が勝手に実行されるものである<br>

with構文だと、__enter__メソッドの返り値がasの変数に代入され、<br>
__exit__メソッドはwithインデント内の文、もしくはエラーが発生した場合に呼び出される<br>
以下が例である
```python
class name_scope:
    def __init__(self, name):
        # ユーザーが指定した名前（例: "Gamma"）を受け取る
        self._name = name
        
        # 古い名前（1つ前の状態）を記憶しておくための変数
        self._old_name_scope = None 

    def __enter__(self):
        # 1. 現在のシステム全体の名前を取得する
        # （もし既に "Model/" の中にいるなら current_name は "Model/" になる）
        current_name = get_current_name_scope()
        
        # 2. 復元できるように、古い名前をクラスの変数に保存（退避）しておく！
        self._old_name_scope = current_name
        
        # 3. 新しい名前を作る（現在の名前に、今回の名前と "/" を足す）
        # 例: "" + "Gamma" + "/" = "Gamma/"
        new_name = current_name + self._name + "/"
        
        # 4. システム全体の設定を、新しい名前に上書きする
        set_current_name_scope(new_name)
        
        # 5. 生成した新しい文字列（"Gamma/"）を返す
        # （これが with ... as name: の name に入る）
        return new_name

    def __exit__(self, exc_type, exc_value, traceback):
        # 1. __enter__ の時に保存しておいた「古い名前」を取り出す
        # 2. システム全体の設定を、「古い名前」に戻す（上書きし直す）
        set_current_name_scope(self._old_name_scope)
        
        # （エラーの処理等は省略）
        return False
```




### with open以外の使用例
#### 1.一時フォルダ作成と自動削除
プログラムを実行している間だけデータの一時保存先が必要だが、<br>
終わった後は残したくないという場合がある<br>

```python
import tempfile
import os

# TemporaryDirectory を with で開く
with tempfile.TemporaryDirectory() as temp_dir:
    print(f"一時フォルダが作成されました: {temp_dir}")
    
    # フォルダの中にファイルを作って作業する
    temp_file_path = os.path.join(temp_dir, "test.txt")
    with open(temp_file_path, "w") as f:
        f.write("一時的なデータ")
        
    print("フォルダ内で作業中です...")

# withのインデントを抜けた瞬間、フォルダごと自動削除される
print("withを抜けたので、フォルダはもう存在しません")
```
#### 2.web通信の効率化と「自動切断」
```python
import requests

# withを使ってセッション（通信の通り道）を開く
with requests.Session() as session:
    
    # 1回目の通信：ユーザー「github」の基本情報を取得
    # ここで「api.github.com」というサーバーとの間に通信の通り道が作られる
    res_user = session.get("https://api.github.com/users/github")
    
    # 2回目の通信：同じユーザーの「リポジトリ一覧」を取得
    # サーバーが同じ（api.github.com）なので、1回目で作った通り道を使い回して高速に通信！
    res_repos = session.get("https://api.github.com/users/github/repos")
    
    print("2つの異なるデータの取得が完了しました。")
# インデントを抜けると、通信のコネクションが自動で閉じられる
# サーバー側にも負担をかけないお行儀の良いプログラムになる
```

#### 3.設定の一時的な変更(もしくは、変更動作の一時化)
ライブラリによっては、あえてそうしているがそれが時として厄介に
なることがある<br>
例えば、DataFrame型はprintの際一部しか表示されないが、全体を表示させたいときなどがある<br>
TensorFlowの*name_scope*はあらゆる計算グラフの処理名を変える力があるが、<br>
その動作を閉じる(終了する)作業を行わないと、あらゆる処理名に変更した名前がついてしまう<br>
よって、変更する際はwithを使う

```python
with tf.name_scope(name) as name:
      dtype = dtype_util.common_dtype(
          [concentration, rate, log_rate], dtype_hint=tf.float32)
      self._concentration = tensor_util.convert_nonref_to_tensor(
          concentration, dtype=dtype, name='concentration')
      self._rate = tensor_util.convert_nonref_to_tensor(
          rate, dtype=dtype, name='rate')

      super(Gamma, self).__init__(
          dtype=dtype,
          validate_args=validate_args,
          allow_nan_stats=allow_nan_stats,
          reparameterization_type=reparameterization.FULLY_REPARAMETERIZED,
          parameters=parameters,
          name=name)
```
