## Python入門　クラス編

### クラスで定義された属性を操作しよう
方法は主に2つある<br>
1.直接属性を操作する方法<br>
2.メゾットを介する方法

### 直接属性を操作する方法
```python
class Post:
    def __init__(self, text):
        self.text = text
        self.likes = 0
    
    def show(self):
        print(f"{self.text} - {self.likes}")

posts = [
    Post("Hello"),
    Post("Hi"),
]

posts[0].likes +=1
```

### メゾットを介して操作する方法
```python
class Post:
    def __init__(self, text):
        self.text = text
        self.likes = 0
    
    def show(self):
        print(f"{self.text} - {self.likes}")
    
    def like(self):
        self.likes += 1 #ここで属性を操作するというメゾットを定義する

posts = [
    Post("Hello"),
    Post("Hi"),
]

posts[0].like()
```
一見前者の方が楽でいいように思えるが、コードの安全性を加味すると後者の方が良いらしい<br>
これからその話をしていく

### 属性へのアクセスを制限しよう
メゾットを介した方がいい理由は、他人との開発をした際それがミスであることに気づきにくいことが挙げられる。

クラス内に書けばその出力を誰でも確認でき、なおかつクラス内の確認で済む

外から属性にアクセスされないようにするには、**self._name**という風にアンダーバーを属性名の前につければよい<br>
この、外から属性にアクセスを防ぐ行為を**カプセル化**という

### setter, getterについて
とはいえ、クラスの外から値を操作し、取得したいという動機は存在する<br>
とはいえ、_を使ったものはアクセス厳禁なため直接はいじれない<br>
この場合でも、別途メゾットを定義する<br>
コード例
```python
class Post:
    def __init__(self, text):
        self._text = text
        self._likes = 0
    
    def show(self):
        print(f"{self._text} - {self._likes}")
    
    def like(self):
        self._likes += 1
    
    #setter(属性に単に値をセットするメゾット)
    def set_likes(self,num):
        self._likes = num
    
    #getter(属性に単に値をゲットするメゾット)
    def get_likes(self):
        return self._likes
```
**setter**(属性に単に値をセットするメゾット)や**getter**((属性に単に値をゲットするメゾット))は重要だが、よりよい記法も存在する

### @propertyを使ってみよう
とはいえ、setterとgetterを一々書くより、直接属性を操作するような書き方ができたら、シンプルでやりやすい

```python
class Post:
    def __init__(self, text):
        self._text = text
        self._likes = 0
    
    def show(self):
        print(f"{self._text} - {self._likes}")
    
    def like(self):
        self._likes += 1
        
    @property#selfしか引数がないので、getterの機能しかない,まず値の取得をする必要がある
    def likes(self):
        return self._likes
    
    @likes.setter #setter,@propertyで決めた機能に付け足していくイメージ、「@」は右のオブジェクトに.~のメゾットを装飾して、という意味あいがある
    #また、命名規則として、「@propertyのメゾット名」+「.setter」と決まっている
    def likes(self,num):
        self._likes = num
```
このように書いたとき、仮に、

```python
posts = [
    Post("Hello"),
    Post("Hi"),
]

posts[0].likes = 100
```
という存在しない.likesが来ても、@propertyがあると、それが属性ではなく、メゾットだ!という認識をさせてくれる<br>
つまり、やっていることは普段できないメゾットへの直接代入ともいえる

### 使い分けについて
setter,getterといったクラスにしっかり書く場合と、@propertyの使い分けだが、主に2つある<br>
一つは、@propertyの方が後からの修正に効くからだ。既存の関数の使い方を残したまま、中身だけ変えたいといったときに便利となる<br>
例えば、@propertyだとあたかも属性に直接編集しているような書き方ができるため、もともとカプセル化しておらずカプセル化したときや、<br>
単なる属性に、**名前はそのままでメゾットの要素を付け足したい**といった要望に応えやすい
二つ目は、@propertyを使うと、*circle.get_area()*のように()を使わず、*circle.area*のように書ける。<br>
これはメゾットの中身があたかも何かステータス(状態やデータ)のようなものであれば、()を使わずかけた方が直感的でよい<br>
逆に、メゾットの中身が、行動にまつわるものだとsetter,getterの方が好ましい

### setterとgetterの順序について
直感的には、値のセットである*setter*が最初にきて、返り値としてゲットする*getter*が後に来た方がわかりやすい<br>
しかし、*@property*を使うときは、getterである、*@property*が最初に来て、*@().setter*が後に来る<br>
こうなる理由は、2つある<br>
1つは、*@().setter*は先に、*@property*を定義しないと使えないからである<br>
コードの内のコメントアウトで述べたように、*@().setter*は、*@property*で決めた**機能に付け足していくもの**である<br>
繰り返しになるが、「@」は右のオブジェクトに.~のメゾットを装飾して、という意味あいがある<br>

2つ目は、そもそも**読み取り専用**が基本だからである<br>
setterはあくまでオプションで、読み取りのみで終わることも多い<br>
また、最初にgetter(property)があることで返り値として何があるか把握しやすい<br>

### @propertyのみを使うメリットについて
先程、@propertyのみで用いることがあると述べたが、直感的にはただあるオブジェクトを読み込んで返り値として吐き出すことのどこに需要があるかわからない<br>
そこでメリットを2つ述べる<br>
1.関数の結果を属性のように()を付けず扱える

```python
# 【クラスを作る側のコード】
class User:
    def __init__(self, full_name):
        self.full_name = full_name  # ただの変数

# 【クラスを使う側のコード（外部のプログラム）】
user = User("山田 太郎")
# あちこちで「 user.full_name 」として呼び出されている
print(user.full_name)
```
上記のコードに新たな要望があって関数ライクのコードを書く必要になったとき、<br>
あらゆるコードで*full_name()*という風にカッコをつける必要に追われる<br>
しかし、@propertyを使えばそのままクラスの中身を変えるだけで対応できる<br>

2.読み取り専用にして安全を守れる
外から勝手に書き換えられては困る変数がある<br>

```python
# 悪い例（普通の変数）
class BankAccount:
    def __init__(self):
        self.balance = 1000  # 残高

account = BankAccount()
account.balance = 99999999  # 外部から残高を変更できてしまう

# 良い例（@propertyで読み取り専用にする）
class BankAccount:
    def __init__(self):
        self._balance = 1000

    @property
    def balance(self):
        return self._balance

account = BankAccount()
print(account.balance)  # 1000 (見るのはOK)

# 外部から書き換えようとすると...
account.balance = 99999999  
# ❌ AttributeError: can't set attribute (エラーになって防いでくれる！)
```
2つ目に関しては、厳密には*account._balance*に代入してしまえば書き換えられてしまう<br>
しかし、_は明示的に書き換え厳禁という印なのでより設計として安全になる<br>


明確に使われやすいのはどちらかと言えば1で、**あるメソッドの結果を属性のように扱いたい**というときに使われる傾向にある<br>

2も良く見られて、実際、@propertyを用いた*balance*を外部用にして、<br>
何も用いない*_balance*を内部のコード用にするということをする