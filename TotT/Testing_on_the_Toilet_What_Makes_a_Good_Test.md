# Testing on the Toilet: What Makes a Good Test?

https://testing.googleblog.com/2014/03/testing-on-toilet-what-makes-good-test.html


単体テストはコードが正しいかを検証するにあたり重要なツールです。しかし、**良いテストを書くことは正当性を検証するだけではありません**。優れた単体テストは読みやすく保守しやすいように、他にもいくつかの特性を表します。

優れたテストの1つの特性として明快さが挙げられます。
**明快さとはテストがその公開APIの観点からテストされているコードを記述して人間にとって読みやすい文書として役立つべきであるということを意味します**。
テストは直接実装の詳細を参照すべきではありません。クラスのテストの名前はそのクラスが行うすべてのことを言うべきであり、そしてテスト自体がそのクラスの使い方の例として役立つべきです。

2つの更に重要な特性は完全性と簡潔さです。
**テストには理解するために必要な情報がその本体に含まれていれば完全であり、その他の注意をそらす情報が含まれていなければ簡潔です** 

このテストは両者の点で失敗しています。

    @Test public void shouldPerformAddition() {
      Calculator calculator = new Calculator(new RoundingStrategy(), 
          "unused", ENABLE_COSIN_FEATURE, 0.01, calculusEngine, false);
      int result = calculator.doComputation(makeTestComputation());
      assertEquals(5, result); // この数字がどこからきたの?
    }

多くの注意をそらす情報がコンストラクタに渡されており、重要な部分はヘルパーメソッドに隠されています。テストはヘルパーメソッドの目的を明確にすることにより完全にすることができ、別のヘルパーを使用して計算機を構築することで関連性のない詳細を隠し、より簡潔にすることができます。


    @Test public void shouldPerformAddition() {
      Calculator calculator = newCalculator();
      int result = calculator.doComputation(makeAdditionComputation(2, 3));
      assertEquals(5, result);
    }

優れたテストの最後の1つの特性は回復力です。**テストが書かれたクラスの目的や動作が変わらない限り、回復力のあるテストは変更する必要がありません**。新しい動作を追加するためには、新しいテストを追加するだけであり、古いテストを変更する必要はありません。上記の元のテストでは、新しい無関係なコンストラクタのパラメータを追加するときはいつでも古いテストを (おそらく多くの他のテストを) 更新する必要があるので回復力を持ちません。これらの詳細をヘルパーメソッドに移動することでこの問題を解決できました。

