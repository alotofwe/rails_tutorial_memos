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

"".length : 文字列の長さが返る

"".empty? : 文字列が空かどうかがtrue or falseで返る

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

## 4.3 Other data structures

上に挙げた他にも，データ構造はたくさんある

### 4.3.1 Arrays and ranges

順序付き配列  
[]で囲まれる

"".split : 文字列から配列を作る．区切り文字は引数として指定可能，省略すると空白で区切る

Rubyの配列は0始まり

[].first : 配列の最初の要素を返す

[].second : 配列の二番目の要素を返す

[].last : 配列の最後の要素を返す

a == b : aとbの値が等しければtrue，そうでなければfalseが返る

a != b : aとbの値が等しければfalse，そうでなければtrueが返る

配列はlength以外にも，empty?, include?, sort, reverse, shuffleなど，様々なメソッドが使える

メソッド名の最後に``` ! ```がつくものは破壊的メソッドであり，メソッドを実行したオブジェクトの内容を書き換える  
そうでないものは，基本的にオブジェクトを書き換えない

[] << Object : 配列の末尾にオブジェクトをプッシュする

[].join : 配列の要素を結合して文字列を返す．引数として要素の間に文字列をいれることができる

rangeは配列に密接に関わり，配列を指定した部分のみ取り出したりすることができる

``` ruby4.3.3 Hashes and symbols
>> a = %w[foo bar baz quux]         # Use %w to make a string array.
=> ["foo", "bar", "baz", "quux"]
>> a[0..2]
=> ["foo", "bar", "baz"]
```

-1というインデックスは，配列の最後の要素を指す  
-2, -3 ... も同様，後ろから配列を見る

``` ruby
>> ary = [1, 2, 3, 4, 5]
=> [1, 2, 3, 4, 5]
>> ary[-2]
=> 4
>> a = (0..9).to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>> a[2..(a.length-1)]               # Explicitly use the array's length.
=> [2, 3, 4, 5, 6, 7, 8, 9]
>> a[2..-1]                         # Use the index -1 trick.
=> [2, 3, 4, 5, 6, 7, 8, 9]
```

### 4.3.2 Blocks

```{ |args| ... }```のようにブロックを指定する  
ブロックの中ではargsの値を使うことができ，argsの値はブロックの外で指定をする

``` { ```の代わりに```do```，``` } ```の代わりに```end```を使うこともできる  
一般的な書き方，ブロックが一行の時に{}，そうでない時にdo ~ endを使う  
(公式な規約だった気がする)

each，timesなどの後にブロックが指定された場合は，各each or timesの値ごとにブロックが実行され，実行ごとに与えられるargsが変わる

mapは，ブロックの実行結果を実行順に配列にしたものを返す  
また，以下の様な略記法もある

``` ruby
>> %w[A B C].map(&:downcase)
=> ["a", "b", "c"]
```

この例はシンボルを使っているが，シンボルはSection 4.3.3で詳しく扱う

今まで書いてきたtestもブロックを利用している (見ての通り，do ~ endで囲まれている)

### 4.3.3 Hashes and symbols

インデックスが必ずしも整数でなくともよいのがHash  
keyとvalueでペアになっている  
要素の順番指定は無い

hashrocket (=>) を使ってハッシュを定義できる

``` ruby
>> user = { "first_name" => "Michael", "last_name" => "Hartl" }
=> {"last_name"=>"Hartl", "first_name"=>"Michael"}
```

keyを指定するのは文字列でもよいが，シンボルを使ってもよい (:name)  
シンボルは文字列の仲間みたいにみえるが，少なくともいくつかのメソッドが使えないという点で文字列と違う

> シンボルを表すクラス。シンボルは任意の文字列と一対一に対応するオブジェクトです。  
> 文字列の代わりに用いることもできますが、必ずしも文字列と同じ振る舞いをするわけではありません。   
> 同じ内容のシンボルはかならず同一のオブジェクトです。

シンボルオブジェクトは以下のようなリテラルで得られます。

[class Symbol](http://docs.ruby-lang.org/ja/2.0.0/class/Symbol.html)

デフォルトでは，宣言されていないkeyのvalueはnilとなる  
Hash.new(default value)でこれを健康することが可能

``` ruby
>> h = Hash.new("empty")
=> {}
>> h['hoge']
=> "empty"
```

シンボルを使うと=>を使わなくとも```{ name: "Michael Hartl", email: "michael@example.com" }```と書くことができる

**```inspect```メソッドはリテラル表現にされた文字列をそのまま返す**  
**しばしば``` p ```と略される**

### 4.3.4 CSS revisited

4.1で見たcssを読み込むコードに戻って文法を再度確かめる

> is a call to this function. But there are several mysteries. First, where are the parentheses? In Ruby, they are optional, so these two are equivalent:

メソッド呼び出し時の括弧は省略可能

> When hashes are the last argument in a function call, the curly braces are optional, so these two are equivalent:

引数のうち最後がHashだった場合は，{}を省略することが可能

4.1のコードではkeyにハイフンが使われているため，これをシンボルとして宣言することは不可能

メソッド呼び出しの途中の空白や改行は無視される

