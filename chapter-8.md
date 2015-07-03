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

以降，情報を一時的に保持しておくためにsessionメソッドを，永く保持しておくためにcookiesメソッドを使用する

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

> repeat (2^cost)

[bcrypt - Wikipedia, the free encyclopedia](https://en.wikipedia.org/wiki/Bcrypt)

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

**また，テスト内では実際のアプリ内で使っているヘルパは使用できないため**，テスト用にログイン済み判定を行うようなテストを追記する

``` ruby
def is_logged_in?
  !session[:user_id].nil?
end
```

ユーザ登録後にログインしていることを確認するテストを追記し，通ることを確かめる

## 8.3 Logging out

ログアウトを実装する

ログアウトをするとは，session[:user_id]に保存されているuser_idを削除し，nilに戻すこと  
また，@current_userの内容もnilに戻す必要がある

また，ヘルパにログイン処理を書いたように，ログアウト処理も追記する

``` ruby
def log_out
  session.delete(:user_id)
  @current_user = nil
end
```

このヘルパを利用して，sessions_controller#destroyにログアウト処理を書く  
ログアウト後はroot_urlにリダイレクトするようにする

``` ruby
def destroy
  log_out
  redirect_to root_url
end
```

また，ログアウト後にきちんとログイン状態が解除されており，再度ログインボタンが表示されていることをテストで確認し，通ることを確かめる

## 8.4 Remember me


> 1. ブラウザを閉じた時にログアウトしている状態にする
> 2. ブラウザを閉じてもユーザがログインしていたことを覚えておき，次回訪問時に自動ログインする
> 3. 1にするか2にするかをユーザが選択できるようにする

の3を実装する (このような機能をremember meと呼ぶ)

### 8.4.1 Remember token and digest

> In this section, we’ll take the first step toward persistent sessions by generating a remember token appropriate for creating permanent cookies using the cookies method, together with a secure remember digest for authenticating those tokens.

remember tokenを発行し，cookiesにそれを使って生成したremember digestを記憶させることで実装する

ログインにはsessionを使ったが，これはブラウザが閉じられると消えてしまうデータなので，ここではcookiesメソッドを使用する

ただしcookiesは暗号化を行わないため，

1. 流れているパケットを覗き見する
2. DBが漏洩する
3. XSSをする
4. ログイン済みの端末を利用する

などでクッキー内容が盗まれる危険がある  
盗まれると，攻撃者がユーザになりすますことができる

[compromise “情報漏洩”を英語で何というべきか？](http://micottan.blogspot.jp/2014/11/compromise.html)

上の問題を考慮して，今回は以下のようにremember meを実装する

1. remember tokenとして，ランダムな文字列を生成する
2. cookiesに1で生成したtokenを，有効期限と共に記憶させる
3. DBにtokenを暗号化したものを保存する
4. user_idを暗号化してcookiesに記憶させる
5. user_idを持ったcookiesを持ったリクエストが届いた際，そのidを元にUser.find_byを行う
6. cookiesのtokenが，DBのdigestにきちんと関連付けられているものかをチェックする

DBにremember digestを保存するため，Userにカラムを追加する

``` rails generate migration add_remember_digest_to_users remember_digest:string ```

![Figure8.9](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/user_model_remember_digest.png)

また，1のランダムな文字列は，``` SecureRandom.urlsafe_base64 ```を使用して生成する

この文字列はA-Z, a-z, 0-9, "-", "_"の文字しか使用しない (合計64文字なので，つまりbase64)

remember tokenはユニークである必要は無いが，ユニークに近いほうがセキュアである (偽装しにくい)

tokenの暗号化によるdigestの生成は，既にUser Modelにdigestメソッドがあるため，これを流用する  
また，tokenを発行するためのメソッドも追記しておく

cookiesにtokenを保存するため，user.remember_tokenのようにしてアクセス可能なことが望ましい  
これは，``` attr_accessor :remember_token ```を宣言することで実現させる (Chapter 4 参照)

User Modelに以下の様なメソッドを追記することで，remember meを有効にできるようにする  

``` ruby
def remember
  self.remember_token = User.new_token
  update_attribute(:remember_digest, User.digest(remember_token))
end
```

selfをつけることでローカル変数であることを明示的に指定している

### 8.4.2 Login with remembering

2. cookiesに1で生成したtokenを，有効期限と共に記憶させる

を実装する

以下のように，有効期限と共に保存する

``` ruby
cookies[:remember_token] = { value:   remember_token,
                             expires: 20.years.from_now.utc }
```

また，Railsでは同じことを以下のようにも書くことができる

``` ruby
cookies.permanent[:remember_token] = remember_token
```

-----

### Box 8.2. Cookies expire 20.years.from_now

Fixnumクラスにはtime helperがある

``` ruby
>> 1.year.from_now
=> Sun, 09 Aug 2015 16:48:17 UTC +00:00
>> 10.weeks.ago
=> Sat, 31 May 2014 16:48:45 UTC +00:00
```

また，以下のようなこともできる

``` ruby
>> 1.kilobyte
=> 1024
>> 5.megabytes
=> 5242880
```

自分でわざわざ計算する必要がないので便利

-----

また，user_idに署名を追加してcookiesに記憶させるため，.signedを使用する

> Returns a jar that'll automatically generate a signed representation of cookie value and verify it when reading from the cookie again. This is useful for creating cookies with values that the user is not supposed to change. If a signed cookie was tampered with by the user (or a 3rd party), nil will be returned.

[ActionDispatch::Cookies::ChainedCookieJars](http://api.rubyonrails.org/classes/ActionDispatch/Cookies/ChainedCookieJars.html)

``` ruby
ookies.permanent.signed[:user_id] = user.id
```

tokenがdigestと関連しているものかどうかを調べる必要があるが，これは以下のようにして調べることができる

``` ruby
BCrypt::Password.new(remember_digest) == remember_token
```

remember_digestを復号化してtokenと比較する方法が考えられるが，bcryptで暗号化したものは復号化することができない  
そのためbcryptは``` == ```を再定義している

また，以下のように記述しても同様に動く

``` ruby
BCrypt::Password.new(remember_digest).is_password?(remember_token)
```

remember tokenとdigestの関連を確認するメソッドをUser Modelに追記し，SessionsHelperにcookiesにuser_idとremember_tokenを記憶させるメソッドを追記する

また，current_userに，remember meが設定されていたら自動的にログインする機能を追加する

``` ruby
def current_user
  if (user_id = session[:user_id])
    @current_user ||= User.find_by(id: user_id)
  elsif (user_id = cookies.signed[:user_id])
    user = User.find_by(id: user_id)
    if user && user.authenticated?(cookies[:remember_token])
      log_in user
      @current_user = user
    end
  end
end
```

この時点で，テストが落ちる (しばらく放置)

### 8.4.3 Forgetting users

remember meを解除するため，forgetメソッドをUser ModelとSessionsHelperに追記する

User Modelではremember_digestをnilに更新し，SessionsHelperではcookiesのuser_idとremember_tokenを削除する

### 8.4.4 Two subtle bugs

更新したcurrent_userメソッドに2つのバグの可能性があるため，修正する

> The first subtlety is that, even though the “Log out” link appears only when logged-in, a user could potentially have multiple browser windows open to the site.
> If the user then logged out in one window, clicking the “Log out” link in a second window would result in an error due to the use of current_user in Listing 8.39.

1つめは，複数のウィンドウでページを表示していた時，ウィンドウaでログアウトした後，ウィンドウbでログアウトボタンを押した時に，current_userが無いために起こる  
これは，ログインしている場合のみログインを行うというifを追記することで対処する

``` ruby
def destroy
  log_out if logged_in?
  redirect_to root_url
end
```

> The second subtlety is that a user could be logged in (and remembered) in multiple browsers, such as Chrome and Firefox, which causes a problem if the user logs out in one browser but not the other.

2つめは，複数のブラウザで，remember_meをonにした状態でログインしていた時，ブラウザaがログインしていてブラウザbがログアウトしている場合に起こる

``` ruby
def current_user
  if (user_id = session[:user_id])
    @current_user ||= User.find_by(id: user_id)
  elsif (user_id = cookies.signed[:user_id])
    user = User.find_by(id: user_id)
    if user && user.authenticated?(cookies[:remember_token])
      log_in user
      @current_user = user
    end
  end
end
```

1. aがcurrent_userを実行しようとする
2. ``` if (user_id = session[:user_id]) ```は，**sessionの中身はブラウザbで消されているためnilとなり**，この行の評価はfalseになる  
  **※ sessionは，データ自体はサーバ側で管理し，管理しているデータを呼び出すためにcookieにkeyを保存する**
3. ``` elsif (user_id = cookies.signed[:user_id]) ```は，**cookieの中身はブラウザごとに独立であり，ブラウザbがログアウトしてもブラウザaのcookieの内容は消されないため**，user_idには先程までログインしていたユーザのidが入り，評価はtrueになる
4. user_idに存在するユーザのidが入っているため，findでユーザが検索されuserに代入される
5. ``` if user && user.authenticated?(cookies[:remember_token]) ```において，``` user ```は正しい値が入っているので評価はtrue，``` user.authenticated?(cookies[:remember_token]) ``` は，authenticate?メソッドは以下のようなものであり

``` ruby
def authenticated?(remember_token)
  BCrypt::Password.new(remember_digest).is_password?(remember_token)
end
```

remember_digestはブラウザbのログアウト時にnilに更新されており，BCrypt::Password(nil)は例外を返すため，これがバグとなる

対策としては，authenticated?メソッドのBCrypt::Passwordの前の行に，remember_digestがnilであれば即falseを返すような行を追記する

``` ruby
def authenticated?(remember_token)
  return false if remember_digest.nil?
  BCrypt::Password.new(remember_digest).is_password?(remember_token)
end
```

どちらのバグも，テストを書いて落ちることを確かめ，修正する

[Railsのセッション管理方法について - Programming log - Shindo200](http://shindolog.hatenablog.com/entry/2014/11/02/164118)

### 8.4.5 “Remember me” checkbo

Remember meの機能ができたので，実際にWebから使えるようにする (チェックボックスを設置する)

form_forにf.check_boxがあるので，それを利用する

そして，form_forを使用するとparamsに値が'0'か'1'で入るため，'1'であればrememberメソッドを，そうでなければforgetメソッドを使うようにする  
0と1ではないので，``` if params[:session][:remember_me] ```は常にtrueとなる点に注意

> there are 10 kinds of people in the world: those who understand binary and those who don’t.

### 8.4.6 Remember tests

remember meの，チェックボックスにチェックした時としていない時のテストを書く

**post_via_redirectメソッドはintegration testでしか使えないため，これが使えるかどうかでそのテストがintegration testかどうかを判別する**

integration testであればpostから，そうでなければsession[:user_id]にidを入れることでログインを行う

> There’s one more subtlety, which is that for some reason inside tests the cookies method doesn’t work with symbols as keys, so that

> cookies[:remember_token]

> is always nil. Luckily, cookies does work with string keys, so that

> cookies['remember_token'] 

> has the value we need.

some reason (調べてみたけど理由はよく分からない) によりテスト内でcookieを使うときにkeyとしてシンボルを使うと常にnilになるので，文字列を使う

また，current_userでテストされていない箇所があるため，対策をする
