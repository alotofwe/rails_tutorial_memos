# Chapter 9 Updating, showing, and deleting users

edit, update, index, destroyを追加し，Users resourceをRESTfulにする

また，DB操作権限を全てのユーザに渡さないために，ユーザに権限の有無を設ける

paginationも導入する

## 9.1 Updating users

ユーザの情報を編集できるようにする

ただし，本人のみ編集可能にする

### 9.1.1 Edit form

/users/:id/editが，そのidの編集画面のURLとなる

また，:idの数はparams[:id]で取得可能

viewはusers/newみたいなフォームを書く

``` target="_blank" ```は，Gravatarのリンクを別タブ or ウィンドウで表示させる

また，form_forで@user (User.find(params[:id])で取得したもの)を利用することにより，現在設定されているnameやemailなどが予めフォームに入力されている

> <input name="_method" type="hidden" value="patch" />

> Since web browsers can’t natively send PATCH requests (as required by the REST conventions from Table 7.1), Rails fakes it with a POST request and a hidden input field.2

**ブラウザは標準でPATCHをサポートしていないため，Rails側でPOSTに偽装する**

**form_for(@user)部分が全く同一にも関わらず，users/newとusers/editがどう区別されているかというと，new_record?メソッドでそれが既にDBにあったかどうかを判定している**

編集画面へのリンクをheaderに追記する

### 9.1.2 Unsuccessful edits

いつも通り，フォームを作った後は，まず不正な入力から実装していく

更新は``` user.update_attributes({}) ``` を用いて行う  
更新が正常に完了した場合にtrue，そうでなければfalseを返す

strong paramsで{}に直接paramsを入れることができないため，ここでもprivateのuser_paramsメソッドを利用する

viewにerror-messagesを出力する部分テンプレートを挿入しているため，入力のどこが不正だったかがviewに表示される

### 9.1.3 Testing unsuccessful edits

不正なフォーム内容に対してのテストを行う

``` rails generate integration_test users_edit ```

patchメソッドを用いてuser_path(@user)にpatchリクエストを投げ，その後users/editが表示されるかを調べる

### 9.1.4 Successful edits (with TDD)

TDDで，正しい内容でユーザ情報が編集されたときの処理を書く

正しい内容の内容をpatchし，

1. flashが空ではないこと (更新が正常に終了した旨のメッセージを入れる)
2. リダイレクト先がuser_path(@user)であること
3. 更新されていること **(@user.reloadを忘れない)**

を確認する

現時点ではテストが落ちるため，通るように実際のコードを書き換える

ただし，フォームでは変更したくない項目はnilでpatchが送られており，user.passwordのvalidatesではnilが認められていないため，``` allow_nil: true ```を追記する  
**一見，変更後パスワードが空白状態になってしまうように見えるが，has_secure_passwordの方のvalidationでそのようなことは起こらないようになっており，実際にはパスワードは変更されない**

[Rails - has_secure_passwordを読む - Qiita](http://qiita.com/dblN/items/9da63698199f4e6135ae)

## 9.2 Authorization

> they suffer from a ridiculous security flaw: they allow anyone (even non-logged-in users) to access either action, and any logged-in user can update the information for any other user.
> In this section, we’ll implement a security model that requires users to be logged in and prevents them from updating any information other than their own.

誰もが全てのアクションを行うことができ，ログインしたユーザであればどのユーザの登録情報でも編集することができてしまっている状態

このセクションでは，それらを行えないようにする

### 9.2.1 Requiring logged-in users

まず，ログインをしていないユーザに見られたくないページにプロテクトをかけ，ログインを促すようにする

before filterをUser Controllerに，以下のように追記

``` before_action :logged_in_user, only: [:edit, :update] ```

このように記述し，privateにlogged_in_userメソッドを追記すると，editアクションとupdateアクションの時のみ，アクションの処理に入る前にlogged_in_userメソッドの処理が走る

onlyを指定すればそのアクションだけが，exceptを指定すればそのアクション以外が，どちらも指定しないと全てのアクションがbefore filterの対象となる

これを追記すると，ログインを必要としているページにアクセスしようとしているテストで落ちるため，テストのほうを直す

次に，before filterがきちんと効いていることを確認するテストを書く (ログインしていない状態でedit, updateにアクセスしようとするとlogin_urlに飛ばされることを確認する)

**``` id: @user ```を使うことができ，これは``` id: @user.id ```と同じ意味になる**

### 9.2.2 Requiring the right user

次に，ユーザ登録情報は本人のみが編集可能であるようにする

ここの記述でミスが起こるとセキュリティ上厄介なので，TDDで書く

本人ではないユーザでeditとupdateにリクエストを投げた際，root_urlに飛ぶことを確認するテストを書く

こちらも前と同じくbefore_actionで，params[:id]で指定されたユーザとcurrent_userが一致するかどうかを確認する

また，user == current_userをメソッド化する

### 9.2.3 Friendly forwarding

認証周りは完了したが，もう少し便利にしてみる

もし

1. 認証が必要なページに，ログインをしていない状態でアクセスする
2. ログインフォームに飛ばされる
3. 1のリクエスト先に関わらず，自分のプロフィールページに飛ばされる

という状態になっているため，3において，1で表示させようとしていたページに飛ぶ機能をつける

**この機能をFriendly forwardingと呼ぶ**

まずFriendly forwardingになるようなテストを書き，落ちることを確かめる

Friendly forwardingを実装するには，2のステップでどのURLを表示させようとしていたかを覚えておく必要があるため，そのようなメソッドをSessionHelperに追記する

sessionメソッドを使い記憶する

``` ruby
def redirect_back_or(default)
  redirect_to(session[:forwarding_url] || default)
  session.delete(:forwarding_url)
end

def store_location
  session[:forwarding_url] = request.url if request.get?
end
```

request.urlで，リクエストされたURLを取得する

store_locationメソッドは，before filterでログインしているかをチェックする際のメソッドに追記する

redirect_back_orメソッドは，session_controller#createで新たなセッションが作られた際のcreateメソッドに追記し，記憶させていたURLにリダイレクトさせる

## 9.3 Showing all users

ユーザ一覧を表示するindexアクションを用意する

実際のサービスでユーザが多くなってしまった時に備え，サンプルである程度ユーザを作成し，pagenateを使うことにより表示に耐えられるようにする

### 9.3.1 Users index

まず，ログインをしていないユーザが，ユーザ一覧を閲覧できないようにする (before actionのonlyにindexを追記するだけ)

表示させるユーザは，全てのユーザを取得するには，User.allを使う (これだとユーザが多くなるにつれ処理が重くなるため，後に改善する)

viewは，@users.eachで各ユーザの情報を表示する

CSSで整え，headerにusers/indexへのリンクも追記し，テストが通ることを確認する

### 9.3.2 Sample users

たくさんのサンプルユーザを作ることで，アプリの環境を本番に近くする

仮のデータを簡単に生成することができるfaker gemがあるため，それを使用する

(Faker-japanese gemもあり，日本語の文を生成することもできる )

[Ruby - Fakerでダミーデータを作成 - Qiita](http://qiita.com/torshinor/items/ef2141d6f3828cf0054f)

Gemfileにfakerを追記して，bundle install

使い方は[fakerのREADME](https://github.com/stympy/faker)参照

db/seed.rb内にて，timesメソッドを活用して100名のユーザを作成する

``` ruby
User.create!(name:  "Example User",
             email: "example@railstutorial.org",
             password:              "foobar",
             password_confirmation: "foobar")

99.times do |n|
  name  = Faker::Name.name
  email = "example-#{n+1}@railstutorial.org"
  password = "password"
  User.create!(name:  name,
               email: email,
               password:              password,
               password_confirmation: password)
end
```

create!メソッドは，作成・保存に失敗した際に例外を投げる

``` bundle exec rake db:migrate:reset ```

``` bundle exec rake db:seed ```

DBを初期化して，seedを入れる

### 9.3.3 Pagination

ユーザが増えた時に1つのページに全てのユーザを表示させようとすると，ものすごくページが重くなる

そのため，1ページにつき30ユーザしか表示させず，ページを操作することができるようにする

これの実現のために便利なwill_paginate gemと，そのデザインのためのbootstrap-will_paginage gemがあるので，Gemfileに追記してbundle installをする

使い方は

**1. viewで``` <%= will_paginate %> ```を追記することにより，ページ送りができるようなインタフェースを構築する**  
**2. controllerで，paginationの対象とする変数に ```@users = User.paginate(page: params[:page]) ```と指定する**

2により，params[:page]の値によって@usersに入るユーザのオフセットや量が変化する(page=1なら1~30番目のユーザ，page=2なら31~60番目のユーザが入る)

実際にWeb上で動くことを確かめる

### 9.3.4 Users index test

users/indexを書いたので，そのテストを書く

pagenateをテストするにはある程度の量のテストユーザが必要になるため，fixturesにその旨，以下のように追記する

``` ruby
<% 30.times do |n| %>
user_<%= n %>:
  name:  <%= "User #{n}" %>
  email: <%= "user-#{n}@example.com" %>
  password_digest: <%= User.digest('password') %>
<% end %>
```

**fixturesではerbを使用することができる**

integration testファイルを追加し，paginateで表示されるべきユーザが全てviewで表示されていることを，assert_selectで確かめる

テストが通ることを確かめる

### 9.3.5 Partial refactoring

ユーザの表示は様々な箇所で使いまわす可能性があるため，users/indexに書いているユーザ表示を部分テンプレート化する

いつも通り，``` <%= render user %> ```とし，部分テンプレートのファイルははじめに_をつけ，その後にrenderで指定している名前にする

**また，renderにusersを指定し，userのための部分テンプレートが存在した場合，Railsは繰り返し部分テンプレートを表示させると解釈するため，@users.eachも必要がなくなる**

リファクタリング後，テストが通ることを確かめる

## 9.4 Deleting users

indexを作成したため，RESTfulへの次のステップとしてdestroyを実装する

> In this section, we’ll add links to delete users, as mocked up in Figure 9.13, and define the destroy action necessary to accomplish the deletion

ユーザを消すためのリンクを作成し，実際に削除を行うためにdestroyアクションを実装する

> But first, we’ll create the class of administrative users, or admins, authorized to do so

ただし全てのユーザ (もしくは，全てのログイン済みユーザ) が削除可能だとサービスの機能として危険であるため，管理者であるかどうかのカラムをUser Modelに追加し，管理者のみ削除が可能にする

### 9.4.1 Administrative users

boolean型のadminカラムを，User Modelに追加する  
この際，自動でadmin?メソッドが追加され，これによりあるユーザが管理者であるかどうかを判定する (trueであれば管理者)

カラム名の末尾に?がついたメソッドはいつでも自動で生成され，そのカラムの値が0, false, nilであればfalse，そうでなければfalseが返る

生成されたmigration fileに，default: falseを指定して，何も指定されなければ自動的にfalseになるようにする

また，admin用ユーザを作成するため，seed.rbにadmin: trueのものを追記する (seedを再び入れることに注意)

adminはリクエストにより操作する予定がない (あってはいけない) ため，strong params対策のpermitに含めない

### 9.4.2 The destroy action

実際にactionを書いて動くようにしていく

まず，viewのほうに，current_user.admin? == true かつ !current_user?(user)であれば，押すとdeleteアクションに飛ぶようなボタンを用意する

!current_user?(user)は，自分自身を消さないための処理

> Web browsers can’t send DELETE requests natively, so Rails fakes them with JavaScript. This means that the delete links won’t work if the user has JavaScript disabled.

PATCHと同様DELETEもブラウザでは標準サポートされていないため，Rails側でJavaScriptを使うことにより偽装する

[link_to に :method =&gt; :delete を指定した時の動作 - happy lie, happy life](http://d.hatena.ne.jp/spitfire_tree/20120323/1332477590)

users_controllerにdestroyを追記して，削除後はusers_urlにリダイレクトするようにする

**ただし，ブラウザ側で管理者のみがdeleteボタンを押せるように工夫をしていても，攻撃者が直接コマンドラインからDELETEリクエストを叩く可能性があるため，ここでもbefore_actionで管理者かどうかをチェックする**

### 9.4.3 User destroy tests

Userが消えるかどうかのテストを書く

ユーザを消すのは (場合によっては) 危険な処理なので，テストをきちんと書いて間違いが起こらないようにする

また，ユーザを削除する処理は管理者しか行えないため，user fixturesに1名管理者を作る (adminカラムをtrueにする)

> first, users who aren’t logged in should be redirected to the login page  

> second, users who are logged in but who aren’t admins should be redirected to the Home page

まず，ログインをしていない人と，ログインはしているが管理者ではないユーザがdestroyを使用とした時に，適切な場所にリダイレクトされるかをテストする

次に，管理者で

1. deleteボタンがviewに表示されているか
2. ``` delete user_path(user) ```を実行すると，ユーザ数が1減るか

をテストする

テストが通ることを確かめる
