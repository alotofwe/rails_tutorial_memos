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

**full_titleヘルパを利用するために，``` include ApplicationHelper ```をする**

``` response.body ```は，現在表示されている (べき) ページのhtmlのソースが文字列として入っている

**assert_selectはタグを指定する必要があるが，assert_matchは第一引数に探したい文字列，第二引数にどの文字列から探すかを指定し，タグを指定する必要は無い**

**``` assert_select 'h1>img.gravatar' ```は，h1の中のimg.gravatarという意味**

## 11.3 Manipulating microposts

今は表示するだけなので，操作 (追加したり消したり) をWeb上から行えるようにする

また，feedも追加する

今回は全ての操作をHome pageで行うため，create, destroyアクションのみがあればよい

### 11.3.1 Micropost access control

まず，ログインしていないユーザにはcreate, destroyが行えないようにする (まずテストを書く)

User Controllerにあったlogged_in_userメソッドをApplicationControllerに移動させ，どのControllerからも使用できるようにする (元々User Controllerにあったものは消す)

次に，Micropost Controllerにbefore_actionとしてlogged_in_userを追加する

### 11.3.2 Creating microposts

createを実装していく

現在，ログインしていてもHome Pageに"Sign up now!"があって不自然なので，micropostを投稿するフォームを，ログインしている時のみHome Pageに表示させる

Userと同じようにcreateメソッドを追記するが，newではなくbuildを使う

また，Home Pageのviewは，ログインしているかそうでないかのif-elseをerbで書くことにより操作する

ユーザ情報を部分テンプレートとして追加する  
pluralize(num, word)は，単数/複数形を自動で管理する

また，投稿フォームも部分テンプレートとして追加する

form_forで使用するため，static_pages_controller#homeにて，ログインしている場合のみ@micropost (Micropost.new) を生成する


また，この投稿フォームでも_error_messages.html.erbを使っているが，これは現在@userのエラー内容を使用することを前提とするため，どのオブジェクトに対しても機能するように，以下のように改良する

改良前

``` ruby
<% if @user.errors.any? %>
```

改良後

``` ruby
<% if object.errors.any? %>
```

そして部分テンプレートを使うことろで，objectを以下のように指定する

``` ruby
<%= render 'shared/error_messages', object: f.object %>
```

こうすることで，どのオブジェクトでも使いまわせるようになる

テストが通り，バリデーションが効いていることを確認する

### 11.3.3 A proto-feed

現在，投稿しても結果をすぐに確認することができないため，Home Pageにfeedを設置する

feedは，あとで機能を拡張するが，この時点では自分の投稿が確認できるようになっている

User Modelにfeedメソッドを追加し，feedを簡単に習得できるようにしておく

後の拡張のために，``` Micropost.where("user_id = ?", id) ```とする

また，"user_id = #{id}"とせず?を使う理由は，これを使うことでエスケープを行い，SQL injectionを防ぐことができるため

feedも長くなる可能性が十分にあるため，paginateを使用する

static_pages_controller#homeにて呼び出し，表示させる

ただし，以下のコードでは

``` ruby
if @micropost.save
  flash[:success] = "Micropost created!"
  redirect_to root_url
else
  # @feed_items = []
  render 'static_pages/home'
end
```

elseの後の``` @feed_items = [] ```が無かったとすると，そのままstatic_pages/homeがrenderされる (static_pages_controller#homeが呼ばれない) ため，@feed_itemsが宣言されずエラーとなる

そのため，ここで宣言を行う (ifの中の場合はredirect_toなので，static_pages_controller#hogeが呼ばれる)

### 11.3.4 Destroying microposts

次に，各micropostsの作成者が，そのMicropostを削除できるようにする

viewで，自分のポストに"delete"ボタンを追加することにより実装する

まず，viewにdeleteボタンを追加する

> The main difference is that, rather than using an @user variable with an admin_user before filter, we’ll find the micropost through the association, which will automatically fail if a user tries to delete another user’s micropost.

ユーザの削除ではbefore_filterで管理者以外のユーザが削除できないようにしていたため，今回はbefore_filterで所有者本人かどうかを確かめる

**request.referrerは，そのページを見る前にどのページを見ていたかの場所を記憶している**

また，(left) || (right)は，leftがfalseまたはnilだった時にrightが評価され，そうでなければleftが評価される

### 11.3.5 Micropost tests

テストを書く

1. micropost作成者ではないユーザがそれを消そうとした場合，消されずrootにリダイレクトされる
2. 不正な入力の時にエラーが表示され，投稿はされない
3. 不正ではない入力の時に投稿され，rootにリダイレクトし，response.bodyに投稿した内容が含まれている
4. micropostの持ち主が消そうとした時，きちんと消える
5. 自分ではないユーザのshowに訪れた時，deleteボタンが0個表示されている

ことをテストし，通ることを確認する

## 11.4 Micropost images

textだけではなく，画像も投稿できるようにする

### 11.4.1 Basic image upload

アップローダはCarrierWaveを使うため，Gemfileに以下を追記する

``` ruby
gem 'carrierwave',             '0.10.0'
gem 'mini_magick',             '3.8.0'
gem 'fog',                     '1.23.0'
```

bundle installを行う

CarrierWaveはgeneratorとしてuploaderを追加するため，それを使って以下を行う

``` rails generate uploader Picture ```

uploaderで生成されたPictureは，外部とstring columnで関連させる必要がある (Pictureではなく，関連させる先にstring columnを追加する)

よって，Micropostにpicture:stringを追加し，Pictureと関連付けるために，Micropost Modelに以下を追記する

``` ruby
mount_uploader :picture, PictureUploader
```

また，view側にf.file_fieldで画像用inputを追加する  
画像投稿を追加する際は，form_forに``` html: { multipart: true } ```を追記する

strong paramsのpermitに，先ほど追加したpictureを追加する

そして表示として，_micropost.html.erbに，画像データを持っていれば表示するよう，以下のように追記する

画像データが存在するかは，``` .picture? ```で調べることができる

``` ruby
  <%= image_tag micropost.picture.url if micropost.picture? %>
```

### 11.4.2 Image validation

画像へのバリデーションを追加する

1. 形式がjpg, jpeg, git, pngのいずれかであるもののみを受け付ける
2. サイズが5MB以下のもののみ受け付ける

というものを追加する

1についてはPictureUploaderにコメントアウトされて記載されているため，コメントアウトを外す

2については，サイズに関するバリデートは標準に無いため，以下のように自作する

``` ruby
validate  :picture_size

private

# Validates the size of an uploaded picture.
def picture_size
  if picture.size > 5.megabytes
    errors.add(:picture, "should be less than 5MB")
  end
end
```

**また，viewの方でも，以下のように制限を行う**

``` ruby
<%= f.file_field :picture, accept: 'image/jpeg,image/gif,image/png' %>
```

``` js
$('#micropost_picture').bind('change', function() {
  var size_in_megabytes = this.files[0].size/1024/1024;
  if (size_in_megabytes > 5) {
    alert('Maximum file size is 5MB. Please choose a smaller file.');
  }
});
```

### 11.4.3 Image resizing

画像の容量の問題はクリアしていても，画像の縦横のサイズが大きく画面からはみ出るものが投稿されてしまう可能性があるため，投稿された時にリサイズを行う

リサイズにはImageMagickが必要なので，入っていなければapt-getやbrew install等で入れる

Rails側では，ImageMagickをRailsで使用するインタフェースであるCarrierWave::MiniMagickをPictureUploaderにincludeしておく
その上で，以下を追加する

``` ruby
process resize_to_limit: [400, 400]
```

**アップロード時にリサイズされるため，リサイズ導入前に投稿された画像にはリサイズはかからない**

### 11.4.4 Image upload in production

画像をローカルに (アプリのあるサーバ内) に保存するということは，production環境においては適していないため (量が多くなり，管理が難しくなる)，外部のストレージサービスに保存する

今回は，Amazon S3を使用する

PictureUploaderのストレージを指定するところで，productionであればfogを使うように指定する

以下の手順で，S3の登録を行う

1. AWSに登録する
2. IAMに入り，新規ユーザの作成を行う
3. key, secret_keyを落とす
4. 使っているユーザの設定画面で「ポリシーのアタッチ」ボタンを押す
5. AmazonS3FullAccessにアタッチする
6. S3のバケットを作成する (名前に``` . ```は含めない)

以上を行った上で，config/initializers/carrier_wave.rbを作成し，以下を追記する

``` ruby
if Rails.env.production?
  CarrierWave.configure do |config|
    config.fog_credentials = {
      # Configuration for Amazon S3
      :provider              => 'AWS',
      :aws_access_key_id     => ENV['S3_ACCESS_KEY'],
      :aws_secret_access_key => ENV['S3_SECRET_KEY']
    }
    config.fog_directory     =  ENV['S3_BUCKET']
  end
end
```

その上で，key, secret_key, バケット名を以下のようにherokuに設定する

```
heroku config:set S3_ACCESS_KEY=<access key>
heroku config:set S3_SECRET_KEY=<secret key>
heroku config:set S3_BUCKET=<bucket name>
```

.gitignoreでローカルに落とされた画像を無視するように設定し (``` /public/uploads ```)，デプロイして画像の保存がheroku上でうまく動くことを確かめる
