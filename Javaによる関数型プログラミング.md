



# Javaによる関数型プログラミング

## コレクションの使用

### 静的スコープとクロージャ

ラムダ式は重複コードを生みやすくしコードの質を落とすという思い込みを静的スコープとクロージャで解決する

#### 静的スコープで重複排除

filter()は、関数を引数に取れるわけではない
コレクションの対象となる要素を表すパラメータを１つだけ持つ関数を受け取り、boolean型の値を返す
つまり、Predicate(述語)を期待している
(Predicateってなんだ)



## 4. ラムダ式で設計する

#### ラムダ式を使った関心の分離

##### デザイン問題の探究

###### 主要な関心の分離

「何を」合計するかが異なるので、「何を合計するのか」の部分はメソッドから切り離すいい傾向 
こういう場合は、Strategyパターンの出番で、通常インターフェースとクラスを使用するが、ラムダ式は一歩進んだ設計手法がある 
```java
// Asset class
public class Asset {
  public enub AssetType {BOND, STOCK};
  private final AssetType type;
  private final int value;
  public Asset(final AssetType assetType, final int assetValue) {
    type = assetType;
    value = assetValue;

    }
  public AssetType getType() { return type; }
  public int getValue() { return value; }
}
```

```java
// AssetUtil class
//before
public static int totalAssertValues(final List<Asset> assets) {
  return assets.stream()
  .mapToInt(Asset::getValue)
  .sum();
}
```

これを関数型インターフェースを引数にとる１つのメソッドに統合する
after
```java
public static int totalAssertValues(final List<Asset> assets,
// java.util.function.Predicateインターフェース
final Predicate<Asset> asestSelector) {
  return assets.stream()
  .filter(assetSelector)
  .mapToInt(Asset::getValue)
  .sum();
}
```


#### ラムダ式を使った委譲

再利用の観点から、委譲は継承よりも優れた設計ツールである
委譲を使うと依存する実装を変化させやすくなり、異なる挙動をより動的にプラグインできる


##### 委譲の生成

責任の一部を他のクラスに委譲するのではなく、ラムダ式やメソッド参照に委譲できる
これにより、クラスの増減を抑えることができる
財務計算を行うCalculateNAVクラスの記述から始めよう
```java
public class CalculateNAV {
  public BigDecimal computeStockWorth(
    final String ticker, final int shares) {
    return priceFinder.apply(ticker).multiply(BigDecimal.valueOf(shares));
  }
  //pricefinderを使用する他のメソッド
}
```

computeStockWorth()は後ほど定義するpriceFinderから株価をリクエストし、その価格と保有株数から合計資産額を計算する
CalculateNAVは、priceFinderが返す株価をもとに利回りなどの計算を行う他のメソッドも持っている可能性がある
そのため、priceFinderはCalculateNAVの特定のメソッドの引数ではなく、クラスのフィールドに出現する

ここで、必要となるpriceFinderをどのような型のオブジェクトにするか決定しなければならない
銘柄コードを与えると、Webサービスから株価を取得するような機能が必要である

java.util.function.FunctionT, R関数型インターフェースはうってつけの軽量なソリューションにみえる
このabstractメソッドは、値を取り（型が異なる可能性のある）他の値を返すことができる
フィールドを定義する

```java
private Function<String, BigDecimal> priceFinder;
```

computeStockWorth()内ですでにこれはFunctionインターフェースのapply()を使用していた
ここではクラス内で直接実装するのではなく、コンストラクタ注入という手法でこのフィールドを初期化してみる
つまり依存性の注入と依存関係逆転の原則を使っている
```java
public CalculateNAV(final Function<String, BigDecimal> aPriceFinder) {
  priceFinder = aPriceFinder;
}
```





# 9.すべてをまとめて

## 関数型スタイルで成功するために実践するべきこと



### より宣言的に近く、命令型からより遠く

抽象化のレベルを引き上げなければいけない
行うべき各ステップの命令に集中するのではなく、より大きな、達成すべきゴールを宣言的に考えて表現しなければならない
たとえば、価格リストpricesを与えられて、その中から最大値を選ぶように依頼されたとする
まず、命令型で書くとこうなる
```java
int max = 0;
for (int price : prices) {
if(max < price) max = price;
}
```
ここで、宣言的に考えてみる 最大値を選ぶように伝える
```java
final int max = prices.stream()
                      .reduce(0, Math::max);
```
このスタイルから得られるメリットは、ただ単にコードの行数が減ることよりも遥かに大きいものがある
命令型ではpriceという具体的な変数が出ていたが、宣言的なコードの方ではpricesそのものを使うだけで抽象度のレベルが上がっている


### 不変性の尊重

状態変更が可能(ミュータブル)変数があると、正確な並列化が難しい
エラーを減らすための方法の1つは、可能な限りミュータブルを減らすこと


### 副作用の削減

参照透過性という考え方がある
プログラムの動作の正確性を損ねずに関数の呼び出しをその結果値で置き換えることができるということで、副作用を持たない
(何を言っているのかわからない)


### 関数型スタイルを採用

「まず動作させ、それから（すぐに）改善する」という格言に従って、素直に実装して、そのコードをすぐにリファクタリングをすればよい





