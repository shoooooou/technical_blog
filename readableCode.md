# リーダブルコードを読んだ感想

## はじめに

Web 業界の企業に未経験採用で入社をして日々開発業務に奮闘しているのですが、やっと自走してタスクを進められるようになったので保守性にも目を向けて学習していこうと思いこの本を読むことにしました。

## 学び

### 変数の命名

#### 汎用的な名前を避ける

以下のコードは後続の処理を追わないと tmp の意味が理解できない。

```
string tmp; // この時点では何に使われるかわからない
tmp = name; // 名前
tmp += ":" + adress; // 住所
console.log(tmp);
```

具体的な変数名にすることで前段のところで何を示しているのかわかるようになる

```
string userInfo; // ユーザに関する情報を示していることがわかる
userInfo = name; // 名前
userInfo += ":" + adress; // 住所
console.log(userInfo);
```

他にも retval(返り値),hoge も使われがちなので注意。

抽象的な命名が必ずしも悪いわけではなく、for 文のイテレータとしてよく使われる i, j, k などは開発者の共通認識であるので可読性を妨げる要因にならなそうと判断できれば命名することに問題はない。

#### 命名の適切な文字数は使用されるスコープによって判断する

変数名は情報を示すことを考えて命名すると思いますが全ての情報を付与すると読み取りづらかったり、複数行に跨ったりとかえって読みにくくなる。

```
calculateTotalAmountForCalculateTheAmountOfTaxes
```

あとでやることはその処理のところで補足すれば良く、上記のメソッドは合計金額を算出する役割を持っているということだけわかれば良いので for 以降は不要と判断できる。

```
calculateTotalAmount
```

この例に倣うと文字列が長すぎるのは NG と捉えてしまうが、省略記法を使用し、短すぎるのも用途によっては適切ではない。  
 命名の判断として、目的は読み手が理解しやすいということを観点として適切な命名をつけるのが良い。  
 短い命名に関してはグローバル変数で日付を示す変数名「day」を使用すると day を使用するたびに何の日付を示しているのか理解しにくいので、「applyDay」等具体的な命名の付け方をするのが望ましいが、3 行程度の if 文の中で定義される変数であれば 3 行の中でしか登場しないことが確約されるので命名が抽象的であっても読み手の理解の妨げにならなければ問題はない。

#### 誤解されない意味合いを意識する

範囲を指定して処理をするメソッドの引数は「first」,「end」、包含または排他的範囲には「begin」,「end」を使用する。

### 整形

#### メソッドの引数の粒度を揃える

ユーザ情報をリストに加える addUserInfo のメソッドを複数呼ぶ場合

```
addUserInfo("hoge","20","male","170");
addUserInfo("hogehoge","",male","170");
addUserInfo("foo","20","female","170");
addUserInfo("bar","20","","170");
addUserInfo("foobar","","","170");
```

上記だと何番目の引数に何が記述されているか理解しにくいので引数毎に縦に揃えると見えやすくなる

```
addUserInfo("hoge"    , "20", "male"  , "170");
addUserInfo("hogehoge", ""  , "male"  , "170");
addUserInfo("foo"     , "20", "female", "170");
addUserInfo("bar"     , "20", ""      , "170");
addUserInfo("foobar"  , ""  , ""      , "170");
```

ただしこれは意見が分かれるようで、新しくメソッドの呼び出しを追加した場合は既存の引数部分を整形する必要があったりと負担が大きくなる懸念がある。  
個人的にはカンマの位置は各引数の直後にあっても可読性は変わらないかなという気がしています。

### ループ、条件

#### 三項演算子

２つの値から選ぶような処理であれば使用した方が可読性が高くなる場合が多い

- 三項演算子

```
timeStr += (hour >= 12) ? "pm" : "am"
```

- if 文

```
if (hour >= 12) {
  timeStr += "pm"
} else {
  timeStr += "am"
}
```

基本的には if/else 文を使うことを意識して、三項演算子の方が簡潔になりそうな場合に使用するという分け方が良いらしいです。

#### ネスト

ネストが深い処理は前段の条件を記憶に保持しながら読まないといけないため可読性が低下する可能性が高い

```
if (user.result == "SUCCESS") {
  if (!user.accessPermission) { // user.resultがSUCCESSであることを記憶していないといけない
    errorStr = "not permission";
  }
} else {
    errorStr = "user invalid value";
}
```

ネストの階層を最小に落とせた方が可読性が高くなる

```
if (user.result != "SUCCESS") {
    errorStr = "user invalid value";
}
if (!user.accessPermission){
    errorStr = "not permission";
}
```
自分で考えたものですが、あまり良い例ではないですね。。  
### 変数
#### 無駄な中間変数を削除する
配列から値を削除する関数があったとする
```
var index_remove = null;
for (var i=0; i < array.length; i += 1) {
  if (array[i] === unnecessaryValue) {
    index_remove = i;
    break;
  }
}
if (index_remove !== null) {
  array.splice(index_remove, 1);
}
```
index_removeは消去する対象の要素の場所を示す役目を担っているが、特別必要な理由は特にないので削除の処理をfor文に直接書いた方が良いと考えられる。
```
for (var i=0; i < array.length; i += 1) {
  if (array[i] === unnecessaryValue) {
    array.splice(index_remove, 1);
    break;
  }
}
```
上記の方が可読性の低下がなく簡潔に書くことができる。
#### 制御フロー変数を削除する
以下のループの処理があったとする。
```
boolean done = false;
while (done) {
  ・・・
  if (条件) {
    done = true;
    continue;
  }
}
```
doneはループの処理でやりたきことが完了したかどうかを示していると考えられるが、こちらも特別必要な理由はないので削除した方が簡潔に書くことができる。
```
while () {
  ・・・
  if (条件) {
    break;
  }
}
```
returnやbreakを活用することで不要な変数を削除した書き方を実現できる
## まとめ  
基本的な理解のしやすいコードの書き方を学ぶことができました。  
この本は１回読んでちゃんと理解するというよりはざっと読んで概要を把握した上で実務経験と照らし合わせながら何度も読み返していくとより理解度が増すという感覚を受けたのでこれからも実務を積んでは読み返すということを継続していこうと思いました。  
可読性の高い実装方法だけでなく、良くない書き方がどのように不便に感じることがあるかなどが説明されていたのがとても良かったです。  
学びで取り上げた例は本書の内容のほんの一部でしかなく、簡潔に書く方法以外にも処理量を考慮した修正パターンなど様々な例が記載されているので気になった方はぜひ読んでいただきたいです。
