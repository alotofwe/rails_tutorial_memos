# Chapter 11 User microposts

Userについて色々いじったので，次はMicropostsをいじる

> add a second such resource: user microposts, which are short messages associated with a particular user.

Tweetみたいに，ユーザが短文を発言できるようになる

## 11.1 A Micropost model

Userで実装したことに加え，default orderとdestructionも指定する

destructionの指定では，親が死んだら子も死ぬ設定を追加することができる (紐付けられているUserが消えたら，micropostsも自動で消える)

### 11.1.1 The basic model

micropostsは，contentカラムとuserカラムを持つ

contentカラムはstringではなくtext

using text better expresses the nature of microposts, which are more naturally thought of as blocks of text. Indeed, in Section 11.3.2 we’ll use a text area instead of a text field for submitting microposts.

> In addition, using text gives us greater flexibility should we wish to increase the length limit at a future date (as part of internationalization, for example).

> Finally, using the text type results in no performance difference in production,2 so it costs us nothing to use it here.

後々拡張するかもしれない，かつtextを使っても問題は無いので，stringではなくtextを使う

``` rails generate model Micropost content:text user:references ```

> The biggest difference is the use of references, which automatically adds a user_id column (along with an index and a foreign key reference)3 for use in the user/micropost association.

referencesを使うと自動的に<class>_idカラムを追加し，かつindexも貼る

> add_index :microposts, [:user_id, :created_at]

**multiple key indexを用いると，indexを使っての検索の際に，必ずこの2つのキーを使用して検索を行う**

### 11.1.2 Micropost validations

user_idは必ず指定される，等のvalidationsを追記する

1. User idは必ず値が入っていなければならない
2. contentはnilや空白であってはならない
3. contentは最大で140文字でなくてはならない

というテストを書き，落ちることを確認し，落ちないようにアプリのコードを書き換える

### 11.1.3 User/Micropost associations

あるモデルと他のモデルを関連付けることができる (associations)

UserとMicropostsでは，ある1つのUserは複数のMicropostsを持ち，各Micropostは1つのUserに持たれている

![Figure11.2](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/micropost_belongs_to_user.png)

![Figure11.3](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_has_many_microposts.png)

**belongs_toとhas_manyを使うと，build等の便利なメソッドが追加される**

* micropost.user	(Returns the User object associated with the micropost)
* user.microposts	Returns a collection of the user’s microposts)
* user.microposts.create(arg)	(Creates a micropost associated with user)
* user.microposts.create!(arg)	(Creates a micropost associated with user (exception on failure))
* user.microposts.build(arg)	(Returns a new Micropost object associated with user)
* user.microposts.find_by(id: 1)	(Finds the micropost with id 1 and user_id equal to user.id)

**buildはnewに似ているが，user_idを記述せずとも自動で挿入される点が異なっている**

Micropost Modelにbelongs_toを，User Modelにhas_manyを追記する

テストでbuildを使用し，通ることを確かめる

### 11.1.4 Micropost refinements

default orderとdestructionを設定する

まずdefault orderのため，created_atを明示したfixtureで，一番最近のものがfirstにきているかどうかを確かめ，テストが落ちることを確認する

次にUser Modelについてdefault_scopeで，デフォルトの並び順を指定する

``` ruby
default_scope -> { order(created_at: :desc) }
```

orderは，``` order(:column_name) ```のように指定するが，指定なしだと昇順 (日時の場合は古い順) になるため，``` order(column_name: :desc) ```で降順に指定する

また，``` -> {...} ```はProcやlambdaと呼ばれる無名関数であり，**Proc.callでProcを評価 (実行) することができる**

``` ruby
>> -> { puts "foo" }
=> #<Proc:0x007fab938d0108@(irb):1 (lambda)>
>> -> { puts "foo" }.call
foo
=> nil
```

テストが通ることを確かめる

次に，dependent: destroyを指定する

親 (User) が死んだ時に子供達 (Microposts) も同時に死ぬ

has_manyのオプションとして，以下のように書く

``` ruby
has_many :microposts, dependent: :destroy
```

親を殺すようなテストを書いて，きちんとMicropostsも死ぬことを確かめる

## 11.2 Showing microposts

表示させてみる (Webからの投稿はまだ)

### 11.2.1 Rendering microposts

users#indexのように，縦にMicropostsを表示させるようにする (paginateもする)

しばらく使わないけれどとりあえずMicroposts Controllerをつくる

``` rails generate controller Microposts ```

@usersが_user.html.erbでeachを使わず表示することができたように，@micropostsも_micropost.html.erbを作ることで同じように表示をする

**time_ago_in_wordsメソッドは，"n minutes ago"等の文字列を，datetime型データを突っ込むことで自動生成する**

また，idにerbを埋め込み，Micropostごとに違うidを割り振っている

users#indexみたいにwill_paginateを入れるが，**paginateは現在のControllerからページ送り対象のインスタンス変数を探すが，今回はUser Controllerで@micropostsを対象としたいため，第一引数で変数を指定する**

ここでも``` .count ```を使用しているが，User全体を取り出してからその数を数えるわけではなく，DBに数を直接問合わせているため，Micropostsが増えてもボトルネックにはならない

viewを追記する，ただしまだ1つもMicropostのデータを入れていないため，悲しいことに表示はされない

### 11.2.2 Sample microposts

サンプルデータを作り，実際にviewでMicropostsの表示をさせる

ただしUser全員にMicropostsを追加すると時間がかかるため，最初の6名のみに追加する

``` ruby
users = User.order(:created_at).take(6)
50.times do
  content = Faker::Lorem.sentence(5)
  users.each { |user| user.microposts.create!(content: content) }
end
```

seed追加後，viewで表示されることを確かめる  
time_ago_in_wordsメソッドがうまく機能していることも確かめる

### 11.2.3 Profile micropost tests

Micropostsのintegration testを書く

fixtureを書く際，Micropostsのuserには``` user: user_name ```と指定することができる (user_nameはuser fixturesで指定している見出し)

full_titleヘルパを利用するために，``` include ApplicationHelper ```をする

``` response.body ```は，現在表示されている (べき) ページのhtmlのソースが文字列として入っている

assert_selectはタグを指定する必要があるが，assert_matchは第一引数に探したい文字列，第二引数にどの文字列から探すかを指定し，タグを指定する必要は無い

``` assert_select 'h1>img.gravatar' ```は，h1の中のimg.gravatarという意味
