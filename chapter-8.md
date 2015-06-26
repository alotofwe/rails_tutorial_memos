# Chapter 8 Log in, log out

ログイン・ログアウトを実装する

We’ll be implementing all three of the most common models for login/logout behavior on the web: “forgetting” users on browser close (Section 8.1 and Section 8.2), automatically remembering users (Section 8.4), and optionally remembering users based on the value of a “remember me” checkbox (Section 8.4.5).

実装にあたり，3つのやり方が考えられる

1. ブラウザを閉じた時にログアウトしている状態にする
2. ブラウザを閉じてもユーザがログインしていたことを覚えておき，次回訪問時に自動ログインする
3. 1にするか2にするかをユーザが選択できるようにする

また，ログインしているかどうかに応じてページの表示を変えるようにする

> This is a long and challenging chapter covering many detailed aspects of login common systems, so I recommend focusing on completing it section by section. In addition, many readers have reported benefiting from going through it a second time.

ちょっと大変だけど頑張ろう!

## 8.1 Sessions

HTTPはステートレスなプロトコルであるため，前のリクエストの内容を覚えておくことができない  
そのためHTTPだけでは，ユーザがログインしているかの情報を保持しておくことができない

情報を保持しておくため，Railsではcookiesを利用する  
ここに情報を入れることで，情報を保持しておくことができる (ページが変わっても前の情報を参照することができる)

> In this section and Section 8.2, we’ll use the Rails method called session to make temporary sessions that expire automatically on browser close,2 and then in Section 8.4 we’ll add longer-lived sessions using another Rails method called cookies.

以降，情報を一時的に保持しておくためにsessionメソッドを，永く保持しておくためにcookieメソッドを使用する

ログイン時にセッションを新しく作り，ログアウト時にそれを消去するようにする

### 8.1.1 Sessions controller 

``` rails generate controller Sessions new ```

Sessions Controllerを作ってREST的にログイン・ログアウトを行う

ログインフォームはnew，ログインはcreate，ログアウトはdestroyに相当する

routes.rbに追記するが，全てのRESTfulなアクションを利用するわけではないため，resourcesではなくアクションごとに個別で追記する

``` ruby
get    'login'   => 'sessions#new'
post   'login'   => 'sessions#create'
delete 'logout'  => 'sessions#destroy'
```

``` bundle exec rake routes ```で全てのルートを確かめることができる

### 8.1.2 Login form

ログインフォームを作成する

ユーザ登録フォームのように，form_forを用いてviewを作成する

> The main difference between the session form and the signup form is that we have no Session model, and hence no analogue for the @user variable. This means that, in constructing the new session form, we have to give form_for slightly more information

Session modelが無く，登録フォームのように``` form_for(@session) ```ができないため，**代わりに``` form_for(:session, url: login_path) ```という風に明示的にリソースの名前とポスト先URLを書く**

リソースの名前を指定すると，ポスト先でparams[:リソースの名前][:カラム名]という風に情報を取り出すことができる

### 8.1.3 Finding and authenticating a user

まず，ログインで不正な入力が行われた時の処理を追記する

不正な入力の場合は，ログインせずにログインフォームに戻り，不正だったことを表示させる

有効なemailとpasswordのペアが送られてきた時のみ，ログイン成功とする

とりあえずpostをしてみると，パラメータが以下のようになっていることをWebのデバッグ出力から確認することができる

```
session:
  email: 'user@example.com'
  password: 'foobar'
commit: Log in
action: create
controller: sessions
```

ここから，createではparams[:session]の情報を利用してユーザを検索すればよいことが分かる

emailはuniqueなので，emailでUser.find_byを，authenticateメソッドでpasswordが正しいかどうかを確認する

``` ruby
def create
  user = User.find_by(email: params[:session][:email].downcase)
  if user && user.authenticate(params[:session][:password])
    # Log the user in and redirect to the user's show page.
  else
    # Create an error message.
    render 'new'
  end
end
```

ifでは，対象がnilもしくはfalseではない場合に真となる  

### 8.1.4 Rendering with a flash message

Session Modelが存在しないため，ユーザ登録フォームのようにerrors.full_messagesからどこがどう間違っていたかの情報を得ることができない

そのためここでは，手動で「入力が不正である」旨のメッセージを入れる

flashのkeyをdangerとすると，Bootstrapがよしなに良い見た目にしてくれる

ただし，以下の様な書き方は良くない

``` ruby
def create
  user = User.find_by(email: params[:session][:email].downcase)
  if user && user.authenticate(params[:session][:password])
    # Log the user in and redirect to the user's show page.
  else
    flash[:danger] = 'Invalid email/password combination' # Not quite right!
    render 'new'
  end
end
```

**renderはリクエストとみなされず，flashの内容更新はリクエスト毎に行われるため，他のページに移動してもflashの情報が残ってしまう**

### 8.1.5 A flash test

flashのバグを直す前に，直ったかどうかを簡単に判定できるようintegration testを書く

``` rails generate integration_test users_login ```

テストで，不正な入力後，他のページに飛び，そこでflashが表示されていないことを確認するテストを書き，落ちることを確認する

``` bundle exec rake test TEST=test/integration/users_login_test.rb ```

**TESTでどのテストを実行するかを指定することが可能**

そして，flashのバグを解消するためには，flash.nowを使用する  
**flash.nowはリクエストに限定せず，renderなどの追加の処理が行われた場合にも内容を更新する**

先ほど落ちたテストが通ることを確認する

## 8.2 Logging in

ログインフォームの内容が適切だった際にログインを行う処理を書く

この段階では，ログイン済みのユーザがブラウザを閉じた時に，自動的にログアウトとなる

SessionsHelperを作り，そこにログイン機能などを追記することにより，Controllerの記述を便利にする

SessionsHelperは，ApplicationController (全てのControllerの基礎となる) にincludeすることで，あらゆるcontroller内で利用できるようにする

### 8.2.1 The log_in method

ユーザがログインしているかどうかを判別するために，session[:user_id]にログインしているユーザのidを入れることにする (ログインしていなければnil)

**sessionはcookiesに情報を，一時的に，暗号化して保存する**  
**ブラウザが閉じられると，sessionに保存した内容は消去される**

ログイン機能は何回も違う箇所で使われる可能性があるため，SessionsControllerに書いて使いまわせるようにする

**sessionはcookieと違い暗号化されるため，session hijackingを防ぐことができる**

sessions_controller#createで，作成したlog_inメソッドを使用してログインする

### 8.2.2 Current user

current_userメソッドで現在ログインしているユーザを取得し，@current_userに代入するようにする

session[:user_id]を使用してfind_byする

**findは存在しないidが渡された時に例外を吐き，ユーザがログインしていない時にsession[:user_id]はnilとなるため，例外が起こらないようにfind_byを使用する**

メソッドは以下のようになる

``` ruby
def current_user
  @current_user ||= User.find_by(id: session[:user_id])
end
```

既に@current_userに情報が入っている状態でもう一度DBに調べに行くのは無駄なので，@current_userに既に値が入っていればそれを使い，そうでなければfind_byをする


``` @current_user ||= User.find_by(id: session[:user_id]) ```は，``` @current_user = @current_user || User.find_by(id: session[:user_id]) ```に等しい
このメソッドをSessionsHelperに追記しておくこのメソッドをSessionsHelperに追記しておくこのメソッドをSessionsHelperに追記しておくこのメソッドをSessionsHelperに追記しておくこのメソッドをSessionsHelperに追記しておくこのメソッドをSessionsHelperに追記しておくこのメソッドをSessionsHelperに追記しておくこのメソッドをSessionsHelperに追記しておく
@current_userが真の場合 (nilでもfalseでもない場合) ，``` || ```の右は評価されない

このメソッドをSessionsHelperに追記しておく

### 8.2.3 Changing the layout links

レイアウトファイルで表示させるリンクを，ログインしている時としていない時で変化させる

ログインしている時はログアウトへのリンクを，そうでないときはログインのリンクを，ドロップダウンメニューで表示させる

view内で，ログイン済み判定によりif-elseで内容を変えることで実装する  
これの実装のため，ログイン済み判定を行うヘルパを追記する

@current_userがnilではない (current_userメソッドがnilを返さない) 場合にログインしているため，ヘルパは以下のようになる

``` ruby
def logged_in?
  !current_user.nil?
end
```

レイアウトファイルでリンク先URLを仮に'#'としていたところに，適切なnamed routesで置き換える

**ドロップダウンメニューを利用するため，application.jsに``` require bootstrap ```を追記する**

### 8.2.4 Testing layout changes

1. Visit the login path.
2. Post valid information to the sessions path.
3. Verify that the login link disappears.
4. Verify that a logout link appears
5. Verify that a profile link appears.

をintegration testでチェックする

このテストを行うためにはログインするべきユーザがテスト用DBに保存されている必要がある  
Railsでは，テスト用DBに保存するデータの宣言にfixturesを使う

ログイン用フォームには正しい値を入れる必要が有るため，テスト用DBに入れるデータも正しいものであるべき

email, name, password_digestに正しい値を指定する

**password_digestの指定には，``` BCrypt::Password.create(string, cost: cost) ```を使用する**

**stringには暗号化したい文字列を，costにはどのくらい強力に暗号化を行うかを指定する (その分暗号化にかかる時間が長くなる)**

本番環境ではある程度強力にする必要があるが，今回はテスト用データであるため，最小コストを指定する

最小コストの指定方法は以下のとおり

``` ruby
cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                              BCrypt::Engine.cost
```

本番環境では適切なコストを，そうでない場合は最小コストを使用する

User Modelにクラスメソッドとしてパスワード生成メソッドを用意しておく

``` ruby
def User.digest(string)
  cost = ActiveModel::SecurePassword.min_cost ? BCrypt::Engine::MIN_COST :
                                                BCrypt::Engine.cost
  BCrypt::Password.create(string, cost: cost)
end
```

これを利用し，fixture fileは以下のようになる

``` ruby
michael:
  name: Michael Example
  email: michael@example.com
  password_digest: <%= User.digest('password') %>
```

暗号化された状態から平文に戻したものを求めることはできないため，fixturesではパスワードは全て``` password ```で統一する

fixturesに定義したデータは以下のようにして取り出す  

``` ruby
@user = users(:michael)
```

ログイン状態でlogin_pathへのリンクが存在しないことを，テストを通すことによって確かめる

```
$ bundle exec rake test TEST=test/integration/users_login_test.rb \
>                       TESTOPTS="--name test_login_with_valid_information"
```

TESTOPTSで実行するメソッドの名前を指定することが可能

### 8.2.5 Login upon signup

ユーザ登録後に自動的にログイン状態になるようにする

users_controller#createに，作ったばかりの@userでログインを行うように追記すればよい  
sessions_controllerのlog_inメソッドを流用する

**また，テスト内ではヘルパは使用できないため**，テスト用にログイン済み判定を行うようなテストを追記する

``` ruby
def is_logged_in?
  !session[:user_id].nil?
end
```

ユーザ登録後にログインしていることを確認するテストを追記し，通ることを確かめる
