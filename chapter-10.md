# Chapter 10 Account activation and password reset

> In this chapter, we’ll put the finishing touches on this system by adding two closely related features

> account activation (which verifies a new user’s email address) and password reset (for users who forget their passwords).


この章では

1. ユーザ登録後にそのメールアドレスが正しいものかどうかを検証するため，そのメールに認証URLを送り，そのURLが踏まれた時から登録したユーザを有効にする
2. パスワードを忘れた際に，パスワードリセットをかける (リセット用URLを登録されているメールアドレスに向けて送信する)

という機能をつける

> In the process, we’ll also have a chance to learn how to send email in Rails, both in development and in production.

Railsアプリからメールが送れるようになる

## 10.1 Account activation

1. ユーザ登録後にそのメールアドレスが正しいものかどうかを検証するため，そのメールに認証URLを送り，そのURLが踏まれた時から登録したユーザを有効にする

を実装する

検証をする手続きは以下のように行う

> 1. Start users in an “unactivated” status
> 2. When a user signs up, generate an activation token and corresponding activation digest.
> 3. Save the activation digest to the database, and then send an email to the user with a link containing the activation token and user’s email address.2
> 4. When the user clicks the link, find the user by email address, and then authenticate the token by comparing with the activation digest.
> 5. If the user is authenticated, change the status from “unactivated” to “activated”.

1. はじめに，全ての (未来に登録される) ユーザは非アクティブ状態である
2. もしあるユーザが登録を行った場合，activated tokenとactivation digestを生成する
3. activation digestはDBに保存し，登録フォームで入力されたアドレスに向けて，activation tokenが含まれているようなURLが書かれているメールを送る
4. ユーザがメールで送られてきたURLを踏むと，そのアドレスを元にどのユーザかを検索し，activation tokenとactivation digestの関係がとれているかを認証する
5. 関係がとれていれば，そのユーザを非アクティブ状態からアクティブ状態にする

### 10.1.1 Account activations resource

User Modelに紐づく，account_activations_controllerを生成する

``` rails generate controller AccountActivations --no-test-framework ```

activation digestを編集するアクションは，editに相当する

routes.rbにも，``` resources :account_activations, only: [:edit] ```を追記する

passwordやremember meほどセキュリティ上重要ではないが，それでも多少は重要なため，digestを用意してtokenとの関係を調べる方法を用いる

authenticated?メソッドを色々なtokenを用いて行えるように拡張し，activated?メソッドでアクティブになっているかを確認可能なようにする

また，(Rails Tutorialでは使わないが) 今後のために，いつアクティブになったかを保存するactivated_atカラムも追加をする

activatedは，デフォルトではfalseにする

ユーザが登録フォームでsubmitを押した時，userがcreateされる直前にactivation tokenとactivation digestを発行したい (user登録とactivation token保存で2回DBに問合わせが発生するため，それらを1回におさめたい)  
そのために，before_saveメソッドを用いる

classの途中にprivateと打ち，その下の行にまたメソッドを書き始めた場合，それはprivate methodとなる

メソッドをprivateとして宣言することで，外部から呼び出すことのできないメソッドとなる

> つまり、外部（s2）からs1のメソッドを呼び出そうとするなら、  
> s1.s1_privateのようにs1というレシーバをつける必要があるが（ルール2）、  
> （s1の）privateメソッドはレシーバを付けて呼び出す事を許してくれない（ルール1）。  
> 結果として、privateメソッドは外部から呼び出すことが出来ない。

[[Ruby] privateメソッドの本質とそれを理解するメリット - Qiita](http://qiita.com/kidach1/items/055021ce42fe2a49fd66)

before_saveで，以下のようにactivation tokenとdigestを設定する

``` ruby
self.activation_token  = User.new_token
self.activation_digest = User.digest(activation_token)
```

ここで，対象ユーザをfindしてupdate_attributeをしない理由は，このメソッドがbefore_saveで呼び出されるため，呼び出された時点ではまだユーザはsaveされてないため (selfは作られようとしているユーザ)

> cookiesにtokenを保存するため，user.remember_tokenのようにしてアクセス可能なことが望ましい  
> これは，``` attr_accessor :remember_token ```を宣言することで実現させる (Chapter 4 参照)

また，remember_token同様，activation_tokenもattr_accessorに追記しておく (cookiesに保存するわけではないがアクセス可能にしておく)

fixturesのユーザは，全てactivatedをtrueにしておき，seedを入れなおす

### 10.1.2 Account activation mailer method

Action Mailerライブラリを用いて，users_controller.createが実行された (そして成功した) と同時にメールを送るようにする

以下でmailerを追加する

```  rails generate mailer UserMailer account_activation password_reset ```

同時に，account_activation, password_resetメソッドを追加している

mailerを上の方法で作成することにより自動的に，plain text用のview (送信するメールの本文) と，html用のviewが作成されている

また，テンプレートファイルにmailer用のテンプレートも追加されている (app/views/layouts/)

また，今までのviewと同様，view内でインスタンス変数を利用して表示をすることができる

generateのままでも動くが，カスタマイズしていく

まず，ApplicationMailer (全てのMailerの親) に，デフォルトのfrom (送信元) とテンプレートファイルを指定する

``` ruby
default from: "noreply@example.com"
layout 'mailer'
```

また，UserMailerに，``` account_activation(user) ```メソッドを以下のように追記する

``` ruby
def account_activation(user)
  @user = user
  mail to: user.email, subject: "Account activation"
end
```

送信先がuser.emailに，件名が"Account activation"になるように指定している

認証を進めるにはmail (どのユーザをアクティブにするかを検索する)とtoken (digestとの関連を調べる) が必要であり  
mailはパラメータとして，tokenはbase64でエンコードしたものをurlでのidとして渡す (usersと同じやり方)

また，正しいURLになるよう，mailの@は自動的に%40にエスケープされる

このURLをメールの本文に，erbで埋め込む

メールのviewがどうなっているかを目で確認するため，Railsにはemail previews機能を使うことができ，これにより文面をWeb上で確認することができる

email previewsを利用するには，config/environments/development.rbにて

``` ruby
config.action_mailer.raise_delivery_errors = true
config.action_mailer.delivery_method = :test
host = 'example.com'
config.action_mailer.default_url_options = { host: host }
```

指定し， (hostはexample.comになっているが，本番環境ではきちんと有効なホストを指定する)

generate時に自動生成されていたuser_mailer_preview#account_activationに，UserMailer.account_activationを実行するように追記する

``` ruby
class UserMailerPreview < ActionMailer::Preview

# Preview this email at
# http://localhost:3000/rails/mailers/user_mailer/account_activation
def account_activation
  user = User.first
  user.activation_token = User.new_token
  UserMailer.account_activation(user)
end

...

```

mail previewsのURLを訪れると (例: /host/rails/mailers/user_mailer/account_activation) メールのプレビューが，textとhtmlの両方とも確認可能になっている

また，視認と同時にテストも書く

ただし，メールアドレスをassert_matchするときは，**CGI::escape(user.email)でエスケープをしたものを探す**

また，テストのためにconfig/environments/test.rbでも，development.rbと同様にhostの指定を行う．

メールを送るために，users_controller#createに，以下のようにメールを送信するための行を追加する

``` ruby
if @user.save
  UserMailer.account_activation(@user).deliver_now
  flash[:info] = "Please check your email to activate your account."
  redirect_to root_url
```

ログイン後にshowに飛ぶテストが落ちるが，このテストはもう古くなったため，動かないようにコメントアウトもしくは消しておく

developmentでは実際にメールが送信されることはないが，rails serverのログに表示されるため，そのURLをコピペして閲覧する  
閲覧可能なことを確認する (まだアクティブ化はされない)

### 10.1.3 Activating the account

実際に，ユーザをアクティブにする処理を記述する

params[:id]とparams[:email]でtokenとemailが取得できるため，これを利用してアクティブにする

まず，emailでUser.find_byを行い，対象のユーザを検索する

ユーザが存在する，かつ認証が通ればアクティブにするが，認証が通ってるかのチェックはUser Modelのauthenticated?メソッドを改良し，それを利用する

変更前のauthenticated?メソッドは以下の通り

``` ruby
def authenticated?(remember_token)
  return false if remember_digest.nil?
  BCrypt::Password.new(remember_digest).is_password?(remember_token)
end
```

現在は，これはremember_tokenのチェックのみ行っているため，他のtokenのチェックも行えるメソッドに進化させたい

具体的には，以下のように使えるように進化させたい

``` ruby
user.authenticated?(:activation, token)
```

第一引数としてどのdigestに対してのチェックかを，第二引数に調べるtokenを渡す

進化させるために，sendメソッドを用いる

下の例のように，send(method name)で動的にメソッドを呼び出すことができる

``` ruby
$ rails console
>> a = [1, 2, 3]
>> a.length
=> 3
>> a.send(:length)
=> 3
>> a.send('length')
=> 3
```

これを用いて，

``` ruby
user.send("#{attribute}_digest")
```

と書き，attribute変数に"remember"や"activation"などを渡すことにより，適切なdigestを指すことができる

そのため，authenticated?メソッドを以下のように書き換える

``` ruby
def authenticated?(attribute, token)
  digest = send("#{attribute}_digest")
  return false if digest.nil?
  BCrypt::Password.new(digest).is_password?(token)
end
```

また，こう書き換えることにより今までauthenticated?メソッドを使っていた箇所でエラーが起きるため，適切な形に変更をする

変更後，account_activations_controller#editでも利用し，authenticated?メソッドがtrueであればuser.activatedをtrueに，user.activated_atを現在時刻にしてDBのデータを更新する

また，sessions_controller#craete (ログイン時) で，userがアクティブであるかどうかをチェックし，アクティブであればログイン処理を，そうでなければflashにアクティブではない旨のメッセージを追加し，root_urlにリダイレクトさせる．

### 10.1.4 Activation test and refactoring
