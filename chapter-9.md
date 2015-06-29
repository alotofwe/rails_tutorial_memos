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
