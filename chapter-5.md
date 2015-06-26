# Chapter 5 Filling in the layout

4章でRubyについて簡単に知ったので，これを踏まえて3章のアプリのレイアウトを整える方法について学ぶ

ただし，自前でCSSをごりごりと書くわけではなく，外部のフレームワークを利用する

> Along the way, we’ll learn about partials, Rails routes, and the asset pipeline, including an introduction to Sass (Section 5.2). We’ll end by taking a first important step toward letting users sign up to our site (Section 5.4).

partials, routes, asset pipeline (sass含む) もここで学ぶ

TDDでまわしていく

## 5.1 Adding some structure

ここではBootstrapを利用する

まず，モックアップで見た目をスケッチしておきましょう

ここでブランチも切っておく

### 5.1.1 Site navigation

ページのナビゲーションをレイアウトファイルに追記する  
今はとりあえず見た目だけで，リンクを貼るなどは後で追記していく

バージョン9未満のIEでは一部のjsがサポートされていないため，ifで特別に処理しておく

> All HTML elements can be assigned both classes and ids; these are merely labels, and are useful for styling with CSS (Section 5.1.2). The main difference between classes and ids is that classes can be used multiple times on a page, but ids can be used only once. 

idはページにつき1回しか使うことができない．classは何度でも使える

``` link_to ```の第一引数はリンクのタイトル，第二引数はリンク先のURL (named routes (in Section 5.3.3) を使用することができる)

liはlist itemの略

``` image_tag ```には画像へのパスを引数として渡す  
これの実行の結果imgタグが生成されるが，**srcがユニークになるように適当な名前に変換される**

> Note that the src attribute doesn’t include images, instead using an assets directory common to all assets (images, JavaScript, CSS, etc.).  
> On the server, Rails associates images in the assets directory with the proper app/assets/images directory, but as far as the browser is concerned all the assets look like they are in the same directory, which allows them to be served faster.

**src="/assets/rails-xxx.png"とあり，画像がapp/assets/imagesの中にあるにも関わらずimagesディレクトリが指定されていない**  
**assets以下を全て同一ファイルとして見ており，これによりより速く表示等をさせることができる**

### 5.1.2 Bootstrap and custom CSS

更に良い見た目になるように，CSSをカスタマイズしていく

**また，Bootstrapは自動的にレスポンシブデザインにしてくれるため，デバイスの種類を考慮する必要はない**

[レスポンシブデザインとは｜RWD｜responsive design - 意味/解説/説明/定義 ： IT用語辞典](http://e-words.jp/w/%E3%83%AC%E3%82%B9%E3%83%9D%E3%83%B3%E3%82%B7%E3%83%96%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3.html)

BootstrapはLess CSSを使用しているが，Sassが使えるようにbootstrap-sass gemを入れておく

Sassが使えるようなCSSは，**"Sassy CSS"の略である.sassがファイルにつく**

cssに``` @import "" ```を書くことで，Bootstrapの関数を利用できるようにすることができる

### 5.1.3 Partials

Railsでは部分テンプレート (Partials) を使ってテンプレートを使いまわすことができる

``` ruby
<%= render 'layouts/shim' %>
```

'directory/html'を部分テンプレートとして表示させる

この場合は，htmlの名前は最初に_をつけて``` _shim.html.erb ```となる  
最初に_をつけるのは部分テンプレートの取り決めである (render宣言時にはつけない)

header, footerも部分テンプレート化しておく

## 5.2 Sass and the asset pipeline

最近のRailsではasset pipeline機能が追加されている  
これはproduction環境下にてCSS, JavaScript, imagesの管理を向上させる機能である

### 5.2.1 The asset pipeline

> The asset pipeline involves lots of changes under Rails’ hood, but from the perspective of a typical Rails developer there are three principal features to understand: asset directories, manifest files, and preprocessor engines.

* asset directories

assetsを入れる3つのディレクトリがある

- app/assets: 大部分のassets
- lib/assets: 独自開発したライブラリ
- vendor/assets: 外から持ってきたライブラリ

* Manifest files

自動生成されるapplication.cssのようなファイルのことを言う  

**コメントアウトされているように見えるが，Sprockets gemにより**きちんと機能している

```
*= require_tree .
```

Manifest fileがあるディレクトリ，加えてそれよりも下のディレクトリにあるcssを全て読み込む

```
*= require_self
```

自分自身を読み込む

* Preprocessor engines

**Railsがテンプレートを表示させるときにプリプロセッサを動かす**  
**典型的なものだと.scss, .coffee, .erbがある**  

いつ，どのファイルで何のプリプロセッサが走るかは，fileの拡張子として指定する

> foobar.js.erb.coffee

**1つのファイルにつき2つ以上のプリプロセッサを走らせることもできる**

* Efficiency in production

**assets pipelineがあるおかげで，ファイルを自力で分割して管理しなくとも**  
**どのcssが必要でどれが不必要かを判断し，不必要なものは削除している**

**この機能があるおかげで，js, css, imagesについてのロード時間を開発者が気にしなくても済む**

### 5.2.2 Syntactically awesome stylesheets

> Sass is a language for writing stylesheets that improves on CSS in many ways. In this section, we cover two of the most important improvements, nesting and variables. (A third technique, mixins, is introduced in Section 7.1.1.)

sassを導入すると，入れ子や変数など，cssにはない機能が使えて便利

cssでできることは全て，scssでも行うことができる

* Nesting

``` scss
.center {
  text-align: center;
  h1 {
    margin-bottom: 10px;
  }
}
```

親の名前を使いたい時は&を利用する

``` scss
#logo {
  float: left;
  margin-right: 10px;
  font-size: 1.7em;
  color: #fff;
  text-transform: uppercase;
  letter-spacing: -1px;
  padding-top: 9px;
  font-weight: bold;
  &:hover {
    color: #fff;
    text-decoration: none;
  }
}
```

* variables

値を変数に代入することができる

``` scss
$light-gray: #777;
```

Less CSSは@，Sassは$を変数名の頭につける

## 5.3 Layout links

今こそリンクを有効にする時

named routesを用いることで，pathをベタ書きせずに済む

named routes一覧は``` bundle exec rake routes ```で確認可能 (各行の先頭に書かれている名前の後ろに_pathをつけたものがそれ)

### 5.3.1 Contact page

コンタクトページを追加する  

まずgetするテストを書いて落ちることを確認

その後通るようにコードを書き換える

### 5.3.2 Rails routes

rootについてはroot_pathとroot_urlが使用可能  
root_pathは完全なURLではなく (host等が省略されている)，root_urlは完全なURLを返す

``` ruby
root_path -> '/'
root_url  -> 'http://www.example.com/'
```

named routesを使用するには，routes.rbを以下の書式になおす必要がある

``` ruby
get 'url' => 'controller#action'
```

### 5.3.3 Using named routes

5.1.1に書いたとおり，link_toの第二引数にnamed routesを指定することができる

### 5.3.4 Layout link tests

レイアウトファイルの各リンクが正しいURLを指しているかをテストする

``` ruby
assert_select "a[href=?]", about_path
```

は，?にabout_pathの結果を挿入する

``` ruby
assert_select "a[href=?]", root_path, count: 2
```

そのようなaタグがいくつあるかもチェックすることができる

テストが通ることを確かめる

## 5.4 User signup: A first step

サインアップ (ユーザ登録) ページを作り始める

> This is a first important step toward allowing users to register for our site; we’ll take the next step, modeling users, in Chapter 6, and we’ll finish the job in Chapter 7.

ちょっとずつ改良して頑張りましょう

### 5.4.1 Users controller

```rails generate controller Users new```

controllerを生成している

controller名はキャメルケースで複数形，その後にオプションとしてactionをつける (複数可)

自動的にテスト，コントローラ，ビューのファイルが生成される

### 5.4.2 Signup URL

routes.rbには自動的に追記されないため，手でnewアクション周りのルーティングを追記する

```get 'signup'  => 'users#new'```と追記したため，named pathはsignup_pathとなる  
Home Pageにこのパスを使ったlink_toを追記し，サインアップページに飛べるようにする
