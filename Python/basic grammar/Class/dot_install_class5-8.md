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