# Chapter 2 A toy app

> As discussed in Box 1.2, the rest of the book will take the opposite approach, developing a full sample application incrementally and explaining each new concept as it arises, but for a quick overview (and some instant gratification) there is no substitute for scaffolding.

> the whole point of this tutorial is to take you beyond this superficial, scaffold-driven approach to achieve a deeper understanding of Rails.

ばさっと概要を把握して見た目も簡単に確認するために，この章ではscaffoldを使ってたくさんの機能を簡単に作る (scaffold-driven)  
この章以降では他の方法を使う

## 2.1 Planning the application

```rails new```してGemfileをいじって```bundle install --without production```してgitに上げてherokuにも上げる (Chapter 1参照)

アプリを作る準備完了だ

> The typical first step when making a web application is to create a data model, which is a representation of the structures needed by our application.

とりあえず最初はデータモデルを作ってアプリの構成を示すのが典型

### 2.1.1 A toy model for users

とりあえず最小限に，これから先に作るアプリのためのデータモデルを決めておく

Usersは以下のとおり

![Figure2.2](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_user_model.png)

### 2.1.2 A toy model for microposts

Micropostsは以下のとおり

![Figure2.3](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_micropost_model.png)

> There’s an additional complication, though: we want to associate each micropost with a particular user.

特定のユーザに各マイクロソフトを紐付けるためにuser_idカラムを定義する  
Section 2.3.3やChapter 11から，どういうふうに紐付けを行っていくかを見ていく

## 2.2 The Users resource

Section 2.1.1のuser modelを実際に実装していく

User resourceを構築することで，created, read, updated, deleteを可能にする

> I urge you not to look too closely at the generated code; at this stage, it will only serve to confuse you.I urge you not to look too closely at the generated code; at this stage, it will only serve to confuse you.

scaffoldで生成されたものについてはあまり難しく考えずに今はさくっといこう

```rails generate scaffold User name:string email:string```

> The argument of the scaffold command is the singular version of the resource name (in this case, User), together with optional parameters for the data model’s attribute

**scaffoldで宣言する名前は単数形**  
その後にオプションとして属性 (attributes) を指定することができる  

ここではFigure 2.2 で定義したカラムを属性として指定する  
ただし，idは自動で生成され，railsでは主キーとしてDBで使用される

(主キー (primary key) : あるレコードを一意に識別するための主な手がかり, 主キーは重複を許さない)

```bundle exec rake db:migrate```

DBのアップデートを行う (Section 6.1.1で詳しく学ぶ)

> Note that, in order to ensure that the command uses the version of Rake corresponding to our Gemfile, we need to run rake using bundle exec.

**使用しているGemfileのバージョンに対応したrakeのバージョンを使うために**，```bundle exec```をつける

### Box 2.1. Rake

> In the Unix tradition, the make utility has played an important role in building executable programs from source code;

makeでソースコードから実行可能なプログラムを作成する

rakeはRuby makeであり，Ruby版makeである

### 2.2.1 A user tour

scaffoldするとページが沢山自動生成される (index, show, new, edit)  
それぞれを実際にブラウザ上で見て確認してみる  

indexでは，全てのユーザを表示させる (/users)

newでは，ユーザが新しくサインアップするフォームが表示される (/users/new)

showでは，指定されたユーザの情報を閲覧することができる (/users/1)

editでは，指定されたユーザの情報を編集するフォームが表示される (/users/1/edit)

ブラウザ上で新しくユーザを生成したり消してみたり遊んでみる

### 2.2.2 MVC in action

User resourceについてざっくり把握したので，User modelという実例に基づいて，MVCについて改めて調べてみる

![Figure2.11](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/mvc_detailed.png)

上の図は，/usersを閲覧しようとした時の流れの説明図  
図の矢印に書かれた番号の順に処理が進んでいく

Users Controllerのindex actionをルートに設定する

(action : Controller Class内での，Rubyで言うところのメソッド)

Controllerを見ると，先ほどの4つのaction以外にもcreate, update, destroyアクションが確認できる  
これはページは表示させず，DB内容の変更を行う

RESTfulだ!!!!! (Webサービスのスタイル (アーキテクチャ) の1つ)

> Fielding氏が示したRESTの設計原則は主として以下の4つの項目から成る。「セッションなどの状態管理を行わない（やり取りされる情報はそれ自体で完結して解釈することができる）」（Webシステムでは、HTTP自体にはセッション管理の機構はない）、「情報を操作する命令の体系が予め定義・共有されている」（WebシステムではHTTPのGETやPOSTなどに相当）、「すべての情報は汎用的な構文で一意に識別される」（URLやURIに相当）、「情報の内部に、別の情報や（その情報の別の）状態へのリンクを含めることができる（ハイパーメディア的な書式で情報を表現する）」

[RESTとは｜REpresentational State Transfer - 意味/解説/説明/定義 ： IT用語辞典](http://e-words.jp/w/REST.html)

> Note from Table 2.2 that there is some overlap in the URLs; for example, both the user show action and the update action correspond to the URL /users/1. The difference between them is the HTTP request method they respond to.

同じURLに対応していてもHTTP requestが違う点に注意  
routerが識別するときには，URLだけでなくHTTP requestも見ている

User controllerとUser modelの連携を，実際にコードを見てみることで調べてみる (User.allとかする)

> In particular, by using the Rails library called Active Record, the code in Listing 2.6 arranges for User.all to return all the users in the database.

Active Recordがあることで，自分でSQL文を書かなくとも，よしなにデータをDBから取ってくることができる

> Once the @users variable is defined, the controller calls the view (Step 6), shown in Listing 2.7. Variables that start with the @ sign, called instance variables, are automatically available in the views

インスタンス変数 (@がついている) はviewで使える  

実際にviewで使われていることの確認

### 2.2.3 Weaknesses of this Users resource

scaffoldで生成されたUser resourceの弱点

* バリデーションが無い
  - 明らかに望まれない入力データでも受け入れて保存してしまう
* 認証がない
  - ログイン/ログアウトが無いため，誰でも何でもすることができる
* テストが無い
  - どの機能についても何も検証されていない
* レイアウトが無い
  - ページごとのレイアウトが一貫していない
* 本当の理解にならない
  - ぽちぽちしてファイルを生成しているだけ

## 2.3 The Microposts resource

Users resourceを作ったので，次はMicroposts resourceを作る

> indeed, seeing the parallel structure of Users and Microposts even at this early stage is one of the prime motivations for this chapter.

UsersとMicropostsを，平行して生成し見ていきましょう

### 2.3.1 A micropost microtour

Userみたく```rails generate scaffold Micropost content:text user_id:integer```して```bundle exec rake db:migrate```をする

Userみたくroutesに```resources :microposts```が追記されているのを確認

UserみたくMicroposts Controllerが生成されているのを確認

Userみたくページが生成されているのをブラウザ上でぽちぽちして確認

### 2.3.2 Putting the micro in microposts

Micropostの内容の文字数制限をかけるために，validationsを追記する

```validates :content, length: { maximum: 140 }```

Micropost Modelにlength validationを追記  
140文字までの制限をかける

validationsについてはSection 6.2で詳しく行う

140文字を超えるような入力をブラウザ上でフォームから行うと，エラーメッセージが表示されるのを確認する

### 2.3.3 A user has_many microposts

Railsの強みの1つが，異なるモデル同士で関連 (associations) が取れるということ

```has_many :microposts```  
``` belongs_to :user```

![Figure2.15](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/micropost_user_association.png)

Micropostにuser_idがあるおかげで，各micropostsはあるuserと関連することができる

rails consoleから，userとmicropostsで関連がとれているのを確認

### 2.3.4 Inheritance hierarchies

オブジェクト指向プログラミングの概要を学ぶ

```class Piyo < Hoge```

PiyoクラスはHogeクラスを継承していることがこの記述から分かる  

Hogeを継承したPiyoは，Hoge (と，Hogeが継承しているfoo, fooが継承しているbar ... ) の機能を受け継ぐ

実例を元にすると以下のとおり

![Figure2.16](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_model_inheritance.png)

![Figure2.17](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_controller_inheritance.png)

### 2.3.5 Deploying the toy app

```git push```して```git push heroku```する (Chapter 1 参照)

**```heroku run rake db:migrate```でherokuのproduction環境でもDBのアップデートを行う**
