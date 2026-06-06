## Python入門　クラス編

### @staticmethodを使ってみよう
主にクラスは、データ型を定義しインスタンスを作ることを目的とするが、関連する関数を単にまとめるために使われることもある<br>
単に関数をまとめる場合は、self等の引数を必要としない<br>
その場合、メゾットを定義する前に**@staticmethod**と明記する

コード例
```python
class HtmlHelper:
    @staticmethod
    def to_h1(str):
        return f"<h1>{str}</h1>"
    @staticmethod
    def to_p(str):
        return f"<p>{str}</p>"
    
print(HtmlHelper.to_h1("Hello"))
print(HtmlHelper.to_p("Wow"))
```