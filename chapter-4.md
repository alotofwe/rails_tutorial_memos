# Chapter 4 Rails-flavored Ruby

3章で，Railsにとって大事なRubyの側面をいくつか確認してきた

Railsでは，もちろんRubyは重要だが，Rubyという言語全体を理解する必要は無い

> This chapter is designed to give you a solid foundation in Rails-flavored Ruby, whether or not you have prior experience in the language.

RailsっぽいRubyを書けるようになることが目的

## 4.1 Motivation

``` ruby
<%= stylesheet_link_tag 'application', media: 'all',
                                       'data-turbolinks-track' => true %>
```

> there are at least four potentially confusing Ruby ideas: built-in Rails methods, method invocation with missing parentheses, symbols, and hashes.

この一行にもbuilt-in，括弧が無いメソッド呼び出し，シンボル，ハッシュというRubyの要素が詰まっている  
(そういえばRubyの本当の初心者の頃は，なぜ括弧がなくても大丈夫なのか気になったりしたな...)

``` ruby
<%= yield(:title) %> | Ruby on Rails Tutorial Sample App
```

> In addition to coming equipped with a large number of built-in functions for use in the views, Rails also allows the creation of new ones. Such functions are called helpers

viewのための組み込みメソッドをhelperと呼ぶ

ところで3章で各ページにてprivideでタイトルの一部を決定しているが，それが指定されなかった時のデフォルトタイトルを指定していない  
デフォルトを指定していないので，レイアウトファイルに書かれた```|```がそのまま表示されてしまう  

ここでhelperの話題が出てきたため，デフォルトタイトルを与えるようなhelperを追記してみる  
引数としてpage_titleを受け取り，空であればデフォルトを，そうでなければpage_titleを正しいフォーマットで追加したタイトルを返す，Rubyのメソッドを追記する (full_title(page_title)というメソッド名にする)

これをviewで，以下のように使用することができる

``` ruby
<%= full_title(yield(:title)) %>
```

heplerが書けた  

Home Pageではデフォルトタイトルを表示するようにテストを書き換え，落ちることを確認し，通るようにコードを修正する

> but it’s full of important Ruby ideas: modules, method definition, optional method arguments, comments, local variable assignment, booleans, control flow, string concatenation, and return values.

ここまででも既にたくさんのRubyの要素に触れている!
