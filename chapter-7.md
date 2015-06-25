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
