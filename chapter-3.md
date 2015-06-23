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

### Box 3.1. Undoing things

```rails g```で生成されたファイルを1つ1つ手で消していくのは大変  
```rails destroy```で取り消すすることができる

また，```bundle exec rake db:migrate```も```bundle exec rake db:rollback```で取り消すことができる

バージョン番号を指定して```bundle exec rake db:migrate VERSION=n```とすることもできる

バージョン番号は```bundle exec rake db:migrate:status```で確認可能

generateでroutesにhome, help actionについて追記されている

``` ruby
get 'static_pages/home'
```

```get```でrequestを指定し，```'static_pages/home'```でURLを指定している  
このURLは，Static Pages Controllerのhomeアクションを指している

### Box 3.2. GET, et cet.

* GET : get a page. データを読む
* POST : ブラウザのフォームから何らかのリクエストを送る
* PATCH : サーバにあるデータをアップデートする
* DELETE : サーバにあるデータを消去する


> This is normal for a collection of static pages: the REST architecture isn’t the best solution to every problem.

RESTはいつでも適用できる，万能アーキテクチャではない  
時と場合によって使い分ける

現在controllerのメソッドは全て空なので何も行わないように思えるが  
実際にはApplication Controllerを継承している恩恵によって，actionを通ったあとによしなにviewをレンダリングしてくれる

viewについて，.erbはSection 3.4で学ぶが，自動生成されるviewは普通のhtmlとなっているので安心

### 3.2.2 Custom static pages

viewを (静的なままに) 書き換えてみる

