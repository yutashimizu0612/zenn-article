---
title: "C#のインターフェースとは何か？利用側と実装側から理解する"
emoji: "🔌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["csharp", "インターフェース", "設計"]
published: false
---

## はじめに

C# .NETで開発をしていたら、インターフェースを目にしないことはないと思います。
しかし、インターフェースとは何か、インターフェースを利用する目的が何かを説明してくださいと言われたら、言葉に窮する人が多いのではないでしょうか。私自身そうでしたので、しっかり理解を深めるためにこの記事を書きました。

インターフェースのイメージや全体像を掴めるよう、身近な例とコードを交えながら、その仕組みと具体的なメリットを説明していきます。

## インターフェースとは

### エアコンを例にインターフェースのイメージを掴む

インターフェース（interface）とは、物事と物事が接する境界や接点のことです。

身近な例として、エアコンを考えてみます。

![利用者とエアコンの間にある、温度設定や風量変更のインターフェース](/images/csharp-interface-purpose/air-conditioner.png)

エアコンには、「電源を入れる」「設定温度を変える」「運転モードを変える」「風量を変える」といった、利用者向けの操作が用意されています。利用者はリモコンやスマートフォンのアプリを通じてこれらを操作します。

利用者が知る必要があるのは、どのような操作が用意されていて、それをどう使うかです。ボタンを押した後、エアコンが温度をどのように検知し、風量をどのように制御しているかといった内部動作を知る必要はありません。

一方、エアコン側から見ると、利用者向けに公開した操作は、それに対応する機能を提供するという約束です。「設定温度を変えられる」と公開する以上、エアコン側には、指定された温度に応じて動作する仕組みを用意しなければなりません。公開された操作は、利用者にとっては「利用できる機能」であり、エアコン側にとっては「実現しなければならない機能」です。

このように、利用者とエアコンの間には、利用者向けに公開された操作があります。この公開された操作が両者の接点、つまりインターフェースです。利用者はこの接点を通じて、内部の仕組みに触れることなくエアコンの機能を利用できます。

### ソフトウェア開発におけるインターフェース

エアコンでは、利用者に公開された操作と、その操作を実現する内部動作が分かれていました。ソフトウェア開発でも同じように、機能を使う側が知る必要のある操作と、その機能を実現する内部の処理を分けて考えます。C#では、この関係を契約として表す仕組みとして[`interface`](https://learn.microsoft.com/ja-jp/dotnet/csharp/language-reference/keywords/interface)が用意されています。

![利用側のコードと具象クラスの間に置かれたC#のインターフェース](/images/csharp-interface-purpose/csharp-interface.png)

`interface`は、利用できる操作を契約として定めます。これを実装すると宣言したクラスなどの型は、その契約に従って、定められたメンバーの実装を提供します。インターフェースが主に表すのは「何ができるか」であり、それを「どのように実現するか」は実装する型の責任です。

:::message
C#のインターフェースは、メンバーの既定の実装や静的メンバーなども定義できます。この記事では、メソッドの宣言に対して実装クラスが処理を提供する基本的な形を扱います。
:::

コードで確認してみます。

```cs
// インターフェース：メッセージを送信できるという契約を定める
public interface INotifier
{
    void Send(string message);
}

// INotifierの契約に従い、メール送信として実装する
public class EmailNotifier : INotifier
{
    public void Send(string message)
    {
        // 宛先を組み立て、SMTPサーバーに接続し、本文を整形して送信する…
        // といったメール送信に関する実装詳細がここに入る
        // 通知機能を利用する側が知る必要のない内部実装
    }
}

// 通知機能を利用する側
public void NotifyCompletion(INotifier notifier)
{
    notifier.Send("処理が完了しました");
}
```

![NotifyCompletionがINotifierを利用し、EmailNotifierが送信処理を実装する関係](/images/csharp-interface-purpose/notifier.png)

`INotifier`は、「メッセージを送信する」という契約を`Send()`として定めています。ここには、宛先をどのように組み立てるか、どのようにSMTPサーバーへ接続するかといった具体的な処理は書かれていません。

`EmailNotifier`は、`INotifier`を実装すると宣言することで、その契約を引き受けています。`INotifier`に`Send()`が定められているため、`EmailNotifier`は契約に従って`Send()`の具体的な処理を提供します。`INotifier`が定めた「メッセージを送信できる」という契約を、`EmailNotifier`はメール送信という方法で実現しているわけです。

`NotifyCompletion`が利用しているのは、具象クラスの`EmailNotifier`ではなく`INotifier`です。引数も`INotifier`として受け取っているため、`NotifyCompletion`は`EmailNotifier`を知りません。エアコンの利用者が内部動作を知らず、リモコンに用意された操作だけを使うのと同じように、`NotifyCompletion`は`INotifier`に公開された`Send()`だけを呼び出しています。

### 利用側と実装側では見え方が異なる

前節のコードでは、`NotifyCompletion`と`EmailNotifier`の間に、`INotifier`という一つの契約が置かれていました。これはどちら側から見ても同じ契約ですが、立場によって見え方が異なります。

:::message
ここでいう利用側と実装側は、開発者やチームの区分ではなく、コード上の立場を表しています。同じ開発者が両方のコードを書く場合でも、この二つの立場は存在します。
:::

利用側から見ると、インターフェースという契約は「どのような操作を呼び出せるか」を示すメニュー表のようなものです。前節の例では、`NotifyCompletion`は`INotifier`を通じて`Send()`を呼び出せます。

一方、実装側から見ると、この契約は「提供しなければならない操作の一覧」です。`EmailNotifier`は`INotifier`を実装すると宣言した以上、契約に含まれる`Send()`を提供しなければなりません。実装しなければ、契約を満たしていないためコンパイルエラーになります。

インターフェースは「実装を強制するもの」と説明されることがあります。これは事実ですが、実装側から見た特徴を述べたものです。利用側から見れば、同じインターフェースは、どのような操作を利用できるかを示すものです。

一つの見方として、同じインターフェースを、利用側からは「呼び出してよい操作の一覧」、実装側からは「提供しなければならない操作の一覧」と捉えることができます。私は、インターフェースを利用するコードなのか、それを実装するクラスなのかを区別することで、インターフェースが絡むコードを理解しやすくなりました。

## インターフェースを利用するメリット

ここまで、インターフェースが利用側と実装側の間に置かれた契約であることを説明しました。では、利用側が具象クラスではなくインターフェースに依存すると、何が嬉しいのでしょうか。この章では、インターフェースを利用するメリットを、利用しない場合と比較しながら見ていきます。

### ポリモーフィズムによって実装ごとの条件分岐を利用側からなくせる

典型的な利用例として、ECサイトの支払い処理を考えます。ECサイトには、クレジットカードや銀行振込など、複数の支払い方法があります。これらを扱う処理が、インターフェースを利用しない場合と利用する場合でどのように変わるかを見ていきます。

まず、インターフェースを利用していないコードを確認し、その問題をインターフェースによってどのように改善できるかを見ていきます。

**インターフェースを利用しない場合**

ここでは、在庫の確認から注文の確定までを担うクラスを`CheckoutService`とします。支払い方法ごとに処理内容は異なりますが、注文を受け付ける`CheckoutService`から見れば、いずれも注文金額を支払うための処理です。クレジットカードで支払うか、銀行振込で支払うかまで、`CheckoutService`が意識する必要は本来ありません。

しかし、インターフェースを用意せず、`CheckoutService`が支払い方法の具象クラスを直接利用する場合、`CheckoutService`自身が支払い方法を判定し、それぞれの具象クラスを呼び分けることになるでしょう。

```cs
public class CheckoutService
{
    private readonly CreditCardPaymentMethod _creditCardPaymentMethod;
    private readonly BankTransferPaymentMethod _bankTransferPaymentMethod;

    public CheckoutService(
        CreditCardPaymentMethod creditCardPaymentMethod,
        BankTransferPaymentMethod bankTransferPaymentMethod)
    {
        _creditCardPaymentMethod = creditCardPaymentMethod;
        _bankTransferPaymentMethod = bankTransferPaymentMethod;
    }

    public void Checkout(decimal orderTotal, PaymentType paymentType)
    {
        // ① 在庫の確認や注文データの作成（処理は省略）

        // ② 支払い処理
        if (paymentType == PaymentType.CreditCard)
            _creditCardPaymentMethod.Pay(orderTotal);
        else if (paymentType == PaymentType.BankTransfer)
            _bankTransferPaymentMethod.Pay(orderTotal);

        // ③ 注文の確定（処理は省略）
    }
}
```

`Checkout`メソッド内に、支払い方法ごとの条件分岐が入り込んでいます。今は支払い方法が2種類だけなので、それほど問題には見えないかもしれません。しかし、支払い方法が増えれば、この条件分岐も長くなります。

また、支払い方法に応じて処理を分ける必要があるのは、`Checkout`メソッドだけとは限りません。アプリケーション内の別の処理でも支払い方法による処理分けが必要になれば、その利用側にも同じような条件分岐が書かれます。このように、利用側が支払い方法を判定する形では、アプリケーションのあちこちに同じような条件分岐が増えていきます。その場合、支払い方法を追加するたびにすべての分岐箇所を確認して変更しなければならず、保守しづらくなります。

**インターフェースを利用する場合**

そこで、支払い方法に共通する契約を`IPaymentMethod`として定めます。`IPaymentMethod`は支払うためのメソッド`Pay`のみを持つシンプルなインターフェースです。

```cs
// インターフェース
public interface IPaymentMethod
{
    void Pay(decimal amount);
}

// インターフェースを実装する具象クラス①: クレジットカード支払い
public class CreditCardPaymentMethod : IPaymentMethod
{
    public void Pay(decimal amount)
    {
        // クレジットカードによる支払い処理
    }
}

// インターフェースを実装する具象クラス②: 銀行振込支払い
public class BankTransferPaymentMethod : IPaymentMethod
{
    public void Pay(decimal amount)
    {
        // 銀行振込による支払い処理
    }
}
```

`CreditCardPaymentMethod`と`BankTransferPaymentMethod`は異なるクラスですが、どちらも`IPaymentMethod`を実装しているため、どちらのオブジェクトも`IPaymentMethod`型として扱えます。そのため、`CheckoutService`はこれらの具象クラスに依存せず、支払い方法を`IPaymentMethod`として受け取って利用できます。

```cs
public class CheckoutService
{
    private readonly IPaymentMethod _paymentMethod;

    public CheckoutService(IPaymentMethod paymentMethod)
    {
        _paymentMethod = paymentMethod;
    }

    public void Checkout(decimal orderTotal)
    {
        // ① 在庫の確認や注文データの作成（処理は省略）

        // ② 支払い処理
        _paymentMethod.Pay(orderTotal);

        // ③ 注文の確定（処理は省略）
    }
}
```

![CheckoutServiceが具象クラスに直接依存する場合と、IPaymentMethodに依存する場合の比較](/images/csharp-interface-purpose/payment-comparison.png)

`CheckoutService`は、実際の支払い方法がクレジットカードなのか銀行振込なのかを判定せず、`Pay()`を呼び出すだけです。それでも、コンストラクターで渡されたオブジェクトが`CreditCardPaymentMethod`ならクレジットカードによる支払い処理、`BankTransferPaymentMethod`なら銀行振込による支払い処理が実行されます。このように、同じメソッドを呼び出しても、そのメソッドを呼び出すオブジェクトの種類によって異なる動作をする仕組みをポリモーフィズムと呼びます。

これにより、支払い方法ごとの条件分岐が`CheckoutService`からなくなり、`CheckoutService`が依存するのは`IPaymentMethod`だけになりました。他の利用側も`IPaymentMethod`を受け取るようにすれば、同じような条件分岐がアプリケーションのあちこちに増えていくことを防げます。

どの支払い方法を使うか決める処理自体は必要です。しかし、その判断は、利用者の選択を受け取って使用する支払い方法を決める側の責務です。インターフェースを利用しない例では`Checkout()`が`PaymentType`を受け取っていましたが、ここでは外部で選ばれた`IPaymentMethod`をコンストラクターで受け取ります。`CheckoutService`の責務は、在庫の確認から注文の確定までの流れを進めることであり、どの支払い方法を使うかを判定することではありません。両者を分けることで、`CheckoutService`は注文処理だけに集中できます。

### 新しい実装の種類を増やしやすい

これは、前節で説明したメリットを、実装を追加する場面から言い換えたものです。利用側が具象クラスではなくインターフェースに依存しているため、新しい実装を増やしても利用側を変更せずに済みます。

たとえば、クレジットカードや銀行振込のほかに、QRコード決済が追加されることになったとします。インターフェースを利用している場合は、`IPaymentMethod`を実装するクラスを新しく用意します。

```cs
public class QrCodePaymentMethod : IPaymentMethod
{
    public void Pay(decimal amount)
    {
        // QRコードによる支払い処理
    }
}
```

既存の`IPaymentMethod`を実装する具象クラスを追加すれば、`CheckoutService`など支払い機能を利用する側のコードには手を加える必要がありません。

**インターフェースを利用しない場合**

利用側が支払い方法ごとの条件分岐を持っている場合、QRコード決済を追加するには、その条件分岐にも処理を追加する必要があります。

```diff cs
 public void Checkout(decimal orderTotal, PaymentType paymentType)
 {
     // 在庫の確認や注文データの作成（処理は省略）

     if (paymentType == PaymentType.CreditCard)
         _creditCardPaymentMethod.Pay(orderTotal);
     else if (paymentType == PaymentType.BankTransfer)
         _bankTransferPaymentMethod.Pay(orderTotal);
+    else if (paymentType == PaymentType.QrCode)
+        _qrCodePaymentMethod.Pay(orderTotal);

     // 注文の確定（処理は省略）
 }
```

新しい支払い方法を追加するたびに利用側も変更することになり、変更箇所が広がりやすくなります。同じような支払い方法による条件分岐が複数箇所にあれば、そのすべてへQRコード決済の分岐を追加しなければなりません。また、既存の条件分岐を書き換えるため、誤ってクレジットカード決済や銀行振込の処理に影響を与える可能性もあります。

利用側が`IPaymentMethod`に依存していれば、新しい支払い方法を追加しても利用側を変更する必要がありません。利用側へ変更が広がらないため、新しい実装を追加しやすくなります。

### 実装漏れをコンパイルエラーで防げる

前節では、`IPaymentMethod`を実装する`QrCodePaymentMethod`を追加しました。このとき、契約に定められた`Pay()`を実装し忘れると、コンパイルエラーになります。インターフェースを実装するクラスには、契約に定められたメンバーの実装が強制されるためです。

```cs
// Pay()を実装していないため、コンパイルエラーになる
public class QrCodePaymentMethod : IPaymentMethod
{
}
```

インターフェースを利用しない元のコードでは、利用側に支払い方法の条件分岐が増えていきます。新しい支払い方法を追加するたびにすべての分岐を修正する必要がありますが、一部を修正し忘れてもコードとしては成立するため、コンパイルエラーでは気づけません。ここでは説明のために単純な例を示していますが、実際のアプリケーションで分岐が増えるほど、修正箇所を把握して漏れなく対応するのは難しくなり、実装漏れのリスクも高まります。

インターフェースを利用すると、利用側から支払い方法ごとの条件分岐そのものがなくなります。さらに、新しい具象クラスには`IPaymentMethod`の実装が強制されるため、必要な`Pay()`を実装し忘れればコンパイルエラーになります。人がすべての条件分岐を探して修正するのではなく、新しい具象クラスが契約を満たしているかをコンパイラに検査させられるため、実装漏れを仕組みとして防ぎやすくなります。

### 実装を差し替えやすい

インターフェースを利用すると、利用側を変更せずに実装を差し替えやすくなります。ここでは、アップロードされた文書の保存先を例に考えます。開発環境ではローカルへ保存し、本番環境ではAmazon S3へ保存したいとします。

まず、ファイルを保存するための契約を`IStorageService`として定めます。ローカルへ保存するクラスとS3へ保存するクラスは、どちらもこのインターフェースを実装します。

```cs
public interface IStorageService
{
    void Save(string fileName, byte[] content);
}

public class LocalStorageService : IStorageService
{
    public void Save(string fileName, byte[] content)
    {
        // ローカルディスクへ保存する処理
    }
}

public class S3StorageService : IStorageService
{
    public void Save(string fileName, byte[] content)
    {
        // Amazon S3へ保存する処理
    }
}
```

文書を扱う`DocumentService`は、具体的な保存先ではなく`IStorageService`に依存します。

```cs
public class DocumentService
{
    private readonly IStorageService _storageService;

    public DocumentService(IStorageService storageService)
    {
        _storageService = storageService;
    }

    public void Upload(string fileName, byte[] content)
    {
        _storageService.Save(fileName, content);
    }
}
```

`DocumentService`が知っているのは、`Save()`を呼び出せばファイルを保存できるという契約だけです。実際にローカルへ保存するのか、S3へ保存するのかは知りません。

そのため、`DocumentService`へ渡す具象クラスを変えるだけで、保存先を切り替えられます。

```diff cs
-IStorageService storageService = new LocalStorageService();
+IStorageService storageService = new S3StorageService();
 DocumentService documentService = new DocumentService(storageService);
```

差し替える前後のクラスは、どちらも`IStorageService`の契約を満たしています。`DocumentService`が依存する契約は変わらないため、`DocumentService`自体を修正する必要はありません。将来、保存先をAzure Blob Storageへ変更する場合も、`IStorageService`を実装する新しい具象クラスを用意すれば、`DocumentService`はそのまま利用できます。

**インターフェースを利用しない場合**

一方、インターフェースを利用せず、`DocumentService`が`LocalStorageService`へ直接依存するコードは次のようになります。

```cs
public class DocumentService
{
    private readonly LocalStorageService _storageService;

    public DocumentService(LocalStorageService storageService)
    {
        _storageService = storageService;
    }

    public void Upload(string fileName, byte[] content)
    {
        _storageService.Save(fileName, content);
    }
}
```

保存先をS3へ変更するには、`DocumentService`が依存する型を`S3StorageService`へ書き換えなければなりません。保存先の変更が、それを利用する`DocumentService`の変更にまで及んでしまいます。

インターフェースを間に置けば、利用側は変わらない契約に依存したまま、その契約を満たす実装だけを差し替えられます。このメリットが得られるのは、利用側が`LocalStorageService`や`S3StorageService`といった具象クラスに依存せず、`IStorageService`にだけ依存しているためです。

同じ考え方は、開発環境ではコンソール、本番環境では外部のログ監視サービスへログを出力する場合にも使えます。また、外部APIへ接続するクラスを、テスト時だけモックへ差し替えることもできます。このように、環境ごとに処理を使い分ける場合や、利用する外部サービスを変更する場合に、実装を差し替えやすいというメリットが役立ちます。

### テストしやすい

実装を差し替えやすいことは、テストのしやすさにもつながります。`CheckoutService`をテストするときに、クレジットカード会社のシステムへ接続して実際の決済を行うわけにはいきません。そこで、実際の決済を行わないテスト用の実装を用意します。

実際の開発では、Moqなどのモックライブラリにインターフェースを指定し、その契約を満たすテスト用オブジェクトを作ることも一般的です。ここでは、実装を差し替えていることが分かりやすいように`FakePaymentMethod`を手書きします。

```cs
public class FakePaymentMethod : IPaymentMethod
{
    public decimal PaidAmount { get; private set; }

    public void Pay(decimal amount)
    {
        PaidAmount = amount;
    }
}
```

`FakePaymentMethod`は`IPaymentMethod`の契約を満たしています。`CheckoutService`は`CreditCardPaymentMethod`などの具象クラスではなく`IPaymentMethod`に依存しているため、本番用の実装と同じように、このテスト用の実装を渡せます。

```cs
[Fact]
public void Checkout_注文金額で支払う()
{
    var paymentMethod = new FakePaymentMethod();
    var checkoutService = new CheckoutService(paymentMethod);

    checkoutService.Checkout(5_000m);

    Assert.Equal(5_000m, paymentMethod.PaidAmount);
}
```

本番用の実装をテスト用の実装へ差し替えられるため、外部のシステムやネットワークの状態に左右されず、`CheckoutService`の処理だけをテストできます。

**インターフェースを利用しない場合**

`CheckoutService`が具象クラスへ直接依存している場合、クレジットカード支払いの経路をテストするにも、本物の`CreditCardPaymentMethod`を使うことになります。すると、`Pay()`から外部通信へ処理が進みます。設定や認証情報がなく途中でエラーになるかもしれませんし、実際に外部へ通信してしまうかもしれません。いずれにしても、このままテストを実行するのは適切ではありません。

そこで、先ほど用意したテスト用の`FakePaymentMethod`を代わりに渡したいところですが、`CheckoutService`が要求しているのは`CreditCardPaymentMethod`です。両者は異なる型なので、`FakePaymentMethod`を渡すことはできません。`CheckoutService`と支払い処理が具象クラスによって密接につながっており、両者を切り離して`CheckoutService`だけを単体テストすることができません。

### 異なる型に共通の役割を持たせられる

C#では、一つのクラスが複数のクラスを継承することはできません（多重継承不可）。一方、一つのクラスが複数のインターフェースを実装することは可能です。つまり、インターフェースを使えば、一つのクラスに複数の役割や能力を持たせられます。これは、ここまでに見てきた具象実装を切り替える使い方とは少し異なります。

例として、お知らせ、ブログ記事、顧客の情報を管理するアプリケーションを考えてみます。いずれも作成・更新日時を記録しますが、お知らせとブログ記事には予約公開のための公開開始日時も必要です。さらに、お知らせには掲載を終了する有効期限も設定するものとします。

お知らせ、ブログ記事、顧客は、それぞれ異なる目的を持つ別のクラスです。しかし、日時を記録する処理から見ると、いずれも「作成・更新日時を記録する対象」という共通の役割を持っています。その役割を`IAuditable`として表します。公開開始日時、有効期限についても別々の契約として定義し、それぞれを必要とするクラスが実装します。

**インターフェース側**

```cs
// 作成・更新日時を記録する
public interface IAuditable
{
    DateTime CreatedAt { get; set; }
    DateTime UpdatedAt { get; set; }
}

// 公開開始日時を指定する
public interface IPublishable
{
    DateTime PublishAt { get; set; }
}

// 有効期限を指定する
public interface IExpirable
{
    DateTime ExpiresAt { get; set; }
}
```

**実装側**

`Announcement`、`BlogPost`、`Customer`は、それぞれ必要なインターフェースだけを実装します。

```cs
// お知らせ：作成・更新日時、公開開始日時、有効期限を持つ
public class Announcement : IAuditable, IPublishable, IExpirable
{
    // …略（タイトルや本文など）

    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
    public DateTime PublishAt { get; set; }
    public DateTime ExpiresAt { get; set; }
}

// ブログ記事：作成・更新日時と公開開始日時を持つ
public class BlogPost : IAuditable, IPublishable
{
    // …略（タイトルや本文など）

    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
    public DateTime PublishAt { get; set; }
}

// 顧客：作成・更新日時だけを持つ
public class Customer : IAuditable
{
    // …略（氏名など）

    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
}
```

**利用側**

例えば、保存時の共通処理で更新日時を設定する設計が考えられます。そこから更新対象の`IAuditable`を処理すれば、お知らせや顧客を更新する個別の処理では、日時の設定を繰り返し書かずに済みます。次のメソッドは、その共通処理のうち更新日時を設定する部分だけを示したものです。

`SetUpdatedAt()`は`IAuditable`だけを使うため、お知らせ、ブログ記事、顧客のどれに対しても、同じ処理で更新日時を設定できます。

```cs
public void SetUpdatedAt(IAuditable entity, DateTime now)
{
    entity.UpdatedAt = now;
}
```

次に、`IPublishable`の利用例として、公開開始日時を迎えているかを判定する処理を見てみます。現在日時と公開開始日時を比較するために、具体的なクラスが何かを意識する必要はありません。

```cs
public bool HasPublicationStarted(IPublishable item, DateTime now)
{
    return item.PublishAt <= now;
}
```

```cs
HasPublicationStarted(announcement, now);
HasPublicationStarted(blogPost, now);

// CustomerクラスはIPublishableを実装していないため、渡すとコンパイルエラー
// HasPublicationStarted(customer, now);
```

有効期限を過ぎているかの判定では、`IExpirable`を使います。

```cs
public bool HasExpired(IExpirable item, DateTime now)
{
    return item.ExpiresAt <= now;
}
```

```cs
HasExpired(announcement, now);

// BlogPostクラスとCustomerクラスはIExpirableを実装していないため、渡すとコンパイルエラー
// HasExpired(blogPost, now);
// HasExpired(customer, now);
```

`Announcement`は、更新日時を設定する処理からは`IAuditable`、公開開始日時を判定する処理からは`IPublishable`、期限切れを判定する処理からは`IExpirable`として扱われます。それぞれの利用側は、自分に必要な契約だけを使っています。

`Customer`には公開開始日時や有効期限が不要なので、それらの契約を実装する必要はありません。実装側は、自分に必要な機能や役割を定めたインターフェースだけを選び、それぞれの契約に従ってメンバーを実装します。

**インターフェースを利用しない場合**

各クラスに日時のプロパティを個別に書くだけでも、情報を持たせることはできます。ただし、作成・更新日時を記録したいすべてのクラスに、必要なプロパティを揃えることは開発者が確認しなければなりません。同名のプロパティを持っているだけでは、これらのクラスを`IAuditable`のような共通の型として受け取ることもできません。

`IAuditable`を実装するクラスとして宣言すれば、`CreatedAt`や`UpdatedAt`の実装が欠けている場合はコンパイルエラーになります。必要な役割の契約を選ぶことで、揃えるべきメンバーが明確になり、利用側もその契約で共通に扱えます。

共通の基底クラスを使う方法もありますが、すべての日時をそこへまとめると、Customerクラスにも不要な公開開始日時や有効期限を持たなくてはなりません。役割ごとに基底クラスを分けても、C#のクラスはそれらを複数継承できません。インターフェースなら、各クラスが必要な契約を組み合わせて実装できます。

## おわりに

ここまで見てきた通り、利用側が具象クラスではなくインターフェースに依存することで、さまざまなメリットが得られます。

ただし、必要のないところにまでインターフェースを適用すると、ソースコードが読みづらくなりますし、理解するための学習コストも増えます。どのようなメリットを得たいのかを意識し、必要なところで使うことが大切です。

この記事が、インターフェースのイメージを掴む手助けになれば幸いです。私自身、理解が十分でない点もあるかと思いますので、誤りや説明が不十分な点があれば、コメントでご指摘いただけるとありがたいです。

## 参考にした資料・実装

この記事の説明とサンプルコードを作成するにあたり、次の資料と実装を参考にしました。

- [classキーワード（C#リファレンス）](https://learn.microsoft.com/ja-jp/dotnet/csharp/language-reference/keywords/class)
- [`IPaymentMethod`](https://github.com/nopSolutions/nopCommerce/blob/develop/src/Libraries/Nop.Services/Payments/IPaymentMethod.cs)
