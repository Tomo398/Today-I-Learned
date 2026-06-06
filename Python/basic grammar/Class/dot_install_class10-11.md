## Python入門　クラス編

### クラスの継承について
あるデータ型を基に新しいデータ型を作りたいという場合がある<br>
基本的に、新しいクラスを定義した際に、その()の中に継承したいクラスを記入すればよい
```python
class Post:
    def __init__(self, text):
        self._text = text
        self._likes = 0

    def show(self):
        print(f"{self._text} - {self._likes}")
    
    def like(self):
        self._likes += 1

class SponsoredPost(Post):
    pass
```
この際、SponsoredPost(Post)はPostのメゾットも含めてすべての機能が使える<br>
継承元のクラスを**親クラス(super)**、継承後のクラスを**子クラス(sub)**という

### メゾットの上書き(オーバーライド)
子クラスで同じメゾットが定義されると、子クラスのメゾットの方が優先して使われる<br>
また、子クラスで親クラスど同名のメゾットを定義することを**メゾットのオーバーライド**という

子クラスでも同様に初期化を行うが、仮に重複する場合は、**super().__init__(text)**と書くことで親クラスの初期化をそのまま使うことができる<br>

```python
class Post: # 親クラス Superクラス
    def __init__(self, text):
        self._text = text
        self._likes = 0

    def show(self):
        print(f"{self._text} - {self._likes}")
    
    def like(self):
        self._likes += 1

class SponsoredPost(Post): # 子クラス Subクラス
    def __init__(self, text, sponsor):#オーバーライド
        # self._text = text
        # self._likes = 0
        super().__init__(text)
        self._sponsor = sponsor

    def show(self):#オーバーライド
        print(f"{self._text} - {self._likes} sponsored by {self._sponsor}")

posts = [
    Post("Hello"),
    Post("Hi"),
    SponsoredPost("Hey", "dotinstall"),
]

posts[2].like()#オーバーライドしていないメゾットも使えることを確認

for post in posts:
    post.show()
```
なお、親クラスにあって子クラスにないメゾットを使うとき、self自体は子クラスのselfが使われる