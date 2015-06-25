# Chapter 7 Sign up

モデリングを終えたので，Webからユーザを閲覧・登録できるようにする

> In this chapter, we’ll rely on the User model validations from Chapter 6 to increase the odds of new users having valid email addresses. In Chapter 10, we’ll make sure of email validity by adding a separate account activation step to user signup.

Chapter 6でバリデーション等を終えたのでそれを信頼していればよく，準備万端

## 7.1 Showing users

まず，ページからユーザ情報を閲覧できるようにする

### 7.1.1 Debug and Rails environments

本格的に動的ページを作る

Webで開発している時にいちいちparamsを見るのは面倒なので，envはdevelopmentの時のみ，Webページ下部にデバッグ情報を出力させる

デバッグ情報は，debug()を使うことで簡単に表示することができる  
実際のコードは以下のとおり

``` <%= debug(params) if Rails.env.development? %> ```

-----

### Box 7.1

rails cでenvを指定する方法は以下のとおり

``` rails console test ```

rails sでenvを指定する方法は以下のとおり
``` rails server --environment production ```

db:migrate等，rakeでenvを指定する方法は以下のとおり

``` bundle exec rake db:migrate RAILS_ENV=production ```

また，Heroku上ではproductionで動いている

-----

CSSで見た目を整え，デバッグ出力がWeb上に表示されていることを確認する

### 7.1.2 A Users resource

ユーザ情報を閲覧するために，rails cでユーザを1つ作っておく

resources (Box 2.2を参照) を用いてリソースを表示させる際は，ユニークな識別子と共にリクエストを送る (idを用いる)

user resourcesを表示することができるように．routes.rbに以下の一行を追加

``` ruby
resources :users
```

前の章で学んだ通り，index, show, new, create, edit, update, destroyアクションが使えるようになる  
ただし，controllerに自動的にメソッドが追加されるということはないので手動で追加する

リソースの情報を表示するときはshowを用いる  
showのviewを簡単に書いておき，controllerにshow methodを追加する

show actionでは，求められたユーザの情報をDBから検索し，結果をインスタンス変数として代入する

id (urlから指定したユーザのid) はparams[:id]に入っているため，これを用いてUser.find()を行う

params[:id]はStringであり，DBに入っているidはIntegerだが，findメソッドは適切に解釈してくれるためto_iなどを行う必要は無い

paramsに何が入っているのかは，先ほどから表示させるようにしたデバッグ出力で確認することができる

### 7.1.3 Debugger

更に詳細な情報を得るためのデバッガが存在する

処理を一旦止めたい行に``` debugger ```という一行を挿入すると，その行に入った時に処理が一旦止まり，rails sを実行している窓に``` (byebug) ```と表示される

この状態になると，rails cのように色々な情報を表示させることができる

### 7.1.4 A Gravatar image and a sidebar

"globally recognized avatar" (Gravatar) を用いて，ユーザの画像を扱う

Gravatarは，画像のアップロード・編集・保存を管理する

まず，画像を表示させるためのヘルパとしてgravatar_forを作成する

> As noted in the Gravatar documentation, Gravatar URLs are based on an MD5 hash of the user’s email address. In Ruby, the MD5 hashing algorithm is implemented using the hexdigest method, which is part of the Digest library:

GravatarはemailをMD5を用いて暗号化したものを使っている
これを行うライブラリはRubyにも存在する

``` ruby
>> email = "MHARTL@example.COM".
>> Digest::MD5::hexdigest(email.downcase)
=> "1fda4469bcbec3badf5418269ffc5968"
```

gravatar_forの内容は以下のとおり

``` ruby
def gravatar_for(user)
    gravatar_id = Digest::MD5::hexdigest(user.email.downcase)
    gravatar_url = "https://secure.gravatar.com/avatar/#{gravatar_id}"
    image_tag(gravatar_url, alt: user.name, class: "gravatar")
end
```

emailをMD5で暗号化し，それをidとしてURLに含め，image_tagで表示させている

モックアップではスライドバーが描かれていたため，それをviewに反映させていく

Bootstrapで指定されているclassを使うことで，簡単に実装することができる

## 7.2 Signup form

登録フォームを作っていく

> The goal of this section is to start changing this sad state of affairs by producing the signup form mocked up in Figure 7.11.

殺風景な現在のフォーム画面をモックアップ通りに華やかにしていく

先ほど作成したユーザの情報は必要がないため，下のコマンドでDBのデータを初期化する

``` ruby
bundle exec rake db:migrate:reset
```

DBをいじった場合は，サーバを再起動する必要があることがある

### 7.2.1 Using form_for

フォームを簡単に作成するため，form_forヘルパを用いる  
form_forはActive Recordのデータを利用するため，controllerでそのデータを用意する

``` ruby
def new
    @user = User.new
end
```

その後，以下のようにform_forでフォームを生成していく  
詳しい内容は，Section 7.2.2で詳しく扱う

``` ruby
<%= form_for(@user) do |f| %>
      <%= f.label :name %>
      <%= f.text_field :name %>

      <%= f.label :email %>
      <%= f.email_field :email %>

      <%= f.label :password %>
      <%= f.password_field :password %>

      <%= f.label :password_confirmation, "Confirmation" %>
      <%= f.password_field :password_confirmation %>

      <%= f.submit "Create my account", class: "btn btn-primary" %>
<% end %>
```

### 7.2.2 Signup form HTML

form_forはいくつかのRuby・erbの機能を用いる

特に，do ~ endで書かれており，ブロック内で使える変数としてfが宣言されている

> As is usually the case with Rails helpers, we don’t need to know any details about the implementation, but what we do need to know is what the f object does: when called with a method corresponding to an HTML form element—such as a text field, radio button, or password field—f returns code for that element specifically designed to set an attribute of the @user object. In other words,

fが何者であるかについて詳しく知る必要はないが，fを通じてcontrollerで宣言した@userを操作することができるようになる

**type="email"はメールアドレスの内容に対して適切なアドバイス (不正な部分等) を表示し，type="password"は入力された文字を直接表示しない**

form_forから生成されたhtmlのinputタグを見ると，name=""を適切に指定することによりフォームの初期値を設定していることが分かる (Section 7.3で詳しく述べる)

また，formタグを見ると，@userのクラスに応じて適切にaction等が指定されていることが分かる  
これは，Railsが各オブジェクトのクラスを把握することができることによる

また，自動的にCSRF対策などがされていることがhtmlから分かる

## 7.3 Unsuccessful signups

フォームを作成したため，不正な入力を行った際に何が起こるかを把握する

> In this section, we’ll create a signup form that accepts an invalid submission and re-renders the signup page with a list of errors, as mocked up in Figure 7.14.

ここでは，不正な入力を受け取った際にユーザ登録フォームをもう一度表示し，どの部分の入力が不正かを表示させる．

### 7.3.1 A working form

form_forから生成されたformタグを見ると分かる通り，フォームが送信された後はuser controllerのcreateメソッドに飛ぶ (post /users)

user controllerのcreateメソッドでは，paramsから入力されたデータを取り出し，それを元にユーザをsaveする  
入力されたデータはparams[:user]に記憶されている

その後，処理の成功/失敗に応じてif-elseで表示させるhtmlを分岐させる  
ここでは，user.saveの返り値を用いるとよい

実際に不正な入力でフォームから送信し，例外が発生していることを確認する

デバッグ出力からparamsを確認すると，以下の様なものが渡されていることが分かる (一例)

``` ruby
"user" => { "name" => "Foo Bar",
            "email" => "foo@invalid",
            "password" => "[FILTERED]",
            "password_confirmation" => "[FILTERED]"
          }
```

**改めてinputタグのnameを確認してみると，user[email]はparamsのuser要素のemailというkeyのvalueとして入力内容を保存するという宣言になっていることが分かる**

そして，入力はparamsに正しく渡されていることも確認することができる

なぜこれでエラーが出るかというと，Rails 4.0からセキュリティ上の理由で，paramsをそのまま引数として渡すとエラーが出るようになっている

### 7.3.2 Strong parameters

ここでは，簡単にmass assignmentについて説明したあと，エラーの解消方法について説明を行う

> The reason is that initializing the entire params hash is extremely dangerous—it arranges to pass to User.new all data submitted by a user.
> In particular, suppose that, in addition to the current attributes, the User model included an admin attribute used to identify administrative users of the site.

paramsから直接DBの変更を行おうとすると，危険な入力内容も受け入れることになる  
特に管理者権限の有無を自由に変更される危険がある

> The way to set such an attribute to true is to pass the value admin=’1’ as part of params[:user], a task that is easy to accomplish using a command-line HTTP client such as curl.

例えば，入力内容として"admin='1'"と入力され，かつadminカラムが存在していた場合，adminカラムが1に書き換えられてしまう  

> This allows us to specify which parameters are required and which ones are permitted.
> In addition, passing in a raw params hash as above will cause an error to be raised, so that Rails applications are now immune to mass assignment vulnerabilities by default.

これを防ぐためにstrong parametersというテクニックが導入され，これは更新するカラムを指定し，それ以外の更新を行わないというテクニックである．

これは，以下のように実装する

``` ruby
params.require(:user).permit(:name, :email, :password, :password_confirmation)
```

上記のコードは，paramsのうちuser要素のみを使い，かつ:name, :email, :password, :password_confirmationカラムのみの更新を許すというものである

簡単のために，対策方法をprivate methodにしておく

privateはSection 8.4で詳しく説明される

この対策を行った後，Web上で例外が発生したことが表示されないことを確認する

### 7.3.3 Signup error messages

Section 6.2.2で述べたとおり，保存時にエラーが発生した場合はerrors.full_messagesにエラー内容が格納される

ここでは，その内容をWeb上に表示させ，その部分を部分テンプレート化する

一般的に複数のコントローラから使用される可能性のあるviewは，sharedディレクトリの中に格納される

errors.countでエラーの個数，errors.(any?|empty?)でエラーが発生しているかどうかを確認することができる

**また，pluralizeヘルパを用いる事により，単数形/複数形の表示が簡単になる**

``` ruby
>> include ActionView::Helpers::TextHelper
>> pluralize(1, "error")
=> "1 error"
>> pluralize(5, "error")
=> "5 errors"
>> pluralize(2, "woman")
=> "2 women"
>> pluralize(3, "erratum")
=> "3 errata"
```

指定されたコードを書き，実際のWeb上でエラー内容が表示されることを確認する

### 7.3.4 A test for invalid submission

不正な入力を受け付けた際のintegration testを行う

``` rails generate integration_test users_signup ```

integration test fileを追加する

> The main purpose of our test is to verify that clicking the signup button results in not creating a new user when the submitted information is invalid.

get/post等のリクエストを行う際は，以下のように書く

``` get target_path ```

不正な入力をpostしてもユーザの数が増えないかどうかを確認するテストは，以下の様に書く

``` ruby
assert_no_difference 'User.count' do
  post users_path, user: { name:  "",
                           email: "user@invalid",
                           password:              "foo",
                           password_confirmation: "bar" }
end
```

postの第二引数でparamsの内容を指定することができる

assert_no_differenceは，ブロック内を実行後に，User.countが変化しないことを期待している

テストが通ることを確認する

## 7.4 Successful signups

次は，正しい入力でpostされたとき，ユーザをDBに登録してshowページにリダイレクトをする機能を実装する

また，リダイレクト後にflashで文章を表示させる

### 7.4.1 The finished signup form

正しい入力内容が来た時，user_controller#createにて，showページにリダイレクトを行う旨を追記する

actionでの操作後に別のページに移動させたい場合は，``` redirect_to ```を使う

また，resourcesを宣言している場合，``` redirect_to @user ```は自動的に``` redirect_to user_url(@user) ```であると解釈する

### 7.4.2 The flash

