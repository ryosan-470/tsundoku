# Testing on the Toilet: Know Your Test Doubles

https://testing.googleblog.com/2013/07/testing-on-toilet-know-your-test-doubles.html

## 訳注: テストダブルとは?

ソフトウェアテストにおける、モックやスタブといったテスト対象が依存しているものを置き換えるものの総称。ダブルは代役とかそういう意味を持つとのこと。
日本語での文献でわかりやすいのは以下のものだろうか。

- http://bliki-ja.github.io/TestDouble/
- http://blog.fujimisakari.com/test_double_pattern/

**テストダブルとは、テスト内で実際のオブジェクトの代わりになることのオブジェクトのこと**で、スタント役が映画の俳優の代役をやるようなものです。これらは一般的にすべて「モック」と呼ばれていますが、テストダブルの種類はそれぞれの用途が異なるため、区別することが重要です。**テストダブルにおける一般的な種類はスタブ、モック、フェイクがあります** 

**スタブ** はロジックを持たず、返すように指定したものだけを返却します。**スタブは、テスト中のコードを特定の状態にするために特定の値を返すオブジェクトが必要なときに利用できます。** 通常手でスタブを書くことは簡単ですが、モッキングフレームワークを利用することで[Boilerplate](https://en.wikipedia.org/wiki/Boilerplate_code)をへらすことができます。


    // モッキングフレームワークから作成されたスタブを引数にとる
    AccessManager accessManager = new AccessManager(stubAuthenticationService);
    // ユーザーは認証サービスがfalseを返したときはアクセスできない
    when(stubAuthenticationService.isAuthenticated(USER_ID)).thenReturn(false);
    assertFalse(accessManager.userHasAccess(USER_ID));
    // ユーザーは認証サービスがtrueを返したときはアクセスできる
    when(stubAuthenticationService.isAuthenticated(USER_ID)).thenReturn(true);
    assertTrue(accessManager.userHasAccess(USER_ID));

**モック** はそれがどのように呼ばれるべきかについて期待を持っており、それが呼ばれないのであればテストは失敗するべきです。**モックはオブジェクト間の相互作用をテストするために利用されており**、他に目に見える状態変化がない場合や、検証できる結果を返す場合に役立ちます。(例: あなたのコードがディスクを読み出し、1つ以上のディスクから読み出していないということを保証したいというときに、モックを用いてメソッドが1度しか読み込みを呼び出していないということを検証できます)


    // モッキングフレームワークによって作成されたモックを引数に取る
    AccessManager accessManager = new AccessManager(mockAuthenticationService);
    accessManager.userHasAccess(USER_ID);
    // accessManager.userHasAccess(USER_ID) が mockAuthenticationService.isAuthenticated(USER_ID) を呼び出さない
    // または1度以上、呼び出される場合、テストは失敗するべき。
    verify(mockAuthenticationService).isAuthenticated(USER_ID);

**フェイク** はモッキングフレームワークを利用しません。本物の実装の振る舞いをする軽量API実装ですが、本番には適していないものになります。(例: インメモリデータベース) **フェイクはテスト内で実際の実装を利用できないときに用います。** (例: 本物の実装がとても遅い、またはネットワーク越しにやりとりする)  フェイクは実際の実装を所有する人やチームによって作成され管理されるため、頻繁に自分でフェイクを書く必要はありません。


    // フェイクを作成することは高速かつ容易である
    AuthenticationService fakeAuthenticationService = new FakeAuthenticationService();
    AccessManager accessManager = new AccessManager(fakeAuthenticationService);
    // ユーザーは認証サービスがそのユーザーを知らないためアクセス権を持たない
    assertFalse(accessManager.userHasAccess(USER_ID));
    // ユーザーが認証サービスに追加されたあと、アクセス権を持つ
    fakeAuthenticationService.addAuthenticatedUser(USER_ID);
    assertTrue(accessManager.userHasAccess(USER_ID));


> 「テストダブル」という用語はxUnit Test Patterns という本のGerard Meszarosによって作られました。その本のテストダブルについてのさらなる情報は、本の[ウェブサイト](http://xunitpatterns.com/Test%20Double%20Patterns.html)で知ることができます。更にMartin Fowler によって書かれたこの記事には異なる[テストダブル](http://martinfowler.com/articles/mocksArentStubs.html)について議論されています。

