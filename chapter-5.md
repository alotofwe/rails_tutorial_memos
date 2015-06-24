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
