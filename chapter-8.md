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
flash.nowはリクエストに限定せず，renderなどの追加の処理が行われた場合にも内容を更新する

先ほど落ちたテストが通ることを確認する
