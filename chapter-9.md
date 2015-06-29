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
