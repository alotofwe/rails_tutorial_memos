# Chapter 6 Modeling users

> In this chapter, we’ll take the first critical step by creating a data model for users of our site, together with a way to store that data.

データモデルを作成することをモデリングと呼ぶ

この期の流れのざっくりした説明

ユーザ管理機能を持つgemもいくつか存在するが，理解のためにここでは手で実装する  

## 6.1 User model

> Thus, the first step in signing up users is to make a data structure to capture and store their information.

ユーザ情報を保存するためのデータ構造から作り始めていく

> In Rails, the default data structure for a data model is called, naturally enough, a model (the M in MVC from Section 1.3.3).

データモデルのことは，Railsでは単に「Model」と呼ぶ

> The default Rails solution to the problem of persistence is to use a database for long-term data storage, and the default library for interacting with the database is called Active Record

データを長期的に保存するためにDBを使う

DBを (インタラクティブに) 操作するためのライブラリとしてActive Recordが存在する  
これにより大体の場合においてSQLを扱う必要は無くなる

> Moreover, Rails has a feature called migrations to allow data definitions to be written in pure Ruby, without having to learn an SQL data definition language (DDL).

migrationsによって，Rubyを書くことに寄ってデータ構造を定義する (DDLを学ぶ必要がない)

### 6.1.1 Database migrations

> Our goal in this section is to create a model for users that won’t disappear quite so easily.

Modelを定義してデータを保存できるようにする

Modelではattributesを宣言してgetterとsetterを生成する必要は無い

下の図のように，tableにcolumnsが定義されており，各データがrowとなる

![Figure6.2](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/users_table.png)

データモデルを定義する

``` rails generate model User name:string email:string ```

**キャメルケース，単数形でモデル名を宣言する (controllerは複数形であったことに注意)**

その後，オプションとしてカラムの名前と型を宣言することができる

上のコマンドにより，マイグレーションファイルが生成される　

ファイルでは，changeメソッドでDBが変更されることを宣言し，create_tableでテーブルを生成することを宣言している  
create_tableで複数形になっているのは，(例えば) userがテーブルにいくつも追加される，つまり「users」が入るテーブルであるため

オプションにカラムを含めると，マイグレーションファイルに自動でそれらを追加するスクリプトが追記される

また，自動的にtimestampsを追加する行も追加される  
これによりcreated_atとupdated_atが追加される

created_atにはデータの作成日時，updated_atにはデータの最終更新日時を入れる

``` bundle exec rake db:migrate ```

DBにマイグレーションファイルの内容を反映させる

マイグレーションファイルにidについての記述がないが，Railsでは自動的にidカラムを追加し，これが主キーとなる

**もしマイグレーション後にミスが発覚したら，```  bundle exec rake db:rollback ```で元に戻す**

また，create_tableの代わりにdrop_tableを用いることで，テーブルを消すことも可能

migration up (db:migrateすること) とmigration down (db:rollbackすること) で違うことを行わせたい場合は，up, downメソッドを使う

[Active Record Migrations — Ruby on Rails Guides](http://edgeguides.rubyonrails.org/active_record_migrations.html)

### 6.1.2 The model file

Userのためのmodel fileが生成されている (ActiveRecort::Baseを継承していることを確かめよ)

### 6.1.3 Creating user objects

``` rails console --sandbox ```

このrails consoleの実行終了後に，ここで行ったDBへの操作を全てなかったことにする (roll back (undo) する)

**--sandboxを付けなければ，DBへの変更はそのままアプリで使用しているDBに反映される**

1. initialization hashを与えなければ全てのカラムがnilであること
2. 与えるとその値が反映されていること
3. .valid?で不正ではない (保存可能である) データかどうかを確認することができること，
4. saveでtrueが返ってくること (保存できていること)
5. 保存するとid, created_at, updated_atに値が入ることを確認すること
6. user.columnで各カラムの値が返ること
7. saveしたデータはuser.destroyで削除することができること
8. 削除されたデータもその後メモリに残ること (DBからは消えたが，代入した変数が生きている限り参照はできること)

を確認する

saveをすると実行されたSQL文が表示される

saveの返り値は，保存が正常に完了した時はtrue，失敗し保存ができなかった時はfalseを返す

### 6.1.4 Finding user objects

保存したデータを検索する

``` User.find(1) ```

idが1のユーザを検索し，そのユーザのデータを返す

存在しないidを指定した場合，``` ActiveRecord::RecordNotFound: Couldn't find User with ID=n ```という例外が発生する

``` User.find_by(email: "mhartl@example.com") ```

id以外で検索する時はfind_byを使用する

**find_byで指定した内容に当てはまる複数のユーザが存在した際は，一番若いidのものを返す**

``` User.first ```

idが一番若いユーザを返す

``` User.all ```

全てのユーザを返す

ArrayではなくActiveRecode::Relationで返る (この後，また検索をつなげることなどができる)

### 6.1.5 Updating user objects

既にDBに保存されているデータのカラム内容を変更する

2つの方法がある

1. 方法1

``` ruby
>> user           # Just a reminder about our user's attributes
=> #<User id: 1, name: "Michael Hartl", email: "mhartl@example.com",
created_at: "2014-07-24 00:57:46", updated_at: "2014-07-24 00:57:46">
>> user.email = "mhartl@example.net"
=> "mhartl@example.net"
>> user.save
=> true
```

代入してsave

saveするとupdated_atがsaveした日時に更新される

更新前にデータを代入している変数を作っていた場合．v.reloadで最新状態に変更する

2. 方法2

``` ruby
>> user.update_attributes(name: "The Dude", email: "dude@abides.org")
=> true
>> user.name
=> "The Dude"
>> user.email
=> "dude@abides.org"
```

update_attributesを使う (変更したい内容をHashで渡す)  
バリデーションチェックも行う

保存成功ならtrue，失敗ならfalseが返る

1つのカラムのみの変更の場合，update_attributeを使う

## 6.2 User validations

空白を許さなかったり，入力されるフォーマットを正規表現で指定するなどの制限 (validation) を設ける

> In this section, we’ll cover several of the most common cases, validating presence, length, format and uniqueness.

### 6.2.1 A validity test

> As noted in Box 3.3, test-driven development isn’t always the right tool for the job, but model validations are exactly the kind of features for which TDD is a perfect fit. It’s difficult to be confident that a given validation is doing exactly what we expect it to without writing a failing test and then getting it to pass.

バリデーションは通っているかどうかの手動テストが難しい (面倒くさい) ので，TDDにうってつけ

``` ruby
def setup
...
end
```

**setupメソッドは，各テストのはじめに実行される**

通るテストを書いて通るか確認をする

### 6.2.2 Validating presence

入力が空白でないかどうかを調べるものがpresence (空白ではないときに保存可能となる)  
今回はnameとemailカラムに適用する

とりあえず落ちるテストを書く

``` ruby
test "name should be present" do
    @user.name = "     "
    assert_not @user.valid?
end
```

``` validates :name, presence: true ```をUser modelに追記して

1. rails cでnameを空にしたものの.valid?がfalseになること
2. user.errors.full_messagesにエラーメッセージが入っていること
3. user.saveでfalseが返ってくること
4. 先ほど落ちたテストが通ること

を確認する

emailについても同様に，落ちるテストを書いてからpresence: trueを追記する

### 6.2.3 Length validation

文字列の長さに対しての制限はlengthを用いる  
今回はuserカラムで50文字，emailカラムで255文字より長いものは落とす制限をかける

落ちるテスト

``` ruby
test "name should not be too long" do
   @user.name = "a" * 51
   assert_not @user.valid?
end

test "email should not be too long" do
   @user.email = "a" * 244 + "@example.com"
   assert_not @user.valid?
end
```

落ちることを確かめた後，

``` ruby
validates :name,  presence: true, length: { maximum: 50 }
validates :email, presence: true, length: { maximum: 255 }
```

を追記し，テストが通ることを確かめる

### 6.2.4 Format validation

emailがメールアドレスとして正しいフォーマットになっているかどうかをバリデーションでチェックする

```assert @user.valid?, "#{valid_address.inspect} should be valid"```

**第二引数でテストが落ちた際のエラーメッセージをカスタマイズすることができる**

フォーマットのバリデーションを指定するときは，以下のようにする

``` ruby
validates :email, format: { with: /<regular expression>/ }
```

``` /<regular expression>/ ```には，validとされる文字列の正規表現を入れる

[Rubular](http://rubular.com/)は便利

Rubyでは，定数の変数名は全て大文字にする

### 6.2.5 Uniqueness validation

任意の2つのデータを選んだ時，指定したカラムのデータはペア間で必ず異なるようにするという制約は，uniquenessで実現する (ユニーク制約)

emailに，以下のように指定する

``` ruby
validates :email, presence: true, length: { maximum: 255 },
                  format: { with: VALID_EMAIL_REGEX },
                  uniqueness: true
```

また，emailは大文字小文字を区別しないため，DBに入れる時は小文字で統一して，同じメールアドレスが登録されることを防ぐ

バリデーション時に大文字小文字を区別しない制約は，以下のように指定する

``` ruby
validates :email, presence: true, length: { maximum: 255 },
                  format: { with: VALID_EMAIL_REGEX },
                  uniqueness: { case_sensitive: false }
```

**ただし，validationはモデルレベル (Rails上でのレベル) の制限であり，DBにこれらの制約が加えられていないことに十分注意する必要がある**

-----

### Box 6.2 Database indices

indexをつけていないカラムについて検索をするということは，データを1つ1つ順番に見ていくということ (full-table scan)  
これは効率が悪い

よく検索されるカラムが分かっている場合は，indexをつけることにより高速に検索が可能となる

-----

ログイン時はemailで該当ユーザを検索するため，emailにインデックスをはる

``` rails generate migration add_index_to_users_email ```

この指定方法では自動的にindexをはる行がマイグレーションファイルに追加されないため，以下のように手動で追加を行う

``` ruby
def change
    add_index :users, :email, unique: true
end
```

unique: trueで同じメールアドレスを許さない (DBレベルの制約)

before_saveはコールバックの一種であり，saveする前に必ず呼び出される　
今回はここで，emailを小文字に変換する

## 6.3 Adding a secure password

パスワードを保存するカラムを追加する

secure passwordはパスワードを暗号化したものをDBに書き込む

### 6.3.1 A hashed password

model fileに``` has_secure_password```を追記するだけで使えるようになる

has_secure_passwordは3つの機能をモデルに追加する

**1. password_digestカラムが存在する場合，そこに暗号化されたパスワードを保存する**  
**2. passwordとpassword_confirmationという2つの仮想属性を追加する (DBにはカラムとして存在しないがRails上で使える属性)**  
**3. 与えられたパスワードが正しいものかどうかを返すauthenticateメソッドを追加する**  

**passwordは入力されたパスワードの平文，password_confirmationは確認のためにもう1度入力されたパスワードの平文が入っている**

**password_digestカラムは自動で追加されないため，以下のように手動で追加する**

``` rails generate migration add_password_digest_to_users password_digest:string ```

また，このサンプルアプリでは暗号化の際のハッシュ関数としてbcrypt gemを利用するため，Gemfileにそれを追記する

### 6.3.2 User has secure password

User modelにhas_secure_passwordを追記して，テストが落ちることを確認

この場合はテストのUser.newに情報が足りないために起こるため，テストの方を変更する

### 6.3.3 Minimum password standards

攻撃者がパスワードを推測することを防ぐため，パスワードの最低限の長さをvalidationで指定する  
また，空のパスワードも防止する

まず落ちるテストを書き，以下のようにvalidationを指定する

``` ruby
validates :password, presence: true, length: { minimum: 6 }
```

### 6.3.4 Creating and authenticating a user

ユーザ認証の挙動を確認する

まだユーザ登録フォームを作っていないため，rails cからユーザを作り挙動を確かめてみる

**user.authenticateは，間違ったパスワードが与えられた場合はfalse，正しければUserのデータを返す**
