## Python入門　クラス編

### 独自のデータ型を作ってみよう
文字列型:post = "Hello"<br>
リスト型:post = ["Hello",3]<br>
辞書型:post = {"text":"Hello","likes":3}

仮にこのように関数を定義したとする
```python
def show(post):
```
postに対してのみ使われるなら、postに対するメゾットとして使えると便利⇒そのためにクラスを使う

関数とデータをまとめて、、、

```python
post = text"Hello" likes3 show()
```
みたいに書きたい、そのために定義してあげる

1.構造を定義する:class Post:<br>
2.新しい値を作る:post = Post()　　(クラスから作られた値は**インスタンス**と呼ぶ)<br>
3.新しい値を使う:post.text = "Hello"

                post.likes = 3 (値に紐づいた変数(likes)は属性という)
                  
                post.show() (特定のデータ型に特化した関数⇒メゾット)

### クラスを使ってみよう

```python
class Post:
    def __init__(self,text,likes):#仮引数の設定
        #モチベとして、独自の属性を設定したい、しかし、属性が所属する先を設定する必要があり、それを明示する必要がある。それゆえ、クラスを定義する際は任意のオブジェクトに対応する変数としてselfを使う
        #独自の属性設定
        self.text = text
        self.likes = likes

posts = [
    Post("Hello",3),
    Post("Hi",5),
]

print(posts[0].text)
print(posts[1].likes)
```
※__init__()はPython側が勝手に解釈してくれる*特殊メゾット*とされる

※そもそも、**クラスの中で定義する関数は全てメゾット**ととされる

### メゾットを定義してみよう

```python
class Post:
    def __init__(self, text, likes):
        self.text = text
        self.likes = likes
    
    def show(self):
        print(f"{self.text} - {self.likes}")

posts = [
    Post("Hello", 3),
    Post("Hi", 5),
]
posts[0].show()
posts[1].show()
```
基本的にクラスでメゾットを定義する際の**第一引数はself**であるが、第二引数以降は別の引数もある場合がある<br>
このselfの属性(インスタンス変数)にする場合と一過性の単なる変数として扱う違いは、**常にそのデータを保持して使いたいか**にある<br>
例えば、*self.name*は不可変であるためずっと固定で使いたい<br>
しかし、*eaten_food*という変数があったときこれを不可変にしたいだろうか?いや、毎日食べるものは変わるため、可変(一過性)としたい<br>

具体例
```python
class User:
    def __init__(self, name):
        self.name = name

    # self 以外に「food」という引数を受け取るメソッド
    def eat(self, food):
        print(f"{self.name}さんは{food}を食べました。")

    # 複数の引数（x, y）を受け取るメソッド
    def move(self, x, y):
        print(f"{self.name}さんは、座標({x}, {y})に移動しました。")

# --- 使い方 ---
user1 = User("おすぎ")

# メソッドを呼ぶときは、selfの分の引数は自動で渡されるので、
# 2番目以降の引数（food や x, y）だけを指定します。
user1.eat("ラーメン")        # 出力: おすぎさんはラーメンを食べました。
user1.move(10, 20)          # 出力: おすぎさんは、座標(10, 20)に移動しました。
```

