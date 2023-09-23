# web 業界 1 年目が「良いコード/悪いコードで学ぶ設計入門」を読んだ感想

## 読んだ経緯

Web 業界の企業に開発未経験で入社し、日々開発業務に奮闘しているのですが基本的なオブジェクト指向、デザインパターンが定着していないことにより理解が進みにくいと感じていました。  
そこで今後のキャッチアップの促進を目指したいと思い当時の部長にお薦めいただいたこの本を読むことにしました。  
「良いコード/悪いコードで学ぶ設計入門」の詳細は[こちら](https://www.amazon.co.jp/%E8%89%AF%E3%81%84%E3%82%B3%E3%83%BC%E3%83%89-%E6%82%AA%E3%81%84%E3%82%B3%E3%83%BC%E3%83%89%E3%81%A7%E5%AD%A6%E3%81%B6%E8%A8%AD%E8%A8%88%E5%85%A5%E9%96%80-%E2%80%95%E4%BF%9D%E5%AE%88%E3%81%97%E3%82%84%E3%81%99%E3%81%84-%E6%88%90%E9%95%B7%E3%81%97%E7%B6%9A%E3%81%91%E3%82%8B%E3%82%B3%E3%83%BC%E3%83%89%E3%81%AE%E6%9B%B8%E3%81%8D%E6%96%B9-%E4%BB%99%E5%A1%B2/dp/4297127830)

## 学びや気づき

本書を読んで学んだこと、新しい気づきをまとめていきます！  
度々でてくるコードは動作確認をとっていないため細かいミス等あるかもしれません。。

### 誤った値の渡し方を型で防ぐ

以下のお金を表すクラスがあったとする

```
// *getterは省略
class Money{
  private int amount; // お金の額
  private Currency currency; // お金の通貨
  // コンストラクタ
  Money(int amount, Currency currency){
    this.amount = amount;
    this.currency = currency;
  }
  // お金の加算を求める
  void add(int otherAmount){
    this.amount += otherAmount;
  }
}
```

上記の場合、お金の加算処理で引数を間違える実装をしても問題なく処理が通ってしまう。

```
Currency currency = Currency.getInstance(Locale.JAPAN); // 日本の通貨(円)
Money totalMoney = new Money(10000,currency); // 1万円のお金
Money addedMoney = new Money(5000,currency); // 5千円のお金
int ticketAmount = 30; // チケットの枚数（どこかで使う想定）
// totalMoney.add(addedMoney.getAmount()); //実際にやりたかったこと
totalMoney.add(ticketAmount); // 間違えてチケットの枚数を加算してしまった。
```

予めお金のクラスを引数にしないとエラーが出るような実行にすることで上記のようなミスができないような設計にできる。

```
class Money{
  private int amount;
  private Currency currency;
  Money(int amount, Currency currency){
    this.amount = amount;
    this.currency = currency;
  }

  //引数の型をint -> Moneyに変更
  void add(Money otherMoney){
    this.amount += otherMoney.getAmount();
  }
}
```

```
Currency currency = Currency.getInstance(Locale.JAPAN);
Money totalMoney = new Money(10000,currency);
Money addedMoney = new Money(5000,currency);
int ticketAmount = 30;
totalMoney.add(ticketAmount); // 型が違う(想定:Money 実際:int)のでコンパイルエラー
```

本来はクラス内フィールドに「final」をつけて少しリファクタされた形が理想系だが、ここでの説明のスコープ外なので省略。

### 値を「不変」にして、安定動作を構築する

以下のお金、人を表すクラスがあったとする

```
class Money{
  int amount; // お金の額 *可変になっている
  private Currency currency; // お金の通貨
  // コンストラクタ
  Money(int amount, Currency currency){
    this.amount = amount;
    this.currency = currency;
  }
}

class Human{
  private String name; // 名前
  private Money money; // お金
  // コンストラクタ
  Human(String name, Money money){
    this.name = name;
    this.money = money;
  }
  void displayProfile(){
    //「名前:this.name 所持金:this.money」と表示する処理
  }
}
```

以下のように同じお金のクラスを使い回し、直接お金の加算をすることで思わぬところにも影響が出てしまう可能性がある。(例が少しわかりにくいですね。。)

```
Currency currency = Currency.getInstance(Locale.JAPAN); // 日本の通貨(円)
Money money = new Money(10000,currency); // 所持金のお金
Human tanaka = new Human("tanaka", money); // 人
Human suzuki = new Human("suzuki", money); // 人
tanaka.money.amount += 1000; // tanakaさんの所持金を増やしたい
tanaka.displayProfile(); // 所持金:11000
suzuki.displayProfile(); // 所持金:11000 -> バグの要因
```

フィールド変数を「可変」にすることで影響範囲が大きくなり保守性に欠ける構成になってしまうのでフィールド変数を「不変」にしてお金クラスで返却する加算メソッドを併用して安定した動作を保証できる構成にする。

```
// *getter,setterは省略
class Money{
  private final int amount; // 不変になっている
  private final Currency currency; // 不変になっている
  Money(int amount, Currency currency){
    this.amount = amount;
    this.currency = currency;
  }
  // お金の加算をする
  Money add(Money otherMoney){
    int addAmount = this.Money.getAmount() + otherMoney.getAmount(); // お金の加算
    Money addMoney = new Money(addAmount, this.Currency); // 加算したお金のクラスを生成
    return addMoney;
  }
}
class Human{
  private final String name;
  private final Money money;
  Human(String name, Money money){
    this.name = name;
    this.money = money;
  }
  void displayProfile(){
    //「名前:this.name 所持金:this.money」と表示する処理
  }
  Human addMoney(Money otherMoney){
    Money addMoney = this.getMoney().add(otherMoney);
    Human addedMoneyHuman = new Human(addMoney, this.getName);
    return addedMoneyHuman;
  }
}
```

```
Currency currency = Currency.getInstance(Locale.JAPAN);
Money money = new Money(10000,currency);
Money addMoney = new Money(1000,currency); // 加算するようのお金
Human tanaka = new Human("tanaka", money);
Human suzuki = new Human("suzuki", money);
// tanaka.money.amount += 1000; コンパイルエラー
tanaka = tanaka.addMoney(addMoney); // お金の加算
tanaka.displayProfile(); // 所持金:11000
suzuki.displayProfile(); // 所持金:10000
```

上記の実装だと修正が必要なところはまだあるが、値を直接変えることによる副作用は防げるようにはなった。

### 初期化ロジックの分散を防ぎ、影響範囲を狭める

次のような要件があったとする。

- 新規会員に入会ポイントを配布する。
- 配布ポイント数は通常会員：3000 ポイント、優遇会員：5000 ポイントとする。

上記に対して NewMenberPoint クラスを実装し、それを扱う箇所で生成して利用する。

```
class NewMenberPoint{
  private final int point; // ポイント
  NewMenberPoint(int point){
    this.point = point;
  }
}
```

```
NewMenberPoint pointForNormalMenber = new NewMenberPoint(3000); // 通常会員用の新規入会ポイント
NewMenberPoint pointForPriorityMenber = new NewMenberPoint(5000); // 優遇会員用の新規入会ポイント
```

今後入会のポイント配布条件が増えたりした先に、配布するポイント数に変更があった場合に上記の該当する pointFor\*Menber のロジック全てを修正しないといけなくなる。  
そのため NewMenberPoint クラスの方で要件に合ったポイントのインスタンスを生成する処理を実装することで影響範囲を狭めることができる。

```
class NewMenberPoint{
  private final int point;
  private final int normalMenberPoint = 3000; //通常会員用
  private final int priorityMenberPoint = 5000; //優遇会員用
  private NewMenberPoint(int point){ // privateをつけて不要な生成を防ぐ
    this.point = point;
  }
  // 新規通常会員用のポイント
  static NewMenberPoint pointForNormalMenber(){
    return new NewMenberPoint(normalMenberPoint);
  }
  // 新規優遇会員用のポイント
  static NewMenberPoint pointForPriorityMenber(){
    return new NewMenberPoint(priorityMenberPoint);
  }
}
```

```
// 生成の処理
NewMenberPoint pointForNormalMenber = NewMenberPoint.pointForNormalMenber();
NewMenberPoint pointForPriorityMenber = NewMenberPoint.pointForPriorityMenber();
```

このように実装することでポイント数の変更があっても NewMenberPoint クラス定義のフィールドを変更するだけで対処できる。

### 目的、責務を考慮した命名をつけることで高凝集な設計を目指す

悪い例：商品の価格を示すクラスを「商品」で一律に命名する  
→「予約」、「注文」、「在庫」、「発注」といろんな役割を全て背負う設計になってしまう（低凝集）  
良い例：「予約される商品」、「注文される商品」、「在庫にある商品」、「発注される商品」に分けて命名する  
→ 責務に合わせて設計することができ、それぞれの責務に対しての追加、変更があった場合に該当部分だけを修正すれば良い設計にできる

## 感想

本書を読んでみて、オブジェクト指向についての理解が足りていなかったと改めて痛感しました。。  
日頃業務で見かけていた値オブジェクトに関しても可読性は上がりそうな気がするものの、そんなに費用対効果があるのか？と少し疑問に思っていました。
しかし、10 年先で起こり得ることを考慮すると未然に防ぐための手段としてとても有用的であると理解できました。  
また、実装の「書き方」だけでなく、ドメイン、テスト駆動に基づく「進め方」も工夫すると保守性を高めることができると理解したので、
今後は実装の進め方についての学習をしたいなと思いました。
