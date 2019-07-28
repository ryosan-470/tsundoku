# Testing on the Toilet: Test Behaviors, Not Methods

https://testing.googleblog.com/2014/04/testing-on-toilet-test-behaviors-not.html


メソッドを書いたあとにそのメソッドが行うすべてを検証するたった一つのテストを書くことは容易です。しかし、**テストと公開メソッドは1対1の関係であるべきだと考えることは危険です**。私達が本当にテストがしたいことは、単一のメソッドが多くの振る舞いを示し、単一の振る舞いが複数のメソッドにまたがる振る舞いをするかということです。

一つのメソッド全体を検証する悪いテストをみてみましょう。


    @Test public void testProcessTransaction() {
      User user = newUserWithBalance(LOW_BALANCE_THRESHOLD.plus(dollars(2));
      transactionProcessor.processTransaction(
          user,
          new Transaction("Pile of Beanie Babies", dollars(3)));
      assertContains("You bought a Pile of Beanie Babies", ui.getText());
      assertEquals(1, user.getEmails().size());
      assertEquals("Your balance is low", user.getEmails().get(0).getSubject());
    }

購入したものの名前を表示することと、残高が足りないとメールを送ることは2つの分割された振る舞いですが、このテストでは同じメソッドが発火することで起きるためにこの2つの振る舞いを同時にテストしています。**このようなテストは時間とともに更に振る舞いが追加されることにより大きく保守が難しいものになっていきます**。つまり、結局のところどの入力がどのアサーションに対応しているのか伝えるのが難しくなってしまうのです。テストの名前がメソッドの名前と同じであるということは悪い兆候です。

**分割された振る舞いに関してそれぞれテストを行うようにすることで更に良くなります。** 


    # 残高の表示に関するテスト
    @Test public void testProcessTransaction_displaysNotification() {
      transactionProcessor.processTransaction(
          new User(), new Transaction("Pile of Beanie Babies"));
      assertContains("You bought a Pile of Beanie Babies", ui.getText());
    } 
    
    # 残高不足をメールすると言うテスト
    @Test public void testProcessTransaction_sendsEmailWhenBalanceIsLow() {
      User user = newUserWithBalance(LOW_BALANCE_THRESHOLD.plus(dollars(2));
      transactionProcessor.processTransaction(
          user,
          new Transaction(dollars(3)));
      assertEquals(1, user.getEmails().size());
      assertEquals("Your balance is low", user.getEmails().get(0).getSubject());
    }

さて、もし誰かが新しい振る舞いを追加したならば、そのふるまいに関して新しいテストを書きます。それぞれのテストはそのままであることで理解がしやすくなり、どんなに多くの振る舞いが追加されても問題ありません。**これは新しいふるまいが既存のテストを破壊することはないため、テストをさらに弾力性のあるものにし、それぞれのテストがある振る舞いにのみ検証することで明瞭になります。** 

