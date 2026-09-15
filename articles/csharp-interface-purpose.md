---
title: "C#のインターフェース入門：仕組みと使う理由を理解する"
emoji: "🔌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["csharp", "インターフェース", "設計"]
published: false
---

## はじめに

C# .NETで開発をしていたら、インターフェースを目にしないことはないと思います。

しかし、「インターフェースとは何か？」「インターフェースを利用する目的は何か？」と聞かれたら、返答に困る人が多いのではないでしょうか。私自身そうでしたので、しっかり理解を深めるためにこの記事を書きました。

インターフェースのイメージや全体像を掴めるよう、身近な例とコードを交えながら、その仕組みと具体的なメリットを説明していきます。

## インターフェースとは

### エアコンを例にインターフェースのイメージを掴む

インターフェース（interface）とは、物事と物事が接する境界や接点のことです。

身近な例として、エアコンを考えてみます。

![利用者とエアコンの間にある、電源・設定温度・運転モード・風量の操作を定めたインターフェース](/images/csharp-interface-purpose/air-conditioner.png)

エアコンには、「電源を入れる」「設定温度を変える」「運転モードを変える」「風量を変える」といった、利用者向けの操作が用意されています。利用者はリモコンやスマートフォンのアプリを通じてこれらを操作します。

利用者が知る必要があるのは、どのような操作が用意されていて、それをどう使うかです。ボタンを押した後、エアコンが温度をどのように検知し、風量をどのように制御しているかといった内部動作を知る必要はありません。

一方、エアコン側から見ると、利用者向けに公開した操作は、対応する機能を提供するという約束です。「設定温度を変えられる」と公開する以上、エアコンには、室温を指定された温度に近づける機能を用意する必要があります。公開された操作は、利用者にとっては「利用できる機能」であり、エアコン側にとっては「実現しなければならない機能」です。

このように、利用者とエアコンの間には、利用者向けに公開された操作があります。この公開された操作が両者の接点、つまりインターフェースです。利用者はこの接点を通じて、内部の仕組みに触れることなくエアコンの機能を利用できます。

### ソフトウェア開発におけるインターフェース

ソフトウェア開発でもエアコンと同じように、利用側に公開する操作と、その操作を提供する実装側の処理を分けて考えることがあります。

C#では、利用側に公開する操作を[`interface`](https://learn.microsoft.com/ja-jp/dotnet/csharp/language-reference/keywords/interface)に定め、実装するクラスがそれらの操作に対応する処理を提供します。

![利用側のコードと具象クラスの間に置かれたC#のインターフェース](/images/csharp-interface-purpose/csharp-interface.png)

`interface`は、利用できる操作を契約として定めます。これを実装すると宣言したクラスは、その契約に従って、定められたメンバーの実装を提供します。

つまり、インターフェースは「何ができるか」を表し、それを実装するクラスは「どのように実現するか」は表すということになります。

:::message
C#のインターフェースには、メソッドやプロパティのほかにも、さまざまなメンバーを定義できます。また、メンバーに既定の実装を持たせることもできます。ただし、この記事ではそれらには踏み込まず、メソッドの宣言を中心に説明します。
:::

通知処理を例に、コードで見てみます。

![NotifyCompletionがINotifierを利用し、EmailNotifierが送信処理を実装する関係](/images/csharp-interface-purpose/notifier.png)

まず、通知するための操作を定める`INotifier`インターフェースに、メッセージを送信するためのメソッド`Send()`を定めます。

```cs
public interface INotifier
{
    void Send(string message);
}
```

`INotifier`には、宛先の組み立て方やSMTPサーバーへの接続方法といった具体的な処理は書かれていません。「メッセージを送信できる」という契約だけを定めています。

次に、実装側となる`EmailNotifier`を定義します。

```cs
public class EmailNotifier : INotifier
{
    public void Send(string message)
    {
        // 宛先を組み立て、SMTPサーバーに接続し、本文を整形して送信する…
        // といったメール送信に関する実装詳細がここに入る
    }
}
```

`EmailNotifier`は、`INotifier`を実装すると宣言することで、その契約を引き受けています。`INotifier`に`Send()`が定められているため、契約に従って具体的な処理を提供しなければなりません。ここでは処理を省略していますが、メッセージをメールで送信するための処理を記述する想定です。

最後に、通知処理を利用する側の`NotifyCompletion`を確認します。

```cs
public void NotifyCompletion(INotifier notifier)
{
    notifier.Send("処理が完了しました");
}
```

注目したいのは、利用側の`NotifyCompletion`が具象クラスの`EmailNotifier`ではなく`INotifier`を利用していることです。引数を`INotifier`として受け取っているため、`NotifyCompletion`は`EmailNotifier`を知りません。

エアコンの利用者が内部動作を知らず、リモコンに用意された操作だけを使うのと同じように、`NotifyCompletion`は`INotifier`に公開された`Send()`だけを呼び出しています。

### 利用側と実装側では見え方が異なる

前節のコードでは、`NotifyCompletion`と`EmailNotifier`の間に、`INotifier`という一つの契約が置かれていました。これはどちら側から見ても同じ契約ですが、立場によって見え方が異なります。

:::message
ここでいう利用側と実装側は、開発者やチームの区分ではなく、コード上の立場を表しています。同じ開発者が両方のコードを書く場合でも、この二つの立場は存在します。
:::

利用側から見ると、インターフェースという契約は「どのような操作を呼び出せるか」を示すメニュー表のようなものです。前節の例では、`NotifyCompletion`は`INotifier`を通じて`Send()`を呼び出せました。

一方、実装側から見ると、この契約は「提供しなければならない操作の一覧」です。`EmailNotifier`は`INotifier`を実装すると宣言した以上、契約に含まれる`Send()`を提供しなければなりません。実装しなければ、契約を満たしていないためコンパイルエラーになります。

インターフェースは「実装を強制するもの」と説明されることがあります。これは事実ですが、実装側から見た特徴を述べたものと言えるでしょう。利用側から見れば、同じインターフェースは、どのような操作を利用できるかを示すものです。

これは絶対的な捉え方ではありませんが、私は、利用側では「呼び出してよい操作」、実装側では「提供しなければならない操作」と区別することで、インターフェースが使われているコードを理解しやすくなりました。

## インターフェースを利用するメリット

ここまで、インターフェースが利用側と実装側の間に置かれた契約であることを説明しました。

では、利用側が具象クラスに直接依存するのではなく、インターフェースに依存すると、何が嬉しいのでしょうか。

ここからは、インターフェースを利用するメリットを、利用しない場合と比較しながら見ていきます。

### 1. ポリモーフィズムによって利用側の条件分岐をなくせる

典型的な利用例として、クレジットカードや銀行振込など、複数の支払い方法を扱うECサイトの注文処理を考えます。ここでは、支払い処理にインターフェースを導入し、複数の支払い方法を共通に扱えるようにします。

まずは具象クラスを直接利用するコードを確認し、その問題をインターフェースによってどのように改善できるかを見ていきます。

**インターフェースを利用しない場合**

![CheckoutServiceが支払い方法の具象クラスに直接依存する場合](/images/csharp-interface-purpose/payment-without-interface.png)

支払い方法ごとに`Pay()`の具体的な処理内容は異なりますが、`CheckoutService`から見れば、いずれも注文金額を支払うための処理です。クレジットカードで支払うか、銀行振込で支払うかを支払い処理を利用する側である`CheckoutService`が意識する必要は本来ありません。

しかし、インターフェースを用意せず、`CheckoutService`が支払い方法の具象クラスを直接利用する場合、`CheckoutService`自身が適切な支払い方法を判定・選択し、それぞれの具象クラスを呼び分けることになってしまいます。

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

`Checkout`メソッド内に、支払い方法ごとの条件分岐が入り込んでいることがわかります。今は支払い方法が2種類だけなので、それほど問題には見えないかもしれません。しかし、支払い方法が増えれば、この条件分岐は長くなります。

また、支払い方法による条件分岐が必要なのは、`Checkout`メソッドだけとは限りません。アプリケーション内の別の処理でも支払い方法による処理分けが必要になれば、そこにも同じような条件分岐が書かれます。

利用側が支払い方法を判定する今の形では、アプリケーションのあちこちに同じような条件分岐が増えていきます。その場合、支払い方法を追加するたびにすべての分岐箇所を確認して変更しなければならず、保守しづらくなります。

**インターフェースを利用する場合**

![CheckoutServiceがIPaymentMethodを通じて支払い方法を利用する場合](/images/csharp-interface-purpose/payment-with-interface.png)

そこで、`CheckoutService`からは、どの支払い方法も「指定された金額を支払えるもの」として扱えるようにします。そのために、支払うためのメソッド`Pay()`を`IPaymentMethod`インターフェースに定めます。

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

`CreditCardPaymentMethod`と`BankTransferPaymentMethod`は異なるクラスですが、どちらも`IPaymentMethod`を実装しているため、どちらのオブジェクトも`IPaymentMethod`型として扱えます。

そのため、`CheckoutService`はこれらの具象クラスに依存せず、支払い方法を`IPaymentMethod`として受け取って利用できます。

```cs
// 利用側は、具象クラスではなくIPaymentMethodだけに依存する
// CreditCardPaymentMethodやBankTransferPaymentMethodは直接利用しない
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

`CheckoutService`は、実際の支払い方法がクレジットカードなのか銀行振込なのかを判定せず、`Pay()`を呼び出すだけです。`_paymentMethod`の実体が`CreditCardPaymentMethod`ならクレジットカードによる支払い処理が、`BankTransferPaymentMethod`なら銀行振込による支払い処理が実行されます。

このように、同じメソッドを呼び出しても、そのメソッドを呼び出すオブジェクトの種類によって異なる動作をする仕組みをポリモーフィズムと呼びます。

`IPaymentMethod`インターフェースを導入したことで、`CheckoutService`から支払い方法ごとの条件分岐がなくなり、`CheckoutService`が依存するのは`IPaymentMethod`だけになりました。アプリケーション内のほかの場所で支払い処理を呼び出す場合も、支払い方法を`IPaymentMethod`として受け取るようにすれば、同じような条件分岐がアプリ内のあちこちに増えていくことを防げます。

:::message
購入者が選んだ支払い方法に応じて、どちらの実装を使うかを決める処理自体がなくなるわけではありません。その処理をどこへ置くかにはいくつかの方法がありますが、ここでは扱いません。

重要なのは、支払い処理の利用側を`IPaymentMethod`だけに依存させることです。これにより、クレジットカード、銀行振込、QRコード決済のどれを使うかという判断が個々の利用側へ散らばらず、実装を選択する一か所にまとめられます。
:::

### 2. 新しい実装の種類を増やしやすい

このメリットは、前節で説明した「利用側から条件分岐をなくせる」というメリットを、実装の追加という別の観点から見たものです。前節の支払い処理を引き続き例として使います。

クレジットカードや銀行振込に加えて、QRコード決済へ対応することになったとします。インターフェースを利用している場合は、`IPaymentMethod`を実装するクラスを新しく用意します。

![IPaymentMethodを実装する支払い方法を追加しても、利用側とインターフェースは変更しない](/images/csharp-interface-purpose/payment-add-implementation.png)

```cs
public class QrCodePaymentMethod : IPaymentMethod
{
    public void Pay(decimal amount)
    {
        // QRコードによる支払い処理
    }
}
```

`IPaymentMethod`を実装する具象クラスを追加しても、`CheckoutService`のコードには手を加える必要がありません。`CheckoutService`は支払い方法の具象クラスに依存せず、`IPaymentMethod`だけに依存しているからです。

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

新しい支払い方法を追加するたびに利用側も変更することになり、変更箇所が広がりやすくなります。同じような支払い方法による条件分岐が複数箇所にあれば、そのすべてへQRコード決済の分岐を追加しなければなりません。また、既存のソースコードの修正が必要なため、誤ってクレジットカード決済や銀行振込の処理に影響を与える可能性もあります。

`CheckoutService`のように支払い処理を実行する側が`IPaymentMethod`に依存していれば、新しい支払い方法を追加しても、そのコードを変更する必要はありません。変更が支払い処理を利用する各所へ広がらないため、新しい実装を追加しやすくなります。

### 3. 実装漏れをコンパイルエラーで防げる

前節では、`IPaymentMethod`を実装する`QrCodePaymentMethod`を追加しました。

このとき、インターフェースに定められた`Pay()`を実装し忘れると、コンパイルエラーになります。インターフェースを実装するクラスには、定められたメンバーの実装が強制されるためです。

```cs
public interface IPaymentMethod
{
    void Pay(decimal amount);
}

// Pay()を実装していないため、コンパイルエラーになる
public class QrCodePaymentMethod : IPaymentMethod
{
}
```

一方、インターフェースを利用せず、利用側が支払い方法ごとの条件分岐を持つ場合はどうなるでしょうか。

当たり前のことではありますが、コンパイラは、既存のコードにQRコード決済の条件分岐を書き忘れたことを検出できません。

```cs
public void Checkout(decimal orderTotal, PaymentType paymentType)
{
    if (paymentType == PaymentType.CreditCard)
        _creditCardPaymentMethod.Pay(orderTotal);
    else if (paymentType == PaymentType.BankTransfer)
        _bankTransferPaymentMethod.Pay(orderTotal);

    // QRコード決済の分岐がなくてもコンパイルは通る
}
```

つまり、新しい支払い方法を追加するたびに、開発者がすべての条件分岐を探して修正する必要があります。ある分岐箇所への追加を忘れても、既存のコードはコンパイルできてしまうため、コンパイラは修正漏れを検出できません。実際のアプリケーションで分岐が増えるほど、修正箇所を把握して漏れなく対応するのは難しくなります。

インターフェースを利用すると、`CheckoutService`のように支払い処理を実行する側から、支払い方法ごとの条件分岐がなくなります。さらに、新しい具象クラスには`IPaymentMethod`の実装が強制されるため、必要な`Pay()`を実装し忘れればコンパイルエラーになります。

人がすべての条件分岐を探して修正するのではなく、新しい具象クラスが契約を満たしているかをコンパイラに検査させられるため、実装漏れを仕組みとして防ぎやすくなります。

### 4. 利用側を変更せずに実装を差し替えられる

利用側が具象クラスではなくインターフェースだけに依存することで得られるメリットの一つが、実装を差し替えやすくなることです。同じインターフェースを実装するクラスであれば、利用側を変更せずに入れ替えられます。

実装を差し替える例として、アップロードされた文書の保存先を取り上げます。この例では、開発環境ではローカルへ保存し、本番環境ではAmazon S3へ保存するとします。

![IStorageServiceの実装を差し替えても、DocumentServiceとインターフェースは変更しない](/images/csharp-interface-purpose/storage-switch-implementation.png)

まず、ファイルを保存するための契約を`IStorageService`として定めます。

ローカルへ保存するクラスとS3へ保存するクラスは、どちらもこのインターフェースを実装します。

```cs
// インターフェース: ファイルを保存するための操作を定める
public interface IStorageService
{
    void Save(string fileName, byte[] content);
}

// 実装クラス①: ローカルディスクへ保存する
public class LocalStorageService : IStorageService
{
    public void Save(string fileName, byte[] content)
    {
        // ローカルディスクへ保存する処理
    }
}

// 実装クラス②: Amazon S3へ保存する
public class S3StorageService : IStorageService
{
    public void Save(string fileName, byte[] content)
    {
        // Amazon S3へ保存する処理
    }
}
```

次に、この保存機能を利用するクラスとして、文書を扱う`DocumentService`を用意します。

`DocumentService`は、Amazon S3といった具体的な保存先ではなく`IStorageService`に依存させます。

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

そのため、必要に応じて`DocumentService`へ渡す具象クラスを変えるだけで、保存先を切り替えられます。

```diff cs
-IStorageService storageService = new LocalStorageService();
+IStorageService storageService = new S3StorageService();
 DocumentService documentService = new DocumentService(storageService);
```

`LocalStorageService`と`S3StorageService`は、どちらも`IStorageService`の契約通りに実装がされています。`DocumentService`が依存する先は`IStorageService`のまま変わらないため、`DocumentService`自体を修正する必要はありません。

将来、保存先をAzure Blob Storageへ変更するといった場合も、`IStorageService`を実装する新しい具象クラスを用意すれば、利用側の`DocumentService`には変更を入れずに保存先を切り替えられます。

**インターフェースを利用しない場合**

一方、インターフェースを利用せず、`DocumentService`が`LocalStorageService`へ直接依存するコードは次のようになります。

```cs
public class DocumentService
{
    // IStorageServiceではなく、具象クラスのLocalStorageServiceに直接依存してしまっている
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

このコードでは、`DocumentService`が`LocalStorageService`という具象クラスに直接依存しています。

そのため、保存先をS3へ変更するには、依存する型を`S3StorageService`へ書き換えなければなりません。本来は保存先という実装詳細を意識する必要がない利用側の`DocumentService`にまで、修正が及んでしまいます。

利用側が`LocalStorageService`や`S3StorageService`といった具象クラスではなく、`IStorageService`だけに依存していれば、利用側のコードは変わりません。

`LocalStorageService`と`S3StorageService`のどちらの実装を使うかの選択は`DocumentService`の外側で行うため、保存先を切り替えるための処理が、保存機能を利用するコードの各所に散らばることもありません。

同じ考え方は、開発環境ではコンソール、本番環境では外部のログ監視サービスへログを出力する場合にも使えます。また、外部APIへ接続するクラスを、テスト時だけモックへ差し替えることもできます。

このように、環境ごとに処理を使い分ける場合や、利用する外部サービスを変更する場合にも、インターフェースが役立ちます。

### 5. テストしやすい

利用側を変更せずに実装を差し替えられることは、テストのしやすさにもつながります。

ここでは、最初に取り上げた支払い処理のサンプルに戻り、`CheckoutService`をテストする例を取り上げます。

テストのたびにクレジットカード会社のシステムへ接続し、実際の決済を行うわけにはいきません。そこで、`IPaymentMethod`の本番用実装を、実際の決済を行わないテスト用実装へ差し替えます。

![IPaymentMethodの本番用実装をテスト用実装へ差し替えても、CheckoutServiceとインターフェースは変更しない](/images/csharp-interface-purpose/payment-test-implementation.png)

実際の開発では、Moqなどのモックライブラリにインターフェースを指定し、その契約を満たすテスト用オブジェクトを作ることが一般的だと思います。

ここでは、実装を差し替えていることが分かりやすいように`FakePaymentMethod`を手書きしています。

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

`FakePaymentMethod`は`IPaymentMethod`の契約を満たしています。

`CheckoutService`は`CreditCardPaymentMethod`などの具象クラスではなく`IPaymentMethod`に依存しているため、本番用の実装と同じように、テスト用の実装である`FakePaymentMethod`を渡せます。

```cs
[Fact]
public void Checkout_注文金額で支払う()
{
    const decimal orderTotal = 5000;
    var paymentMethod = new FakePaymentMethod();
    var checkoutService = new CheckoutService(paymentMethod);

    checkoutService.Checkout(orderTotal);

    Assert.Equal(orderTotal, paymentMethod.PaidAmount);
}
```

本番用の実装をテスト用の実装へ差し替えられるため、外部のシステムに左右されず、`CheckoutService`の処理だけをテストできます。

**インターフェースを利用しない場合**

`CheckoutService`が具象クラスへ直接依存している場合、`CheckoutService`の処理だけをテストしたくても、本物の`CreditCardPaymentMethod`を使うことになります。

すると、`Pay()`から外部通信へ処理が進みます。

設定や認証情報がなく途中でエラーになるかもしれませんし、実際に外部へ通信してしまうかもしれません。いずれにしても、このままテストを実行するのは適切ではありません。

本来は、先ほど用意した`FakePaymentMethod`を渡すか、`IPaymentMethod`をもとにモックライブラリで作ったテスト用オブジェクトへ差し替えたいところです。

しかし、`CheckoutService`が要求しているのは`CreditCardPaymentMethod`です。両者は異なる型なので、`FakePaymentMethod`を渡すことはできません。

つまり、`IPaymentMethod`という共通の差し込み口がないため、この`FakePaymentMethod`へ差し替えて`CheckoutService`だけを単体テストすることができません。

### 6. 異なる型に共通の役割を持たせられる

C#では、一つのクラスが複数のクラスを継承することはできません。一方、一つのクラスが複数のインターフェースを実装することは可能です。

そのため、C#では一つのクラスに複数の役割を持たせたいとき、それぞれの役割を表すインターフェースを組み合わせて実装します。これは、ここまでに見てきた具象クラスを切り替える使い方とは少し異なります。

例として、お知らせ、ブログ記事、顧客の情報を管理するアプリケーションを考えてみます。

![Announcement、BlogPost、Customerが必要なインターフェースだけを組み合わせて実装する関係](/images/csharp-interface-purpose/interface-role-combination.png)

**インターフェース側**

この例では、「作成・更新日時を記録する対象」「公開できる対象」「有効期限を設定できる対象」という三つの役割を、それぞれ別のインターフェースとして定めます。

```cs
// ※ サンプルを簡潔にするため、各役割に必要なメンバーを日時のプロパティだけに絞っています

// 作成・更新日時を記録する対象
public interface IAuditable
{
    DateTime CreatedAt { get; set; }
    DateTime UpdatedAt { get; set; }
}

// 公開できる対象
public interface IPublishable
{
    DateTime PublishAt { get; set; }
}

// 有効期限を設定できる対象
public interface IExpirable
{
    DateTime ExpiresAt { get; set; }
}
```

お知らせ、ブログ記事、顧客は、それぞれ異なる目的を持つ別のクラスです。

しかし、日時を記録する処理から見ると、いずれも「作成・更新日時を記録する対象」という共通の役割を持っています。

その役割を表すのが`IAuditable`です。

**実装側**

いずれも`IAuditable`を実装しますが、お知らせとブログ記事だけが、予約公開に必要な`IPublishable`も実装します。さらに、お知らせは、掲載を終了する有効期限を持つため`IExpirable`も実装します。

このように、`Announcement`、`BlogPost`、`Customer`は、それぞれ必要なインターフェースだけを組み合わせます。

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

ここでは、保存時の共通処理で更新日時を設定するものとします。更新対象を`IAuditable`として受け取れば、それぞれを更新する個別の処理で、日時の設定を繰り返し書かずに済みます。

次のメソッドは、その共通処理のうち、更新日時を設定する部分だけを示したものです。

`SetUpdatedAt()`は`IAuditable`だけを使うため、お知らせ、ブログ記事、顧客のどれに対しても、同じ処理で更新日時を設定できます。

```cs
public void SetUpdatedAt(IAuditable entity, DateTime now)
{
    entity.UpdatedAt = now;
}
```

次に、`IPublishable`の利用例として、公開開始日時を迎えているかを判定する処理を見てみましょう。現在日時と公開開始日時を比較するために、具体的なクラスが何かを意識する必要はありません。

この処理に渡せるのは、`IPublishable`を実装しているクラスだけです。`Announcement`と`BlogPost`は渡せますが、`IPublishable`を実装していない`Customer`を渡すとコンパイルエラーになります。

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

`Announcement`は、更新日時を設定する処理からは`IAuditable`、公開開始日時を判定する処理からは`IPublishable`、期限切れを判定する処理からは`IExpirable`として扱われます。

それぞれの利用側のクラスは、自分に必要な契約だけを使っています。例えば、`Customer`には公開開始日時や有効期限が不要なので、それらの契約を実装する必要はありません。

実装側は、自分に必要な機能や役割を定めたインターフェースだけを選び、それぞれの契約に従ってメンバーを実装します。

**インターフェースを利用しない場合**

各クラスに日時のプロパティを個別に書くだけでも、必要な情報を持たせることはできます。

ただし、作成・更新日時を記録したいすべてのクラスに必要なプロパティが揃っているかは、開発者が確認しなければなりません。また、複数のクラスが同名のプロパティを持っているだけでは、それらを`IAuditable`のような共通の型として受け取ることもできません。

`IAuditable`を実装するクラスとして宣言すれば、`CreatedAt`や`UpdatedAt`の実装が欠けている場合はコンパイルエラーになります。必要な役割の契約を選ぶことで、揃えるべきメンバーが明確になり、利用側もその契約で共通に扱えます。

共通の基底クラスを使う方法もありますが、すべての日時を共通の基底クラスにまとめると、各クラスに不要な実装を強いることになってしまいます。例えば、`Customer`にも不要な公開開始日時や有効期限を持たせることになります。

では、役割ごとに基底クラスを分け、各クラスに必要な基底クラスだけを継承させればよいのでしょうか。しかし、C#では一つのクラスが複数のクラスを継承することはできません。

役割ごとに分けた契約を必要な分だけ組み合わせるには、複数実装できるインターフェースを使います。

## おわりに

ここまで見てきた通り、利用側が具象クラスではなくインターフェースに依存することで、さまざまなメリットが得られます。

ただし、必要のないところにまでインターフェースを適用すると、ソースコードが読みづらくなりますし、理解するための学習コストも増えます。どのようなメリットを得たいのかを意識し、必要なところで使うことが大切です。

この記事が、インターフェースのイメージを掴む手助けになれば幸いです。私自身、理解が十分でない点もあるかと思いますので、誤りや説明が不十分な点があれば、コメントでご指摘いただけるとありがたいです。

## 参考にした資料・実装

この記事の説明とサンプルコードを作成するにあたり、次の資料と実装を参考にしました。

- [classキーワード（C#リファレンス）](https://learn.microsoft.com/ja-jp/dotnet/csharp/language-reference/keywords/class)
- [`IPaymentMethod`](https://github.com/nopSolutions/nopCommerce/blob/develop/src/Libraries/Nop.Services/Payments/IPaymentMethod.cs)
