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

もしマイグレーション後にミスが発覚したら，```  bundle exec rake db:rollback ```で元に戻す

また，create_tableの代わりにdrop_tableを用いることで，テーブルを消すことも可能

migration up (db:migrateすること) とmigration down (db:rollbackすること) で違うことを行わせたい場合は，up, downメソッドを使う

[Active Record Migrations — Ruby on Rails Guides](http://edgeguides.rubyonrails.org/active_record_migrations.html)

### 6.1.2 The model file

Userのためのmodel fileが生成されている (ActiveRecort::Baseを継承していることを確かめよ)

### 6.1.3 Creating user objects

``` rails console --sandbox ```

このrails consoleの実行終了後に，ここで行ったDBへの操作を全てなかったことにする (roll back (undo) する)

--sandboxを付けなければ，DBへの変更はそのままアプリで使用しているDBに反映される

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

find_byで指定した内容に当てはまる複数のユーザが存在した際は，一番若いidのものを返す

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
