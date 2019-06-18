# Testing on the Toilet: Don’t Overuse Mocks

https://testing.googleblog.com/2013/05/testing-on-toilet-dont-overuse-mocks.html



> この記事は[Google Testing on the Toilet](http://googletesting.blogspot.com/2007/01/introducing-testing-on-toilet.html) (TotT) というエピソードの導入になります。TotTエピソードに関する[印刷バージョン](https://goo.gl/IEfP0)をダウンロードしてオフィスに掲示することができます。

自分のコードに対してテストを書くとき、**それらをモックすることによってコードの依存性を無視することは簡単に見えるかもしれません。** 


    public void testCreditCardIsCharged() {
      paymentProcessor = new PaymentProcessor(mockCreditCardServer);
      when(mockCreditCardServer.isServerAvailable()).thenReturn(true);
      when(mockCreditCardServer.beginTransaction()).thenReturn(mockTransactionManager);
      when(mockTransactionManager.getTransaction()).thenReturn(transaction);
      when(mockCreditCardServer.pay(transaction, creditCard, 500)).thenReturn(mockPayment);
      when(mockPayment.isOverMaxBalance()).thenReturn(false);
      paymentProcessor.processPayment(creditCard, Money.dollars(500));
      verify(mockCreditCardServer).pay(transaction, creditCard, 500);
    }

しかし、**モックを使わないことはときどきテストを簡単にかつ便利にする結果をもたらします** 


    public void testCreditCardIsCharged() {
      paymentProcessor = new PaymentProcessor(creditCardServer);
      paymentProcessor.processPayment(creditCard, Money.dollars(500));
      assertEquals(500, creditCardServer.getMostRecentCharge(creditCard));
    }

**モックを多用するといくつかの問題を引き起こすことがあります**


- **テストを理解することがより困難になる** コードを単純に利用するのではなく (例えば、テスト中のメソッドにいくつかの値を渡して結果を確認する)、モックに振る舞いを伝えるために追加のコードを含める必要があります。この余分なコードがあると、テストをしようとしている実際の意図が失われます。また本番のコード実装に慣れていないと、このコードを理解するのが難しくなります。
- **テストをメンテナンスすることがより困難になる** モックにどのように振る舞うかを伝えることでコードの実装の詳細がテストコードに漏れていきます。プロダクションにおける実装の詳細を変更するとテストにもこれらの変更を反映する必要があります。テストは一般的にコードの実装についてはほとんど知らないはずで、コードの公開インターフェースのテストに集中するべきです。
- **テストによってコードが正しく機能しているという保証が少なくなる** モックにどのように振る舞うかを伝えることで、テストで得られる唯一の保証は、モックが実際の動作と同じように動作すればコードが正しく動作するということです。これを保証することは難しく、実際の実装の振る舞いがあなたのモックと同期しなくなる可能性があるため、問題はコードが時間とともに変化するにつれて悪化していきます。モックの利用しすぎという兆候としてはクラスを1つまたは2つ以上モックしている、あるモックが1つか、2つ以上のメソッドがどのように振る舞うべきかを指定する必要があるようなものです。

**モックを使用しているテストを読み、テストを理解するためにコードを精神的に調べてみているのだとしたら、おそらくモックを使いすぎているということでしょう** 

**テストで本当の依存関係を利用できないことがあります** (例: とても遅い、ネットワークを介して話すなど) **けれども、モックを使うよりも適切な方法があるかもしれません** 例えば外部に接触しないローカルサーバー(例えばあなたがマシン上で起動したクレジットカードのサーバー) や偽の実装かもしれません。(例えば、インメモリクレジットカードサーバー)

*外部に接触しないサーバーを利用することに関しては、http://googletesting.blogspot.com/2012/10/hermetic-servers.html を読んでみてください。* 





