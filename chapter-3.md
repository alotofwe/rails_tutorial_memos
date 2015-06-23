# Chapter 3 Mostly static pages

これ以降の章に渡って作っていくアプリをようやく作り始める

まずはシンプルに，静的ページから

> we can easily add just a small amount of dynamic content. In this chapter we’ll learn how.

動的な要素もちょっとだけ入れる (本当にちょっとだけ)

> Along the way, we’ll get our first taste of automated testing, which will help us be more confident that our code is correct  

また，この章からテストを書いていく

> Moreover, having a good test suite will allow us to refactor our code with confidence, changing its form without changing its function.

良いテストを書くと，自信を持ってリファクタリングができる  
きちんとテストをすれば，書き換えによって機能が望まれないかたちに変化してしまうことがない

## 3.1 Sample app setup

```rails new```してGemfileをいじって```bundle install --without production```する (Chapter 1 参照)

Gemfileのバージョンを今一度指定するために```bundle update```を行うべき  
(Gemfile.lockを無視する)

> Update the gems specified (all gems, if none are specified), ignoring the previously installed gems specified in the Gemfile.lock. In general, you should use bundle install(1) to install the same exact gems and versions across machines.

[bundle-update(1) - Update your gems to the latest available versions](http://bundler.io/v1.5/man/bundle-update.1.html)

```git init```して```git commit```してREADMEを変更して```git commit```して```git push```して```heroku create```して```git ush heroku master```する (Chapter 1 参照)

```heroku logs```

herokuでのログを出力して見る (production環境)

## 3.2 Static pages

準備が終わったので開発を始める

> In this section, we’ll take a first step toward making dynamic pages by creating a set of Rails actions and views containing only static HTML.

controllerとviewを使って動的ページも作ることができる (ということもやる)

```git checkout master```  
```git checkout -b static-pages```

ここでブランチを切ってみましょう

### 3.2.1 Generated static pages

```rails generate controller StaticPages home help```

```rails generate``` ( = ```rails g```) を使ってStatic Sages Controllerを生成する

**コマンド内のcontroller指定はキャメルケースで，複数形**

ただしこれはRailsの慣習で，スネークケースでも同様に動く

> キャメルケースとは、アルファベットで複合語やフレーズを表記する際、各単語や要素語の先頭の文字を大文字で表記する手法のことである。

[キャメルケースとは - IT用語辞典 Weblio辞書](http://www.weblio.jp/content/%E3%82%AD%E3%83%A3%E3%83%A1%E3%83%AB%E3%82%B1%E3%83%BC%E3%82%B9)

オプションとして，actionを小文字で指定  
ここではhome, helpを指定

Railsには色々なショートカットがある

* rails server -> rails s
* rails console -> rails c
* rails generate -> rails g
* **bundle install -> bundle**
* **rake test -> rake**

ここで```git add && git commit -m 'message' && git push```しておく

-----

### Box 3.1. Undoing things

```rails g```で生成されたファイルを1つ1つ手で消していくのは大変  
```rails destroy```で取り消すすることができる

また，```bundle exec rake db:migrate```も```bundle exec rake db:rollback```で取り消すことができる

バージョン番号を指定して```bundle exec rake db:migrate VERSION=n```とすることもできる

バージョン番号は```bundle exec rake db:migrate:status```で確認可能

-----

generateでroutesにhome, help actionについて追記されている

```ruby
get 'static_pages/home'
```

getでrequestを指定し，'static_pages/home'でURLを指定している  
このURLは，Static Pages Controllerのhomeアクションを指している

-----

### Box 3.2. GET, et cet.

* GET : get a page. データを読む
* POST : ブラウザのフォームから何らかのリクエストを送る
* PATCH : サーバにあるデータをアップデートする
* DELETE : サーバにあるデータを消去する

-----

> This is normal for a collection of static pages: the REST architecture isn’t the best solution to every problem.

RESTはいつでも適用できる，万能アーキテクチャではない  
時と場合によって使い分ける

現在controllerのメソッドは全て空なので何も行わないように思えるが  
実際にはApplication Controllerを継承している恩恵によって，actionを通ったあとによしなにviewをレンダリングしてくれる

viewについて，.erbはSection 3.4で学ぶが，自動生成されるviewは普通のhtmlとなっているので安心

### 3.2.2 Custom static pages

viewを (静的なままに) 書き換えてみる

## 3.3 Getting started with testing

homeとhelpのページを作成したので，次はaboutページを作成する  
ここでは，自動テストを用いてaboutページを作っていく(TDD)

> Developed over the course of building an application, the resulting test suite serves as a safety net and as executable documentation of the application source code.

テストはソースコードのドキュメントになり，かつアプリのセーフティーネットとなる (アプリの安全を確保することができる)  
(そんなに言うほどドキュメント代わりになるものだろうか...)

> When done right, writing tests also allows us to develop faster despite requiring extra code, because we’ll end up wasting less time trying to track down bugs.

テストを書くのは多少手間ではあるけれど，バグの発見が早くなるので，結果的に開発が速くなる

-----

### Box 3.3. When to test

いつ，どのようにしてテストを書くかを考えるときは，なぜテストを書くかを考えてみる

いつテストを書くべきかのガイドラインは以下のとおり

**1. テストコードが実際のコードに比べて簡潔になりそうな時はtest first**  
**2. 今のコードの状態では目的の完成までまだ遠く，先がまだまだ長そうな時は結果を明確にするためにtest first**  
**3. セキュリティ周りは最重要事項であり何かあっては困るので，テストを書く**  
**4. バグを再現し，それにたいして正しい対策がされているかどうか確かめるために，テストを書く**  
**5. この後変更される可能性が高い場合は，テストを書く**  
**6. リファクタリングする前に，リファクタリングによって求められる機能が失われないようにテストを書く**

-----

> Our main testing tools will be controller tests (starting in this section), model tests (starting in Chapter 6), and integration tests (starting in Chapter 7).

**NOBODY expects the Spanish Inquisition!!!!**

Our chief wepon is controller tests...and mode tests...and controller tests...  
Our two weapons are controller and model tests...and integration tests...  
Our **three** weapons are controller and model and integration tests...and controller tests... I'll come in again.

### 3.3.1 Our first test

aboutページへのテストを書いていく

テストの準備は，```rails generate controller```を実行した時に既に終わっている

```ruby
test "should get home" do
  get :home
  assert_response :success
end
```

get :homeはstatic_pagesのhomeページに飛ぶこと

:successとはHTTPのstatus codeとして200が返ってくること

> says “Let’s test the Home page by issuing a GET request to the home action and then making sure we receive a ‘success’ status code in response.”

テストすると通ることが分かる

### 3.3.2 Red

TDDとは本来，落ちるテストを書き，落ちることを確かめ，落ちないようにコードを修正し，通ることを確かめるというやり方  
落ちるテストを書いてみる

まだaboutページは作成していないが，aboutページに飛ぶとsuccessが返るテストを書き，落ちることを確かめる

### 3.3.3 Green

3.3.2のテストが落ちたので，通るようにアプリのコードを修正する

テストが落ちた際のエラーメッセージを読み，それを解消するように修正していく

```No route matches {:action=>"about", :controller=>"static_pages"}```

routesに```get 'static_pages/about'```を追記

```The action 'about' could not be found for StaticPagesController```

StaticPagesControllerにaboutメソッドを追記

```ActionView::MissingTemplate: Missing template static_pages/about```

viewとしてabout.html.erbを作成し，htmlを追記

これでテストが通る

### 3.3.4 Refactor

テストが通ったら香り高いコードを修正すべき

## 3.4 Slightly dynamic pages

ちょっと動的にする  
ページごとにタイトルを変える

本格的に動的にするのはChapter 7から

test firstにする

タイトルのフォーマットは"<page name> | Ruby on Rails Tutorial Sample App"というものにする

```rails new```をしたときに，app/views/layouts/application.html.erbが生成されているはずなので，一旦別名にして退避させておく  
(一般的な方法ではない)

### 3.4.1 Testing titles (Red)

TDDらしく，とりあえず落ちるコードを書く

典型的なHTMLの構成では，doctypeでhtmlのバージョンを指定し，headでtitleなどを指定し，bodyにページの内容を書く

```assert_select "HTML tag", "expected text"```

ここでのテストでは，```assert_select```を用いることで，HTMLタグを指定してその内容を検証する

これを使用したtitleチェックのテストを書き，現時点では落ちることを確認する

### 3.4.2 Adding page titles (Green)

落ちるテストが通るように，コードを修正する

簡単に，home, help, aboutの各htmlに違うtitleを指定し，テストが通ることを確認する
