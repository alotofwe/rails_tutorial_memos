# Chapter 1 From zero to deploy

> If you are new to the subject, the Ruby on Rails Tutorial will give you a thorough introduction to web application development, including a basic grounding in Ruby, Rails, HTML & CSS, databases, version control, testing, and deployment—sufficient to launch you on a career as a web developer or technology entrepreneur.

Rails Tutorialでは，初心者がRuby, Rails, HTML, CSS, DB, バージョン管理，テスト，開発の基礎を学ぶことができる

> On the other hand, if you already know web development, this book will quickly teach you the essentials of the Rails framework, including MVC and REST, generators, migrations, routing, and embedded Ruby.

もし開発のことについてよく知っていたとしても，MVCやREST，ジェネレータ，マイグレーション，ルーティング，erbについてなどのRailsのフレームワーク周りについて学ぶことができる

> We’ll develop the sample app using a combination of mockups, test-driven development (TDD), and integration tests.

サンプルアプリケーションの開発ではモックアップ，TDD，インテグレーションテストを活用していく

## 1.1 Introduction

> What makes Rails so great? First of all, Ruby on Rails is 100% open-source, available under the permissive MIT License, and as a result it also costs nothing to download or use.

Railsはオープンソースであり，MITのライセンスを取得しており，ダウンロードしたり使用するためのコストが無いところが素晴らしい

> For example, Rails was one of the first frameworks to fully digest and implement the REST architectural style for structuring web applications.

RailsはRESTや構造的webアプリケーション(??)をざっくりと使ってみるための最初のフレームワークの1つだ

### 1.1.1 Prerequisites

この本を始める前に勉強しておくべきこと (Rubyの文法とか)

### 1.1.2 Conventions in this book

> For convenience, code resulting in a failing test is thus indicated with red, while code resulting in a passing test is indicated with green.

そのコードではtestが落ちるところはred, そのコードの修正でtestが通る場合はgreenと記述してある

> Such highlighted lines typically indicate the most important new code in the given sample, and often (though not always) represent the difference between the present code listing and previous listings.

ハイライトしてあるところは新しく追記されたコードの中で最も重要な部分  
ただし，新しくないところでもハイライトされている場合もある

> These dots represent omitted code and should not be copied literally.

ドットは省略してある印なのでコピペしないこと

## 1.2 Up and running

Windowsの人とかそうでない人とか色々な人がいるので，クラウド環境にしてみんなで環境諸々統一しよう (お断りします)

### 1.2.1 Development environment

Cloud9の簡単な操作説明

### 1.2.2 Installing Rails

```gem install rails -v 4.2.2```

あれ，Tutorialを進めていた時は4.2.0だったのに，脆弱性対策で4.2.2になっている (素晴らしい)

## 1.3 The first application

"hello world!"と表示されるだけのページを作る  
development, environmentの両方で動くようにする

作業場としてworkspaceディレクトリを作ってその中にサンプルアプリケーションをrails newしていこう

> Box 1.3. A crash course on the Unix command line

unix commandsの簡単な説明

> Note that Listing 1.3 explicitly includes the Rails version number (_4.2.2_) as part of the command. 

```rails _4.2.2_ new hello_app```

バージョン指定してrails new  
必要最小限の，Railsにとって一般的なファイル群が作成される

### 1.3.1 Bundler

> After creating a new Rails application, the next step is to use Bundler to install and include the gems needed by the app.

Bundlerでアプリケーションに必要なgemsをインストールすることができる  
必要なgemsはGemfileで管理する

> Unless you specify a version number to the gem command, Bundler will automatically install the latest requested version of the gem.

```gem 'sqlite3'```

指定しなければ常に最新版のgemが入る

> allows us to exert some control over the version used by Rails. The first looks like this:

```gem 'uglifier', '>= 1.3.0'```

指定したバージョン以降の，最新のgemが入る

> the >= notation always installs the latest gem, whereas the ~> 4.0.0 notation only installs updated gems representing minor point releases (e.g., from 4.0.0 to 4.0.1), but not major point releases (e.g., from 4.0 to 4.1).

```gem 'coffee-rails', '~> 4.0.0'```

**~>にすると，指定したバージョンのマイナーアップデート版のうち最新が入る**

### 1.3.2 rails server

ローカルで動かすには```rails server```

(Figure 1.10，2015-06-22 16:48:30の時点でrails 4.2.0.beta1のままだ)

### 1.3.3 Model-View-Controller (MVC)

> This is a hint that Rails follows the model-view-controller (MVC) architectural pattern, which enforces a separation between “domain logic” (also called “business logic”) from the input and presentation logic associated with a graphical user interface (GUI).

MVCとは，GUIで入力と表示を行うためのロジック

図を見て理解するのが早い

![MVC](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/mvc_schematic.png)

### 1.3.4 Hello, world!

controllerに```render text: ...```を追記して"hello world!"を表示させる

> To do this, we’ll edit the Rails router, which sits in front of the controller in Figure 1.11 and determines where to send requests that come in from the browser

routesを書き換えることで，ブラウザから来たリクエストをどのコントローラで処理するのかを決める  
ここでは，rootを変更する

## 1.4 Version control with Git

> Version control systems allow us to track changes to our project’s code, collaborate more easily, and roll back any inadvertent errors (such as accidentally deleting files).

バージョンコントロールは，コード変更の軌跡を追い，何かアクシデントが起きた時に過去の状態にもどることができるようなシステム  
簡単にプロジェクトに取り入れることが可能

TotorialではGitを使用し，オンラインリソースとしてBitbucket 101を利用する.  
(私はGithubを利用した)

### 1.4.1 Installation and setup

```git config --global user.name "Your Name"```  
```git config --global user.email your.email@example.com```

Gitのセットアップ  
セットアップで設定した名前とメールアドレスは，全てのリポジトリで公開される

```git init```  
アプリのルートディレクトリにて実行することで，新しいリポジトリの作成  

```git add -A```  

.gitignoreにマッチしないような全てのファイルをリポジトリに追加  
.gitignoreはrails newをした時に自動的に作成されている

追加されたファイルは**staging area**に置いてあり，この時点ではまだ変更は反映されていない  
staging areaのファイルは```git status```で見ることができる

```git commit -m "message"```

ファイル変更の反映  
-mでコミットにメッセージをつける

この時点ではlocalにコミットしている  

```git log```

コミットメッセージ等を見る

### 1.4.2 What good does Git do you?

ディレクトリをわざと消してみて，roll backを試してみる

> We see here that a file has been deleted, but the changes are only on the “working tree”; they haven’t been committed yet. This means we can still undo the changes using the checkout command with the -f flag to force overwriting the current changes:

コミットしていなければ，```git checkout -f```で直前のコミットした状態でやり直すことができる

### 1.4.3 Bitbucket

Bitbucketの説明  
リポジトリを簡単に他人とシェアすることができる

Githubは無制限にfree reposを，Bitbucketは無制限にprivate reposを使うことができる  

push等の仕方は，Githubでリポジトリを新たに作成すれば書いてあるので省略

### 1.4.4 Branch, edit, commit, merge

READMEを.mdに変更する

> Git is incredibly good at making branches, which are effectively copies of a repository where we can make (possibly experimental) changes without modifying the parent files.

Branchは，親ファイルを変更せずにいくらでもリポジトリをコピーすることができる  
(この説明，ざっくりしすぎなような...)

```git checkout -b new-repo-name```

新たなリポジトリの作成

```git branch```

ブランチ一覧の表示  
*がついているものが今いるブランチ

```git commit -a -m "message"```

> Be careful about using the -a flag improperly; if you have added any new files to the project since the last commit, you still have to tell Git about them using git add -A first.

-aは，最後のコミット以降で新しく作成したファイルのみを追加する

> Git models commits as a series of patches, and in this context it makes sense to describe what each commit does, rather than what it did.

**何をしたかではなく，どんなことができるようになるコミットであるかを，コミットメッセージに書こう**

```git checkout master```  
```git merge repo-name```

masterとマージ

```git branch -d repo-name```

ブランチを消す  
-Dで，対象ブランチに変更が加えられていて，かつマージされていない状態でも強制的に消す

```git push```

(Bitbucket|Github)にプッシュ

## 1.5 Deploying

Deployして世の中に出そう

今回はcloud deployment servicesとしてHerokuを利用する

Herokuはrailsアプリをデプロイするのが簡単であり，version constrolとしてgitを利用している

### 1.5.1 Heroku setup

HerokuはPostgreSQLを使っているので，pgというgemをproductionのために入れる (Gemfileに追記する)

また，assets管理のためにrails_12factorも追記する

**production用のgemはローカルに入れる必要はないが**  
**Gemfile.lockの更新のために```bundle install --without production```をしておく**

herokuをコマンドライン上で扱うために[Heroku Toolbelt](https://toolbelt.heroku.com/)を入れておく

```heroku login```  
```heroku keys:add```  
```heroku create```

**ログインしてssh keyを追加して，createでサーバ上にアプリを上げる場所を用意する**

### 1.5.2 Heroku deployment, step one

```git push heroku master```

**masterブランチをデプロイする**

### 1.5.3 Heroku deployment, step two

```heroku open```

**デプロイしたアプリをブラウザ上で閲覧する**

### 1.5.4 Heroku commands

```heroku rename rails-tutorial-hello```

**アプリの名前を変更する (既に存在するアプリの名前は使えない)**
