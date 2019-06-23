# Testing on the Toilet: Fake Your Way to Better Tests

https://testing.googleblog.com/2013/06/testing-on-toilet-fake-your-way-to.html


長年のブログを経て、ブログプラットフォームのAPIを試してみることにします。遊び始めると、テストコードとリモートのブログサーバーと通信する必要なく動作するコードが機能することをどのように確認できるのだろうか。ということに気づくでしょう。


    public void deletePostsWithTag(Tag tag) {
      for (Post post : blogService.getAllPosts()) {
        if (post.getTags().contains(tag)) { blogService.deletePost(post.getId()); }
      }
    }

フェイクが助けになります！**フェイクは本番には適しませんが、実際の実装を真似た軽量API実装のことです。**ブログサービスの場合では、気にかけることのすべては投稿を取得し削除できるということでしょう。本物のブログサービスではデータベースや、複数のフロントエンドサーバーをサポートする必要がありますが、コードをテストするためにはそれを使用する必要はなく、必要なのはブログサービスのAPI実装だけです。これは簡単なインメモリ実装で達成することができます。

    public class FakeBlogService implements BlogService {  
      private final Set<Post> posts = new HashSet<Post>(); // Store posts in memory
      public void addPost(Post post) { posts.add(post); }
      public void deletePost(int id) {
        for (Post post : posts) {
          if (post.getId() == id) { posts.remove(post); return; }
        }
        throw new PostNotFoundException("No post with ID " + id);
      }
      public Set<Post> getAllPosts() { return posts; }
    }

さて、テストをフェイクとして本物のブログサービスとして置き換えることができ、テストコード内ではその違いがわかりません。

本物の実装がとても遅い (例えば起動に数分かかる) や、非決定的である (例: テストの実行時には利用できない可能性のある外部のマシンと対話する) といった、**テスト内で本物の実装を利用できないときにフェイクは便利です** 

**それぞれのフェイクが実際の実装を所持しているチームや人によって作成され保守されているため**、フェイクを自身で書くことはないことも多いです。フェイクが提供されていないAPIを用いる場合でも、自身でフェイクを作成することは簡単です。テストコード内で利用できない部分のコードのラッパーを書き、フェイクとしてラッパーを作成します。**できるだけ低いレベルでフェイクを作成することを忘れないでください** (例: テスト内でデータベースを用いることができない場合、データベースと通信するすべてのクラスのフェイクを作成するのではなく、データベースをフェイクします)、そうすれば管理するフェイクが少なくなり、テストでシステムの重要な部分に対してより実際のコードが実行されるようになります。

**フェイクには実際の実装のように振る舞っていることを保証するためのテストを持つべきです**。(例: 特定の入力を受け取ったときに実際の実装が例外を投げるのであれば、フェイク実装も同じ入力で例外を投げるべきです) これを行う1つの方法は、APIのパブリックインターフェースに対してテストを作成しそれらのテストを実際の実装とフェイクの実装の両方に対して行うことです。

すべてのテストでフェイクを利用しても、コードが本番環境で動作するとはまだ十分に信頼できない場合には、少数の統合テストを作成し、コードが実際の実装で動作することを確認できます。

