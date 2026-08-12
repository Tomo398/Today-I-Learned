## selenium 基本メゾット

### driver = webdriver.Chrome()
ChromeDriverを読み込み

### driver.get("URL")
ブラウザを起動、ページを遷移

### close()
ウィンドウを閉じる

### title
ページのタイトルを取得

### element = find_element(By.,"name")
要素を検索し、見つかればその要素を返す<br>
By.の後で検索する要素(属性,cssセレクタ,tag名など)を第一引数、具体的な要素名を第二引数で指定する<br>
第一引数には、*By.TAG_NAME*,*By.ID*,**By.NAME**,*By.CLASS_NAME*,**By.CSS_SELECTOR**,**By.XPATH**,*By.LINK_TEXT*,*By.PARITAL_LINK_TEXT*がある

仮に返り値が複数欲しいなら**find_elements**と複数系にする必要がある

### tag名とは
要素の種類を表す<br>
タグ自体は*<>*を指している。*input*だと入力、*p*だと段落、*a*ならリンク、imgだと画像を定義している

### 属性とは
タグの中に書かれている要素(追加情報)のこと<br>
**id属性**:要素の固有の識別番号。原則ページ内の**1つだけ**<br>
**name属性**:フォームの入力データを送信する際の名前(重複あり)、基本的に直感的に見てわかる機能名がnameとなる<br>
例:ユーザー名を入れるボックス(user_name)<br>
   支払方法(payment)<br>
**class属性**見た目(デザイン)のグループ名(重複あり)<br>
*href属性*:リンク先のURL(aタグによくある)
*type属性*:入力の細かい種類(test,password,submitなど)
*value属性*:その要素が持つ中身のデータ<br>
            1.初期値
            2.ボタンの表面に表示される文字
            3.サーバーに送られるデータ

#### cssセレクタとは
上記の属性などの複数条件を**組み合わせて**、条件にぴったり当てはまる要素を指定するための構文<br>
クラスを指定する際は**ドット(.)**をつけて、IDを指定する際は**シャープ(#)**をつける<br>
また、IDやクラス以外の自由な属性を指定する際は、**角カッコ([])**を使い、階層を下りたい場合は**半角スペース**を使う
```html
<div class="login-form">
    <input type="text" name="userId" class="input-field">
    <button type="submit" class="btn btn-primary">ログイン</button>
</div>
```

```python
#パターンA(タグ+クラス)
#CSSセレクタを使って、ログインボタンを見つけてクリックする
#button.btn-primary （buttonタグで、かつクラス名に btn-primary を持つもの）
driver.find_element(By.CSS_SELECTOR, 'button.btn-primary').click()

#パターンB(タグ+属性)
#button[type="submit"] （buttonタグで、かつtype属性が submit のもの）
driver.find_element(By.CSS_SELECTOR, 'button[type="submit"]').click()

#パターンC(クラス+クラス)
#.login-form .btn （login-form というクラスの中にいる、btn というクラスの要素）
driver.find_element(By.CSS_SELECTOR, '.login-form .btn').click()
```

### 部分一致について
CSSセレクタには部分一致の機能がある。<br>
仮にAサイトではclass="btn-submit-red",Bサイトではclass="btn-submit-blue"だとしても、<br>
親クラスに button[class*="btn-submit"]（クラス名に btn-submit を含むボタン）というセレクタを書いておけば<br>
両方のサイトを1つのメゾットでキャッチできる

### By.LINK_TEXT,"text"
対象の要素が**aタグ**で、かつ画面にわかりやすく文字が表示されているときに使う
```html
<a href="/logout.php" class="nav-item">ログアウトする</a>
```


### BY.XPATH,"xpath"
xpathで要素を検索<br>
HTML構造をPCのフォルダ階層のようにルートから辿って要素を見つける方法<br>
cssセレクタよりも複雑な条件で探索できる


### element.send_keys("文字列")
(任意の要素)テキストボックスに文字列を入力

### element.click()
(任意の要素)ボタンをクリックする

### WebDriverWait(driver,time).until(EC.)
指定した要素が出るまで待機するメゾット<br>
*WebDriverWait(drver,time)*:driverをtime秒間見張るタイマーをセットする<br>
*until(...)*:()内の条件が満たされるまで待つ<br>
返り値は対応する要素を返す

### E(xpected)C(onditions).の種類
#### presence_of_element_located
画面には見えていなくても、要素が登場するまで待機

#### element_to_be_clickable
クリックできる状態になるまで待機

#### frame_to_be_available_and_switch_to_it
別のwebページが埋め込まれたframeが読み込まれるのを待って、自動でその中に移動(視点を切り替える)<br>
例:ページ内のYoutube動画、Twitterのタイムライン、クレカの決済入力欄

#### lambda d: d.execute_script("return document.readyState") == "complete"
意味あいとしては、webページ全体の読み込みを100%と待つメゾット<br>
*"document.readyState"*:ブラウザが持つページの読み込みステータス<br>
*d.execute_script("return ...")*:seleniumからブラウザに対して直接javascriptの命令を実行させて、結果を受け取るメゾット

#### new_window_opened
ボタンなどをクリックした後、新しいタブ(window)が実際に開くのを待つメゾット


### Select(select_element).select_by_value("")
webサイトの**プルダウン**からvalue属性の値を指定して、任意の値を直接入力するメゾット


### find_elementの第一引数の使い分けについて
第一引数には様々な指定方法があり悩ましいが、以下のような手順で考えていきたい

1.IDはあるか？ <br>
  ➡ ある： By.ID で即決！<br>
  ➡ ない： 2へ<br>

2.NAMEはあるか？（特に入力欄）<br>
  ➡ ある： By.NAME で決まり！<br>
  ➡ ない： 3へ<br>

3.操作したいものが「リンク(aタグ)」でわかりやすい文字があるか?<br>
  ➡ ある： By.LINK_TEXT で決まり！<br>
  ➡ ない： 4へ<br>


4.独自の属性や、簡単なクラス名の組み合わせで特定できるか？<br>
  ➡ できる： By.CSS_SELECTOR でスマートに書く！<br>
  ➡ 構造が複雑すぎる、または「文字（テキスト）」でしか判断できない： 5へ<br>

5.最終手段<br>
  ➡ By.XPATH を使って執念で見つけ出す！


### .execute_script("arguments[0].click();", button)
複数のフレーム等に邪魔されて.click()できないとき、ブラウザのJSの力を借りてクリックする<br>
第一引数はJSのコードで次に渡される引数(データ)をクリックするという意味あい

