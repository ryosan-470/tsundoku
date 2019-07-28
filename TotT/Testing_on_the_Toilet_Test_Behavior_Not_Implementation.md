# Testing on the Toilet: Test Behavior, Not Implementation

https://testing.googleblog.com/2013/08/testing-on-toilet-test-behavior-not.html


あなたの信頼できるCalculatorクラスは最も有名なオープンソースプロジェクトの1つであり、多くのユーザーを幸せにしています。

    public class Calculator {
      public int add(int a, int b) {
        return a + b;
      }
    }

**更にテストにより、**正しく動作することを保証しています。

    public void testAdd() {
      assertEquals(3, calculator.add(2, 1));
      assertEquals(2, calculator.add(2, 0));
      assertEquals(1, calculator.add(2, -1));
    }

しかしながら加算演算子をこのメソッドの代わりに用いるならば、この空想の新しいライブラリはあなたのコードを数倍加速することを約束します。あなたは興奮してこのライブラリを用いるようにコードを修正しました。


    public class Calculator {
      private AdderFactory adderFactory;
      public Calculator(AdderFactor adderFactory) { this.adderFactory = adderFactory; }
      public int add(int a, int b) {
        Adder adder = adderFactory.createAdder();
        ReturnValue returnValue = adder.compute(new Number(a), new Number(b));
        return returnValue.convertToInteger();
      }
    }

これに関しては簡単ですが、この部分のコードのテストに関してはどうしますか? コードの実装を変更しただけなので既存のテストをどれも変更する必要はありませんし、ユーザー向けの動作も変わりませんでした。多くの場合、 **テストは公開されたAPIに対して焦点を当てるべきであり、コード実装の詳細をテストに晒すべきではありません** 

実装を変更してもテストコードを修正する必要がないため、**実装の詳細から独立しているテストは維持が容易です。**テストコードが基本的にクラスのメソッドとして利用されることができるすべての異なる方法を示すコードサンプルとして振る舞うため理解することが容易であり、そのため実装に慣れていない人にとっても、通常はテストを読むことでクラスの使い方を理解できるでしょう。

実装の詳細をテストしたい場合が多くありますが、(例えば実装がデータストアからではなくキャッシュから読み取るようにしたい場合など) テストは実装から独立しているべきであり、あまり一般的ではありません。

**テストの初期化は実装が変わることにより修正が必要になることがあるかもしれないということに留意してください** (例えば、コンストラクタへの新しい依存をクラスに渡す変更をした場合、テストにおいてもクラスを作成する段階でこの依存を渡す必要があります。) しかし、実際のテスト自身は一般的にユーザー向けの振る舞いが変わらなければ変わることはありません。

