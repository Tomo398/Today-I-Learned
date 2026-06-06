## Python入門　クラス編

### クラスに紐づいた属性、メゾット
今までは*self*というインスタンスに紐づいた属性、メゾットだったが、クラスそのものに紐づけることも可能<br>
```python
class Post:
    _count = 0　#ここでポストに紐づいた変数を作成する、実質的な初期化
    def __init__(self, text):
        self._text = text
        self._likes = 0
        Post._count += 1
        
    @classmethod
    def show_count(cls):#この際の引数は暗黙知的にclsとなる
        print(f"{cls._count} instances created")
    
    def show(self):
        print(f"{self._text} - {self._likes}")
    
    def like(self):
        self._likes += 1

posts = [
    Post("Hello"),
    Post("Hi"),
]
    
Post.show_count()
```
上記のとおり、クラスに紐づいた変数を作成する際には、*__init__*の前に変数を代入させさえすればよい。<br>
しかし、初期化の挙動を捉えたいという希望があれば、改めて__init__にも書く<br>
また、そのクラスに紐づいたメゾットを書く際には **@classmethod**と書くこと

基本的にインスタンスに紐づいた属性、メゾットが主だが、クラス全体に関わる何かを示したいときに稀に使われる