



# 1. Iterator

```java
// Aggregateインターフェース
public interface Aggregate {
  public abstract Iterator iterator();

}

// Iteratorインターフェース
public interface Iterator {
  public abstract boolean hasNext();
  public abstract Object next();
}

// Bookクラス
public class Book {
  private String name;
  public Book[String name] {
    this.name = name;
  }
  public String getName() {
    return name;
  }
}

// BookShelfクラス
public class BookShelf implements Aggregate {
  private Book[] books;
  private int last = 0;
  public BookShelf(int maxsize) {
    this.books = new Book[maxsize];
  }

  public Book getBookAt(int index) {
    return books[index];
  }

  public void appendBook(Book book) {
    this.books[last] = book;
    last++;
  }

  public int getLength() {
    return last;
  }

  public Iterator iterator() {
    return new BookShelfIterator(this);
  }
}

// BookShelfIteratorクラス
public class BookShelfIterator implements Iterator {
  private BookShelf bookShelf;
  private int index;

  public BookShelfIterator(BookShelf bookShelf) {
    this.bookShelf = bookShelf;
    this.index = 0;
  }

  public boolean hasNext() {
    if (index < bookShelf.getLength()) {
      return true;
    } else {
      return false;
    }
  }

  public Object next() {
    Book book = bookShelf.getBookAt(index);
    index++;
    return book;
  }
}

//Mainクラス
public class Main {
  public static void main(String[] args) {
    BookShelf bookShelf = new BookShelf(4);
    bookShelf.appendBook(new Book("Arround the World in 80 Days"));
    bookShelf.appendBook(new Book("Bible"));
    bookShelf.appendBook(new Book("Cinderella"));
    bookShelf.appendBook(new Book("Daddy-Long-Legs"));
    Iterator it = BookShelf.iterator();
    while (it.hasNext()) {
      Book book = (Book)it.next();
      System.out.println(book.getName());
    }
  }
}
```
Q.なぜ、Iteratorパターンを使うのか？ループならforで回せばいいじゃないか
A.大きな理由はIteratorを使うことで、 **実装とは切り離して** 数え上げができる
```java
while (it.hasNext()) {
  Book book = (book)it.next();
  System.out.println(book.getName());
}
```
ここで使われているのは、hasNextとnextというIteratorのメソッドだけ。
BookShelfの実装で使われているメソッドは呼び出されていない。
つまり、上のwhileループは、BookShelfの実装には **依存しない** ことになる
デザインパターンは再利用を促進するものである。
再利用化を促進するとは、クラスを **部品** として使えるようにするということであり、
1つの部品を修正しても他の部品の修正が少なくて済むということである。

そう考えると、上記のMainクラスでメソッドiteratorの戻り値をBookShelfIterator型の変数に代入せず、
iterator型の変数に代入している理由もわかる。BookShelfIteratorを変更しても、ここには影響しない
BookShelfIterator(具象クラス)のメソッドを使ってプログラミングを行うのではなく、
あくまでIteratorのメソッドを使ってプログラミングを行おうという姿勢を示している


# 2.Adapterパターン

プログラムの世界で、すでに提供されている物がそのまま使えない時に必要な形に変換してから利用することがよくある。
「すでに提供されている物」と「必要なもの」の間の「ずれ」を埋めるようなデザインパターンがAdapterパターン
クラスによるAdapterパターンを使ってサンプルプログムを読んでみよう
ここでつくるのは、与えられた文字列を (Hello) のように表示したり、**Hello** のように表示したりする簡単なもの
AdapterパターンはWrapperパターンと呼ばれることもある。
- 提供されているもの: Bannerクラス(showWithParen, showWithAster)
- 反感装置: PrintBannerクラス
- 必要なもの: Printインターフェース(printWeak, printStrong)

```java
// Bannerクラス
public class Banner {
  private String string;

  public Banner(String string) {
     this.string = string;
  }

  public void showWithParen() {
    System.out.println("(" + string + ")");
  }

  public void showWithAster() {
    System.out.println("*" + string + "*");
  }
}

// Printインターフェース
public interface Print {
  public abstract voidd printWeak();
  public abstract voidd printStrong();
}

//PrintBannerクラス
public class PrintBanner extends Banner implements Print {

  public PrintBanner(String string) {
    super(string);
  }

  public void printWeak() {
    showWithParen();
  }

  public void printStrong() {
    showWithAster();
  }
}
```
PrintBannerクラスが **アダプター** の役割を果たしている
用意されているBannerクラスを拡張(extends)して、showWithParen()とshowWithAster()を継承する
また、要求されているPrintインターフェースを実装(implements)してprintWeak()とprintStrong()を実装
```java
// Mainクラス
public class Main {
  public static void main(String[] string) {
    Print p = new PrintBanner("Hello");
    p.printWeak();
    p.printStrong();
  }
}
```

PrintBannerクラスのインスタンスをPrintインターフェースの型に代入している
このMainクラスはPrintインターフェースを使ってプログラミングしているので、
Bannerクラスや、showWithParen()や、showWithAster()がMainクラスから隠蔽されている
つまり、PrintBannerクラスをいくら修正しようと、Printインターフェースを実装してさえすればMainクラスには影響が出ない
さっきのIteratorと同じだね

上のサンプルプログラムは「クラスによる」Adapterパターンだった。
今度は「インスタンス」によるAdapterパターンを見てみよう
先ほどのサンプルプログラムでは「継承」を使ったが、今度は「委譲」を使ってみる
Printインターフェースをクラスとして定義する
```java
// Printクラス
public abstract class Print {
  public abstract void printWeak();
  public abstract void printStrong();
}

// PrintBannerクラス
public class PrintBanner extends Print {
  private Banner banner;

  public void printWeak() {
    banner.showWithParen();
  }

  public void printStrong() {
    banner.showWithAster();
  }
}
```

Adapterパターンの登場人物
- Target(対象)の役
  今必要となっているメソッドの役割。サンプルプログラムではPrintインターフェース（継承）や
  Printクラス(委譲)がこの役をつとめた。

- Client(依頼者)の役
  Target役のメソッドを使って仕事をする役
  サンプルプログラムではMainクラスがこれに該当する
  
- Adaptee(適合される側)役
  Adapt-er(適合する側)じゃなくてAdapt-ee(適合される側)。
  Adapteeはすでに用意されているメソッドを持っている役である。
  このAdaptee役のメソッドがTarget役のメソッドに一致していたら、次のAdapter役はいらなかったのだが...

- Adapterの役
  Adapterパターンの主役である
  Adaptee役のメソッドを使って何とかTarget役を満たそうというのがAdapterパターンの目的である
  サンプルプログラムではPrintBannerがこの役をつとめた
  クラスによるAdapterパターンの場合には、Adapter役は「継承」を使ってAdapter役を利用する
  一方、インスタンスによるAdapterパターンの場合には、「委譲」を使ってAdapter役を利用する
  
  **どんな時に使うのだろう**
  プログラミングするとき、既存クラスをつかうこともある。
  特にそのクラスが十分にテストされ、バグも少なくまた実際にこれまで使われてきた実績があるならば、なおさら。
  Adapterパターンは既存のクラスに一皮被せて必要とするクラスを作る。
  このパターンであればたとえバグが見つかったとしても、既存クラスにバグがないことがわかっているので
  Adapter役のクラスを重点的に調べれば良いことになりプログラムのチェックが楽になる
  

# 3. Template Method

## Template Methodとは何か

スーパークラスで処理の枠組みを定め、サブクラスでその具体的内容を定めるようなパターンをいう。

