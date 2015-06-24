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

## 4.2 Strings and methods

Rails ConsoleはInteractive Ruby (irb)の上に構築されている  
そのため，ターミナルで```rails console```と打つと，Rubyの様々な要素を試すことができる

> By default, the console starts in a development environment, which is one of three separate environments defined by Rails (the others are test and production).

**NOBODY expects the Spanish Inquisition!!!!**

Our chief wepon is development environment...and test environment...  
Our two weapons are development environment and test environment...and production environment...  
Our **three** weapons are development environment and test environment and production environment...and development... I'll come in again.

rails consoleは，デフォルトではdevelopmentで起動する (environmentについてはSection 7.11で更に詳しく学ぶ)

### 4.2.1 Comments

``` # ```以降の，その行の文字は全てコメントとなる  
コメントは人間がコードを読むときに有効

### 4.2.2 Strings

ダブルクオートで囲まれたものは文字列リテラルとなる

```"foo" + "bar"```で連結，文字列の中に```#{}```を入れることで他の文字列を挿入することができる

```puts "string"```で"string"を出力することができる  

putsは副作用として文字列を出力し，puts自体の結果はnil (nothing at all) になる

また，putsは文字列の後ろに改行文字を自動で含ませるが，```print```は改行文字を含ませずに出力を行う

シングルクオートを使ったものも文字列となる

> There’s an important difference, though; Ruby won’t interpolate into single-quoted strings:

シングルクオートを使った文字列は，```#{}```が使えない

> They are often useful because they are truly literal, and contain exactly the characters you type. For example, the “backslash” character is special on most systems, as in the literal newline \n. If you want a variable to contain a literal backslash, single quotes make it easier:

**シングルクオートを使った文字列の利点は，囲んだ文字は必ずそのまま文字列リテラルとなること**  
**余計な操作が加わることもなければ，手動でエスケープをする必要がない**

> There’s really nothing to be done about this, except to say, “Welcome to Ruby!”

### 4.2.3 Objects and message passing

Rubyでは，nilも含めて全てがオブジェクトである

> you have to build up your intuition for objects by seeing lots of examples.

例をたくさん見ることで理解していこう

String.length : 文字列の長さが返る

String.empty? : 文字列が空かどうかがtrue or falseで返る

Rubyではtrue or falseで結果が返るメソッドには，最後に?がつく  

true or falseが返るようなものは，if-elsif-elseなどのような処理の分岐に役立つ  
また，&& (and), || (or), ! (not)が使える

もちろんnilもオブジェクトなので，いくつかのメソッドが使える  
ただし存在しないメソッドを使おうとするとNoMethodErrorが起こる

また，chaining (メソッドを連鎖させること) が使える

後置ifが使える

ifとは逆に，条件式がfalseになるときにその中の処理を実行する```unless```がある

nilはfalseであるという意味も含む (!!nilするとfalseになる)

### 4.2.4 Method definitions

メソッドのお話

デフォルト引数が使える  
もし呼び出されるときにその引数が指定されなかった時は，そのデフォルト値が使われる

Rubyのメソッドでは，明示的にreturnをせずとも，一番最後の評価が返り値となる  
明示的にreturnすることも可能

### 4.2.5 Back to the title helper

メソッドのことを学んだので，改めてこの章の最初で書いたhelperを見てみる

helperをincludeすることで，includeした先でhelperのメソッドを使用することができる (mixed in)

